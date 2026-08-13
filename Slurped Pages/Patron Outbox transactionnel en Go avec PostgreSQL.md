---
link: https://www.glukhov.org/fr/app-architecture/integration-patterns/transactional-outbox-pattern-go/
byline: À propos de Rost Glukhov
site: Rost Glukhov | Site personnel et blog technique
excerpt: Arrêtez de perdre des événements entre votre base de données et votre broker de messages. Découvrez le pattern de la boîte de sortie transactionnelle en Go avec PostgreSQL, FOR UPDATE SKIP LOCKED et un relais de sonde.
slurped: 2026-07-03T15:34
title: Patron Outbox transactionnel en Go avec PostgreSQL
tags:
  - postgresql
  - workflow
---

Deux écritures qui devraient réussir ensemble échoueront éventuellement séparément. Votre service de commandes enregistre la commande dans la base de données, puis publie un événement `order.created` dans un broker de messages.

Ces deux opérations s’exécutent l’une après l’autre.

Entre les deux, les choses peuvent mal tourner : le broker est indisponible, le réseau expire, le processus redémarre ou le conteneur est expulsé. L’écriture dans la base de données a réussi. La publication a échoué. Le service en aval qui doit connaître la nouvelle commande n’en est jamais informé. Personne n’a remarqué avant qu’un client n’appelle.

C’est le problème de l’écriture double, et c’est l’une des sources les plus courantes de perte de données silencieuse dans les systèmes distribués. Le motif de la boîte de sortie transactionnelle (transactional outbox) est la solution standard.
## Le problème de l’écriture double

Le mode de défaillance est facile à comprendre une fois qu’on le voit :

```
BEGIN;
  INSERT INTO orders ...   -- réussit
COMMIT;

PUBLISH order.created ...  -- échoue, plante ou n'est jamais atteint
```

La base de données et le broker de messages ne partagent pas de frontière de transaction. Il n’y a pas de rollback qui couvre les deux. Chaque service qui effectue `save -> publish` en séquence présente cette lacune. Le motif se manifeste sous de nombreuses formes :

- `db.Save(order)` suivi de `events.Publish(OrderCreated{...})`
- Gestionnaire HTTP qui commit une transaction puis appelle un webhook externe
- Travailleur (worker) qui traite un enregistrement d’une file d’attente et écrit les résultats dans une autre

Le résultat est le même dans tous les cas : un côté réussit tandis que l’autre échoue, et le système se retrouve dans un état invisible pour la surveillance car les deux opérations individuelles ont retourné un succès à un moment donné.

Une boucle de réessai ne corrige pas cela. Réessayer la publication après le commit de la base de données ne fonctionne que si le réessai lui-même est fiable – ce qui nécessite exactement la garantie de durabilité que vous n’avez pas.

## Ce que fait le motif de la boîte de sortie transactionnelle

Le motif de la boîte de sortie élimine la lacune en supprimant entièrement la publication directe. Au lieu d’appeler le broker depuis votre logique métier, vous écrivez un enregistrement d’événement dans une table `outbox` dans la même transaction de base de données que les données métier. Un processus en arrière-plan séparé – le relais – lit depuis la table de la boîte de sortie et publie dans le broker.

```
BEGIN;
  INSERT INTO orders ...         -- données métier
  INSERT INTO outbox_events ...  -- enregistrement d'événement
COMMIT;

-- Processus relais (séparément) :
SELECT ... FROM outbox_events FOR UPDATE SKIP LOCKED;
PUBLISH order.created ...
UPDATE outbox_events SET processed_at = NOW() WHERE id = $1;
```

Les deux écritures réussissent ou échouent toutes les deux. La garantie de transaction que vous avez déjà avec PostgreSQL couvre désormais aussi l’enregistrement d’événement. Le relais peut réessayer de publier autant de fois que nécessaire car l’événement réside dans un stockage durable. Si le relais plante en cours de route, il redémarre et réessaie. Le pire scénario est que l’événement soit publié plus d’une fois – ce qui est géré en rendant les consommateurs idempotents (voir [Idempotence dans les systèmes distribués](https://www.glukhov.org/fr/app-architecture/integration-patterns/idempotency-in-distributed-systems/ "L'idempotence n'est pas une astuce HTTP. Apprenez à arrêter les écritures dupliquées, les messages rejoués et les doubles facturations à travers les API, les files d'attente, les webhooks et les workflows.")).

## Schéma PostgreSQL pour la table de la boîte de sortie

Le schéma est délibérément simple :

```
CREATE TABLE outbox_events (
    id             UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
    aggregate_type VARCHAR(100) NOT NULL,
    aggregate_id   VARCHAR(100) NOT NULL,
    event_type     VARCHAR(100) NOT NULL,
    payload        JSONB        NOT NULL,
    attempts       INT          NOT NULL DEFAULT 0,
    created_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    processed_at   TIMESTAMPTZ
);

-- Index partiel : indexe uniquement les lignes non traitées, reste petit à mesure que les lignes sont marquées comme terminées
CREATE INDEX idx_outbox_unprocessed
    ON outbox_events (created_at)
    WHERE processed_at IS NULL;
```

L’index partiel sur `created_at WHERE processed_at IS NULL` est important. Sans lui, l’index grandit avec chaque événement écrit et la requête de sondage du relais devient plus lente au fil du temps. Avec lui, l’index couvre uniquement les lignes en attente, qui constituent, en état stable, un petit ensemble borné, indépendamment du nombre d’événements publiés.

Choix clés des champs :

- `aggregate_type` et `aggregate_id` décrivent l’entité à laquelle l’événement appartient. Utile pour les garanties d’ordonnancement et le routage.
- `event_type` est le nom de l’événement attendu par vos consommateurs.
- `payload JSONB` stocke le corps de l’événement. Utilisez `JSONB` plutôt que `TEXT` afin de pouvoir l’interroger si nécessaire.
- `attempts` suit le nombre de fois que le relais a tenté de publier cette ligne. Utilisé pour les limites de réessai et la gestion des lettres mortes.
- `processed_at` est `NULL` pour les lignes en attente et est défini lorsque le relais publie avec succès.

## Écriture des données métier et de l’événement de la boîte de sortie dans une seule transaction

La logique métier écrit les deux enregistrements à l’intérieur d’un seul appel `BeginTx` / `Commit`. Il n’y a pas d’appel de publication ici – uniquement des écritures de base de données.

```
type OrderService struct {
    db *sql.DB
}

func (s *OrderService) CreateOrder(ctx context.Context, order Order) error {
    tx, err := s.db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("begin tx: %w", err)
    }
    defer tx.Rollback()

    if _, err := tx.ExecContext(ctx, `
        INSERT INTO orders (id, customer_id, total, created_at)
        VALUES ($1, $2, $3, NOW())
    `, order.ID, order.CustomerID, order.Total); err != nil {
        return fmt.Errorf("insert order: %w", err)
    }

    payload, err := json.Marshal(map[string]any{
        "order_id":    order.ID,
        "customer_id": order.CustomerID,
        "total":       order.Total,
    })
    if err != nil {
        return fmt.Errorf("marshal payload: %w", err)
    }

    if _, err := tx.ExecContext(ctx, `
        INSERT INTO outbox_events
            (aggregate_type, aggregate_id, event_type, payload)
        VALUES ($1, $2, $3, $4)
    `, "order", order.ID, "order.created", payload); err != nil {
        return fmt.Errorf("insert outbox event: %w", err)
    }

    return tx.Commit()
}
```

Si `tx.Commit()` échoue, ni la ligne de commande ni la ligne de la boîte de sortie ne sont persistées. Si elle réussit, les deux sont garanties d’être dans la base de données. Le relais peut publier l’événement à tout moment après cela – immédiatement, dans une seconde, ou après le redémarrage du relais suite à un plantage.

C’est la seule modification de code requise dans votre couche métier. Le reste du motif réside dans le relais.

## Implémentation du relais en Go

Le relais est un travailleur en arrière-plan qui sonde la table de la boîte de sortie selon un minuteur. Il récupère un lot de lignes non traitées, publie chacune et la marque comme terminée. Gardez-le dans le même binaire que votre application ou exécutez-le comme un processus séparé – les deux fonctionnent, mais le même binaire est plus simple à exploiter.

```
type OutboxRelay struct {
    db          *sql.DB
    publisher   Publisher
    logger      *slog.Logger
    batchSize   int
    pollInterval time.Duration
    maxAttempts  int
}

func (r *OutboxRelay) Run(ctx context.Context) error {
    ticker := time.NewTicker(r.pollInterval)
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-ticker.C:
            if err := r.processBatch(ctx); err != nil {
                r.logger.Error("outbox relay batch failed", "err", err)
            }
        }
    }
}
```

Le relais respecte l’annulation du contexte, ce qui facilite l’intégration avec l’arrêt élégant. Pour un traitement détaillé de la durée de vie du contexte et des motifs d’annulation, consultez [Go context.Context Done Right](https://www.glukhov.org/fr/app-architecture/code-architecture/go-context-cancellation-timeouts/ "Maîtrisez Go context pour l'annulation, les délais d'attente et les valeurs à portée de requête. Couvre les gestionnaires HTTP, les appels de base de données, les travailleurs en arrière-plan, les fuites de goroutines et l'arrêt élégant.").

## FOR UPDATE SKIP LOCKED : le motif de travailleur concurrent

La fonction `processBatch` utilise `FOR UPDATE SKIP LOCKED` pour gérer en toute sécurité les travailleurs relais concurrents :

```
func (r *OutboxRelay) processBatch(ctx context.Context) error {
    tx, err := r.db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("begin tx: %w", err)
    }
    defer tx.Rollback()

    rows, err := tx.QueryContext(ctx, `
        SELECT id, aggregate_type, aggregate_id, event_type, payload
        FROM outbox_events
        WHERE processed_at IS NULL
          AND attempts < $1
        ORDER BY created_at
        LIMIT $2
        FOR UPDATE SKIP LOCKED
    `, r.maxAttempts, r.batchSize)
    if err != nil {
        return fmt.Errorf("query outbox: %w", err)
    }
    defer rows.Close()

    type row struct {
        id            string
        aggregateType string
        aggregateID   string
        eventType     string
        payload       json.RawMessage
    }

    var batch []row
    for rows.Next() {
        var e row
        if err := rows.Scan(
            &e.id, &e.aggregateType, &e.aggregateID, &e.eventType, &e.payload,
        ); err != nil {
            return fmt.Errorf("scan row: %w", err)
        }
        batch = append(batch, e)
    }
    if err := rows.Err(); err != nil {
        return err
    }

    for _, e := range batch {
        if err := r.publisher.Publish(ctx, e.eventType, e.aggregateID, e.payload); err != nil {
            r.logger.Error("publish failed", "event_id", e.id, "err", err)
            if _, err := tx.ExecContext(ctx,
                `UPDATE outbox_events SET attempts = attempts + 1 WHERE id = $1`, e.id,
            ); err != nil {
                r.logger.Error("increment attempts failed", "event_id", e.id, "err", err)
            }
            continue
        }

        if _, err := tx.ExecContext(ctx,
            `UPDATE outbox_events SET processed_at = NOW() WHERE id = $1`, e.id,
        ); err != nil {
            return fmt.Errorf("mark processed: %w", err)
        }
    }

    return tx.Commit()
}
```

`FOR UPDATE SKIP LOCKED` fait deux choses. Premièrement, `FOR UPDATE` verrouille les lignes sélectionnées pour la durée de la transaction, empêchant toute autre transaction de les sélectionner. Deuxièmement, `SKIP LOCKED` signifie que si une ligne est déjà verrouillée par une autre transaction, la requête la saute au lieu d’attendre. Le résultat est que plusieurs travailleurs relais peuvent s’exécuter en parallèle et chacun récupérera un sous-ensemble de lignes non chevauchant.

Sans `SKIP LOCKED`, un second travailleur bloquerait jusqu’à ce que la première transaction soit commitée avant de voir les mêmes lignes – à ce moment-là, elles seraient déjà marquées comme terminées. Avec `SKIP LOCKED`, le second travailleur récupère immédiatement des lignes différentes au lieu d’attendre, vous offrant une mise à l’échelle horizontale sûre.

Notez la séparation scan-puis-publication dans le code ci-dessus : toutes les lignes sont lues dans une tranche avant que la boucle de publication ne commence. Cela évite de maintenir un curseur `*sql.Rows` ouvert pendant les appels réseau vers le broker, ce qui tiendrait la transaction ouverte plus longtemps que nécessaire.

## Idempotence et déduplication

Le relais publie au moins une fois. S’il publie un événement puis plante avant de committer la mise à jour `processed_at`, il publiera à nouveau le même événement au redémarrage. C’est inévitable – la livraison exactement une fois entre une base de données et un broker sans un coordonnateur de transaction distribué nécessite ce compromis.

Les consommateurs doivent être idempotents. L’approche la plus simple est de suivre les IDs d’événements traités dans une table `processed_events` :

```
CREATE TABLE processed_events (
    event_id   UUID PRIMARY KEY,
    processed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

```
func (h *OrderHandler) HandleOrderCreated(ctx context.Context, eventID string, payload []byte) error {
    // Dédupliquer en utilisant l'ID de l'événement comme clé naturelle
    _, err := h.db.ExecContext(ctx, `
        INSERT INTO processed_events (event_id) VALUES ($1)
        ON CONFLICT (event_id) DO NOTHING
    `, eventID)
    if err != nil {
        return fmt.Errorf("dedup check: %w", err)
    }

    // Vérifier si l'insertion a vraiment eu lieu (1 ligne) ou était un no-op (0 lignes)
    // Une approche plus simple : utiliser RETURNING ou vérifier les lignes affectées
    // Si 0 lignes affectées, c'est un doublon -- le sauter
    ...
}
```

En pratique, de nombreuses équipes s’appuient sur les en-têtes de déduplication du broker lui-même (tels que le champ `key` de Kafka pour les topics à compactage de journal, ou l’en-tête `message-id` de RabbitMQ) et considèrent la déduplication au niveau de la base de données comme une solution de repli. Les deux sont des couches valides à appliquer.

Incluez l’`id` de l’événement de la boîte de sortie (un UUID) dans le message publié comme clé de déduplication. Les consommateurs peuvent alors l’utiliser quel que soit le mécanisme de déduplication qu’ils préfèrent.

## Politique de réessai et messages empoisonnés

La colonne `attempts` pilote la politique de réessai. Le relais saute les lignes où `attempts >= maxAttempts` et traite ces lignes comme des lettres mortes. Un processus séparé ou une alerte opérateur les gère.

Une vue simple pour les lettres mortes :

```
CREATE VIEW outbox_dead_letters AS
SELECT *
FROM outbox_events
WHERE attempts >= 5
  AND processed_at IS NULL
ORDER BY created_at;
```

Une bonne politique de réessai en production :

- Définir `maxAttempts` à 5-10 en fonction du coût des réessais.
- Envisager une attente exponentielle : inclure une colonne `retry_after` et sauter les lignes où `retry_after > NOW()`.
- Alerter lorsque `COUNT(*) FROM outbox_dead_letters` dépasse un seuil.
- Fournir un chemin de réessai manuel : un point de terminaison administrateur ou un script qui réinitialise `attempts = 0` et `retry_after = NULL` pour des lignes spécifiques.

Les messages empoisonnés – des lignes qui échouent systématiquement en raison d’un bug dans le consommateur ou d’une incompatibilité de schéma – ne doivent pas bloquer les messages sains. Comme le relais traite un lot par tick et marque les échecs avec une incrémentation de tentative plutôt que de les retirer de la file d’attente, les lignes saines progressent normalement tandis que les empoisonnées accumulent des tentatives jusqu’à atteindre le seuil de lettre morte.

## Ordonnancement des événements et partitionnement

La requête de sondage trie par `created_at`, ce qui donne un ordonnancement premier entré premier sorti au sein d’un lot. Pour la plupart des cas d’utilisation, cela suffit. Lorsque l’ordonnancement strict par entité est important – par exemple, s’assurer que `order.updated` n’est jamais publié avant `order.created` pour la même commande – vous avez besoin d’un ordonnancement par agrégat.

Ajoutez `aggregate_id` à la clause `ORDER BY` et utilisez-le comme clé de message lors de la publication dans un topic partitionné comme [Apache Kafka](https://www.glukhov.org/fr/data-infrastructure/stream-processing/apache-kafka/ "Apprenez Apache Kafka 4.2 rapidement avec tarball ou Docker, démarrez un broker KRaft local, maîtrisez les outils CLI de clé et exécutez des exemples pratiques de producteur, consommateur et Connect."). Kafka route tous les messages avec la même clé vers la même partition, et les partitions sont consommées dans l’ordre. Cela vous donne des garanties d’ordonnancement par agrégat sans ordonnancement global, qui nécessiterait une seule instance de relais.

```
ORDER BY aggregate_id, created_at
```

Pour les brokers qui ne prennent pas en charge l’ordonnancement partitionné (tels que les files d’attente AMQP de base), une instance unique de relais ou des vérifications d’ordonnancement au niveau de l’application dans le consommateur sont les alternatives pratiques.

## Réduire la latence de sondage avec LISTEN/NOTIFY

Un intervalle de sondage d’une seconde signifie une latence d’événement moyenne de 500 millisecondes. Pour la plupart des charges de travail, cela convient. Pour les cas où vous avez besoin d’une latence quasi nulle, le mécanisme `LISTEN/NOTIFY` de PostgreSQL permet au relais de se réveiller immédiatement lorsqu’une nouvelle ligne de boîte de sortie est insérée.

Ajoutez un déclencheur à la table de la boîte de sortie :

```
CREATE OR REPLACE FUNCTION notify_outbox_insert() RETURNS trigger AS $$
BEGIN
    PERFORM pg_notify('outbox_event', NEW.id::text);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER outbox_insert_notify
AFTER INSERT ON outbox_events
FOR EACH ROW EXECUTE FUNCTION notify_outbox_insert();
```

Dans le relais, écoutez le canal et réveillez-vous sur les notifications tout en conservant une solution de repli vers le sondage périodique :

```
func (r *OutboxRelay) Run(ctx context.Context) error {
    listener := pq.NewListener(r.dsn, 10*time.Second, time.Minute, nil)
    defer listener.Close()

    if err := listener.Listen("outbox_event"); err != nil {
        return fmt.Errorf("listen: %w", err)
    }

    ticker := time.NewTicker(5 * time.Second) // sondage de secours
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        case <-listener.Notify:
            if err := r.processBatch(ctx); err != nil {
                r.logger.Error("outbox batch failed (notify)", "err", err)
            }
        case <-ticker.C:
            if err := r.processBatch(ctx); err != nil {
                r.logger.Error("outbox batch failed (poll)", "err", err)
            }
        }
    }
}
```

Le minuteur de secours gère toute notification manquée lors d’un redémarrage du relais ou d’un problème réseau. Gardez l’intervalle de secours à quelques secondes plutôt qu’à des millisecondes – son rôle est la récupération, pas la faible latence.

## Observabilité : métriques, journaux et alertes

La boîte de sortie est de l’infrastructure. Traitez-la comme telle et instrumentez-la en conséquence.

**Métriques clés :**

```
var (
    outboxPublished = prometheus.NewCounter(prometheus.CounterOpts{
        Name: "outbox_events_published_total",
        Help: "Total outbox events successfully published.",
    })
    outboxFailed = prometheus.NewCounterVec(prometheus.CounterOpts{
        Name: "outbox_events_failed_total",
        Help: "Total outbox publish failures by event type.",
    }, []string{"event_type"})
    outboxPending = prometheus.NewGauge(prometheus.GaugeOpts{
        Name: "outbox_events_pending",
        Help: "Current number of unprocessed outbox events.",
    })
    outboxBatchDuration = prometheus.NewHistogram(prometheus.HistogramOpts{
        Name:    "outbox_batch_duration_seconds",
        Help:    "Duration of each outbox processing batch.",
        Buckets: prometheus.DefBuckets,
    })
)
```

**Actualisation du jauge :** exécutez une requête périodique pour garder `outbox_events_pending` précis :

```
SELECT COUNT(*) FROM outbox_events WHERE processed_at IS NULL;
```

**Seuils d’alerte à considérer :**

- `outbox_events_pending > 1000` pendant plus de deux minutes : le relais est en retard ou bloqué.
- `outbox_events_pending` en croissance monotone : le broker est hors ligne ou le relais a planté.
- Compteur de lettres mortes non nul : un bug de schéma ou de consommateur nécessite une investigation.
- `outbox_batch_duration_seconds p95 > 5s` : la base de données est lente ou la taille du lot est trop grande.

**Champs de journal structurés :** incluez `event_id`, `event_type`, `aggregate_id` et `attempt` dans chaque ligne de journal du relais. Ces champs vous permettent de corréler une publication échouée avec la ligne de boîte de sortie spécifique et la trace du consommateur en aval.

## Boîte de sortie vs file d’attente directe vs saga

Le motif de la boîte de sortie n’est pas l’outil approprié pour chaque problème de coordination. Voici la comparaison :

|Approche|Atomicité|Complexité|Quand utiliser|
|---|---|---|---|
|Publication directe|Aucune|Faible|Acceptable de perdre occasionnellement des événements|
|Boîte de sortie transactionnelle|Forte|Moyenne|Livraison fiable d’événements depuis un seul service|
|Motif Saga|Éventuelle|Élevée|Transactions multi-services s’étendant sur plusieurs bases de données|
|Commit en deux phases|Forte|Très élevée|Rarement pratique ; évité dans la plupart des systèmes distribués|

Le motif de la boîte de sortie garantit qu’un seul service émet de manière fiable des événements qui reflètent ses propres changements d’état. Il ne coordonne pas les changements d’état entre plusieurs services – c’est le rôle du [motif Saga](https://www.glukhov.org/fr/app-architecture/integration-patterns/saga-transactions-in-microservices/ "Guide complet de l'implémentation du motif Saga en Go pour les microservices. Apprenez l'orchestration vs chorégraphie, les stratégies de compensation, l'idempotence, le sourcing d'événements et les meilleures pratiques avec des exemples pratiques."). Le choix du broker – qu’il s’agisse de [RabbitMQ, SQS](https://www.glukhov.org/fr/data-infrastructure/messaging/rabbitmq-on-eks-vs-sqs/ "Comparaison des fonctionnalités et des coûts d'hébergement de Rabbitmq sur Eks vs Sqs"), ou de Kafka – est indépendant du motif de la boîte de sortie lui-même ; le relais publie dans le broker que votre système utilise.

Si vous construisez une saga, le motif de la boîte de sortie est toujours utile : chaque participant à la saga écrit son changement d’état local et son événement de saga dans une seule transaction en utilisant la boîte de sortie, puis l’orchestrateur de saga ou la chorégraphie lit ces événements de manière fiable.

## WAL-based CDC comme relais alternatif

Au lieu de sonder, vous pouvez suivre le journal d’avance d’écriture (WAL) de PostgreSQL et lire les insertions de la boîte de sortie directement depuis le flux de réplication. Des outils comme Debezium font cela. Les avantages sont une latence plus faible et aucune pression de verrouillage sur la table de la boîte de sortie. Les inconvénients sont la complexité opérationnelle, une slot de réplication PostgreSQL dédié et un service externe à exécuter et surveiller.

Pour la plupart des équipes, le relais de sondage décrit ci-dessus est le bon point de départ. Le suivi du WAL a du sens lorsque vous avez des taux d’insertion de boîte de sortie élevés (des dizaines de milliers par seconde), avez besoin d’une latence d’événement inférieure à 100 ms, ou exécutez déjà Debezium pour d’autres besoins de capture de changements.

## Intégration sqlc

Si vous utilisez [sqlc](https://www.glukhov.org/fr/app-architecture/data-access/comparing-go-orms-gorm-ent-bun-sqlc/ "Comparaison des ORMs Go pour PostgreSQL : GORM vs Ent vs Bun vs sqlc - avec des exemples de code") pour du code de base de données Go typé en sécurité, les requêtes de la boîte de sortie s’adaptent naturellement :

```
-- name: InsertOutboxEvent :exec
INSERT INTO outbox_events (aggregate_type, aggregate_id, event_type, payload)
VALUES (@aggregate_type, @aggregate_id, @event_type, @payload);

-- name: FetchOutboxBatch :many
SELECT id, aggregate_type, aggregate_id, event_type, payload
FROM outbox_events
WHERE processed_at IS NULL
  AND attempts < @max_attempts
ORDER BY created_at
LIMIT @batch_size
FOR UPDATE SKIP LOCKED;

-- name: MarkOutboxProcessed :exec
UPDATE outbox_events SET processed_at = NOW() WHERE id = @id;

-- name: IncrementOutboxAttempts :exec
UPDATE outbox_events SET attempts = attempts + 1 WHERE id = @id;

-- name: OutboxPendingCount :one
SELECT COUNT(*) FROM outbox_events WHERE processed_at IS NULL;
```

sqlc génère des fonctions typées en sécurité pour chaque requête, ce qui évite les erreurs d’interpolation de chaîne et garde la logique de requête de la boîte de sortie co-localisée avec le reste de votre couche d’accès à la base de données.

## Liste de contrôle de production

Utilisez ceci avant de livrer une implémentation de boîte de sortie :

### Base de données

- La table de la boîte de sortie possède l’index partiel sur `created_at WHERE processed_at IS NULL`
- La colonne `attempts` est présente avec une valeur par défaut de 0
- La vue ou la requête de lettres mortes est définie
- Les anciennes lignes traitées sont archivées ou supprimées périodiquement (un travail de nettoyage nocturne suffit)

### Relais

- `FOR UPDATE SKIP LOCKED` est utilisé dans la requête de sondage
- Le relais s’exécute à l’intérieur d’une transaction (début avant la requête, commit après toutes les mises à jour)
- La taille du lot est bornée (50-200 lignes est typique)
- Le relais respecte l’annulation du contexte pour l’arrêt élégant
- Les publications échouées incrémentent `attempts` plutôt que de provoquer l’arrêt du lot

### Idempotence

- Le message publié inclut l’`id` de la boîte de sortie comme clé de déduplication
- Les consommateurs sont idempotents ou le broker fournit la déduplication
- Consultez [Idempotence dans les systèmes distribués](https://www.glukhov.org/fr/app-architecture/integration-patterns/idempotency-in-distributed-systems/ "L'idempotence n'est pas une astuce HTTP. Apprenez à arrêter les écritures dupliquées, les messages rejoués et les doubles facturations à travers les API, les files d'attente, les webhooks et les workflows.") pour les motifs de déduplication

### Observabilité

- Le jauge `outbox_events_pending` est surveillé et alerté
- Le compteur de lettres mortes est alerté
- La durée du lot du relais est suivie
- Les journaux structurés incluent `event_id`, `event_type` et `aggregate_id`

### Opérations

- Un chemin de réessai manuel existe pour les lignes de lettres mortes
- Le comportement de redémarrage du relais est testé (republie-t-il correctement ?)
- Le comportement en cas de panne du broker est testé (la boîte de sortie grandit et se draine-t-elle correctement ?)

## Réflexions finales

Le problème de l’écriture double est facile à rejeter comme un cas extrême jusqu’à ce qu’il cause un incident. Le motif de la boîte de sortie transactionnelle le résout avec des outils que vous avez déjà : une transaction PostgreSQL, une goroutine en arrière-plan et une table supplémentaire. Le relais est simple à construire, simple à exploiter et simple à comprendre.

Le coût est que les consommateurs doivent être conçus pour une livraison au moins une fois. C’est un compromis raisonnable. La livraison exactement une fois entre une base de données et un broker sans transactions distribuées n’est pas réalisable en pratique – et prétendre le contraire conduit à des systèmes qui abandonnent ou traitent en double les événements silencieusement dans des conditions de défaillance.

Écrivez l’événement avec les données. Reliez-le de manière fiable. Rendez les consommateurs idempotents. C’est tout le motif.

Cet article fait partie du cluster [Architecture d’application en production](https://www.glukhov.org/fr/app-architecture/ "Pilier pratique d'architecture d'application pour les systèmes de production : motifs d'intégration basés sur le chat avec Slack et Discord, motifs de conception d'architecture propre Python, et compromis d'accès aux données Go à travers GORM, Ent, Bun et sqlc.").

## Sources

- [PostgreSQL FOR UPDATE SKIP LOCKED](https://www.postgresql.org/docs/current/sql-select.html#SQL-FOR-UPDATE-SHARE)
- [PostgreSQL LISTEN/NOTIFY](https://www.postgresql.org/docs/current/sql-listen.html)
- [Debezium outbox event router](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html)
- [Support LISTEN de lib/pq](https://pkg.go.dev/github.com/lib/pq#hdr-Notifications)
- [Package database/sql](https://pkg.go.dev/database/sql)