[](https://www.shpv.fr/blog/capacity-planning)

Le capacity planning, c'est la différence entre une infrastructure qui sert votre croissance et une infrastructure qui l'étouffe. Trop souvent, les équipes fonctionnent en mode pompier : elles réagissent aux alertes saturation, aux appels des utilisateurs, aux incidents en prod. Résultat ? Pertes de données, dégradations de performance, frais de provisioning d'urgence.

À l'inverse, sur-provisionner sans données factuelles gaspille des budgets colossaux sur du hardware inutilisé. Entre surcoûts et sous-dimensionnement, la marge est mince. **Le capacity planning est votre GPS pour naviguer cette tension.**

---

#### Plan de l'article

1. Pourquoi le capacity planning est essentiel
2. Les 4 ressources à surveiller
3. Collecter les métriques (Prometheus, exporters, rétention)
4. Modéliser la croissance et les patterns saisonniers
5. Définir des seuils et alertes proactives
6. Outils et dashboards
7. Dimensionnement par service
8. FinOps et optimisation des coûts
9. Automatiser le scaling
10. Perspectives complémentaires et bonnes pratiques

---

#### Pourquoi le capacity planning est essentiel

##### Le coût de l'impréparation

Une **saturation CPU non anticipée** peut paralyser un cluster Kubernetes en quelques heures. Une **partition disque pleine** sans prévention = corruption de données. Un **RTO (Recovery Time Objective) explosé** parce que vous aviez 2 jours avant rupture de stock mémoire mais aucune visibilité.

Les études secteur montrent qu'une **panne non prévue** coûte 10 à 50 fois plus cher qu'un investissement proactif en capacity planning. Ajoutez les délais de livraison matériel (2-4 semaines pour serveurs dédiés), et vous comprenez que **l'anticipation n'est pas un luxe, c'est une stratégie de survie.**

##### L'autre piège : over-provisioning coûteux

Provisionner "avec marge" sans mesure, c'est accepter des coûts cloud ou matériel qui décuplent. Une startup avec 10 serveurs surdimensionnés peut perdre 30-40 % de son CAPEX chaque année. **Une bonne méthodologie élimine le gaspillage tout en sécurisant la croissance.**

---

#### Les 4 ressources à surveiller : la méthode USE

Pour ne rien oublier, utilisez le framework **USE (Utilization, Saturation, Errors)** popularisé par Brendan Gregg.

##### 1. **CPU**

- **Utilization** : Pourcentage de temps où les cores traitent une tâche
- **Saturation** : Profondeur de queue (processus en attente)
- **Errors** : Throttling, thermal issues

**Seuil d'alerte prudent** : 80 % utilisation moyenne, 90 % saturation CPU → provisionner dans les 30 jours.

##### 2. **RAM**

- **Utilization** : % de mémoire allouée
- **Saturation** : Swapping intense (I/O de pages)
- **Errors** : OOM kills, allocation failures

**Critique** : Le swap n'est **pas une buffer zone**. Si vous touchez le swap en prod, vous avez déjà un problème. Anticipez.

##### 3. **Stockage disque**

- **Utilization** : Espace utilisé / total
- **Saturation** : Latence I/O élevée, full capacity
- **Errors** : Write failures, inodes pleins

**Seuil strict** : 80 % = alarme rouge. Le disque plein est l'ennemi juré des databases.

##### 4. **Réseau (bande passante)**

- **Utilization** : Débit actuel / capacité de l'interface
- **Saturation** : Packet loss, retransmissions
- **Errors** : Collisions, CRC errors

**Contexte** : SHPV opère un backbone AS41652 dimensionné à 150 Gbps avec 500+ peerings directs, ce qui permet une facturation prévisible sans surcharge imprévue.

---

#### Collecter les métriques : fondation du capacity planning

##### Prometheus + node_exporter

La plupart des teams utilisent **Prometheus** pour scraper les métriques à intervalle régulier (par défaut 15s).

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  retention: 15d # Critique pour le trending !

scrape_configs:
  - job_name: 'nodes'
    static_configs:
      - targets: ['localhost:9100'] # node_exporter
```

**node_exporter** expose les métriques du système :

```bash
# Installation sur Linux
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-*.linux-amd64.tar.gz
./node_exporter
```

Métriques clés à tracker :

```
node_cpu_seconds_total            # CPU usage
node_memory_MemAvailable_bytes    # RAM disponible
node_filesystem_avail_bytes       # Espace disque libre
node_network_transmit_bytes_total # Débit réseau
```

##### Rétention et trending

**Règle d'or** : Conservation minimum 15 jours de data brute, 1 an agrégée (downsampling).

Avec seulement 7 jours d'historique, vous ratez les patterns **saisonniers hebdomadaires** (pic le lundi) ou les **croissances graduelles** (tendance mensuelle). 15 jours = minimum viable.

##### Métriques applicatif

Au-delà du système, instrumentez vos apps :

```go
// Exemple Go avec Prometheus client library
import "github.com/prometheus/client_golang/prometheus"

requestCounter := prometheus.NewCounterVec(
    prometheus.CounterOpts{Name: "http_requests_total"},
    []string{"method", "endpoint"},
)

// Incrémenter lors de chaque requête
requestCounter.WithLabelValues("GET", "/api/users").Inc()
```

---

#### Modéliser la croissance et les patterns

##### Regression linéaire simple

Si votre CPU passe de 40 % il y a 30 jours à 65 % aujourd'hui, la **pente est positive**. Extrapolez :

```
Y = A + (B × jours)
Saturation = 90% dans ~20 jours → provisionnez maintenant
```

**Prometheus** supporte `predict_linear()` :

```promql
predict_linear(node_cpu_usage_percent[30d], 7*24*3600)
# Prédiction CPU dans 7 jours basée sur tendance 30j
```

##### Patterns saisonniers

- **Trafic e-commerce** : pics massifs décembre, été plat
- **SaaS B2B** : pics lundi-jeudi matin, creux week-end
- **Vidéo streaming** : pointe soirée 19h-23h

Décomposez vos données brutes avec des outils STL (Seasonal and Trend decomposition using Loess) pour isoler la tendance réelle.

##### Projections métier

Les meilleurs modèles de capacity planning combinent :

1. Données historiques (trending automatisé)
2. Projections commerciales ("on passe de 100k à 500k users en Q3")
3. Cycles saisonniers connus

Réunion trimestrielle engineering + product + finance, avec une feuille de calcul partagée = meilleure prédictabilité.

---

#### Seuils et alertes proactives

##### Hiérarchie d'alertes

|Seuil|Action|Délai|
|---|---|---|
|70 % utilisation|Info (monitoring)|-|
|80 % utilisation|Warning (ticket)|7-14 jours|
|90 % utilisation|Critical (escalade)|2-3 jours|
|98 %+|Incapacité à servir|Immédiat|

##### Alertes trend-based

Au lieu d'alerter sur un seuil statique, alertez sur la **trajectoire** :

```yaml
# Alertmanager : "disk full dans 7 jours"
- alert: DiskFullProjection
  expr: |
    predict_linear(node_filesystem_avail_bytes[7d], 7*24*3600) < 0
  for: 1h
  annotations:
    summary: 'Disque {{ $labels.device }} plein dans ~7 jours'
    runbook: 'Croissance stockage anormale, vérifier bloat ou rétention logs'
```

---

#### Outils et dashboards de capacity planning

##### Grafana

Créez un dashboard dédié avec :

- **Graphiques trending** (30-90j) pour chaque ressource
- **Prédictions** (predict_linear)
- **Cartes de heat** affichant utilisation par instance
- **Tableau de capacité résiduelle** (jours avant saturation par métrique)

```json
{
	"title": "Capacity Planning Dashboard",
	"panels": [
		{
			"title": "CPU Projection (30j)",
			"targets": [
				{
					"expr": "predict_linear(node_cpu_usage_percent[30d], 30*24*3600)",
					"legendFormat": "{{ instance }}"
				}
			]
		}
	]
}
```

##### Spreadsheets & FinOps

Beaucoup de teams utilisent des **Google Sheets / Excel** comme source de vérité, complétés par des exports Prometheus. C'est old school mais **ça marche** : flexibilité, collaboration, calculs custom.

---

#### Dimensionnement par service

##### Web servers (nginx, Apache)

**Formula** : Requêtes par seconde × latence moyenne (en secondes) = connexions concurrentes

```
RPS = 1000 req/s
Latence moy = 0.1s
→ Connexions concurrentes = 100

1 server nginx = ~1000 req/s
Donc : 1 instance suffit, mais pour HA : 2 × 1000 req/s = capacité 2000 req/s
```

##### Databases

- **CPU** : Proportionnel aux requêtes complexes (JOINs, agrégations)
- **RAM** : **Critique**, cache index + working set. Rule of thumb : RAM ≥ 80 % de dataset actif
- **Storage** : Croissance log (inserts/updates) + backups (2-5× taille data brute)

**PostgreSQL** :

```sql
-- Estimation taille table
SELECT
  schemaname, tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname != 'pg_catalog'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

##### Kubernetes clusters

- **Per pod** : 10-100 mCPU (apps) + 128-512 Mi RAM
- **Per node** : Réserver 10 % CPU + 100 Mi RAM pour kubelet, kube-proxy
- **Overhead** : ETCD (5 nodes en HA) + CoreDNS + monitoring

**Formule rapide** : Pour N pods en HA (3 repliques chacun) × 50 mCPU average = CPU requis, puis over-provision de 20-30 %.

##### Storage (block, object, file)

- **Croissance historique** : Si +20 TB/mois, vous avez ~6 mois avant rupture
- **Snapshots/backups** : Multiplier espace brut par 2-5×
- **Garbage collection** : Logs applicatifs, caches expirant = croissance "cachée"

---

#### FinOps et optimisation des coûts

##### Right-sizing

**Objectif** : Chaque instance = ressources utilisées + 20-30 % marge, pas plus.

```
Instance oversized : 16 vCPU / 64 GB RAM, utilisation moyenne 2 vCPU / 8 GB
Action : Downsizer à 4 vCPU / 16 GB
Économie : 60-70% du coût instance
```

##### Reserved capacity vs on-demand

- **Workload stable** (web servers, databases) : 80-90 % reserved instances (36 mois) = -60 % coût
- **Workload variable** (batch jobs) : 100 % on-demand ou spot (-70 % vs on-demand)
- **Mix recommandé** : 70 % reserved + 30 % on-demand pour flexibilité

##### Quotas et limite d'allocation

Imposez des quotas par équipe/service pour éviter des "surprise" provisioning :

```yaml
# Kubernetes ResourceQuota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: engineering-quota
spec:
  hard:
    requests.cpu: '100'
    requests.memory: '200Gi'
    pods: '500'
```

---

#### Automatiser le scaling

##### Horizontal Pod Autoscaler (Kubernetes)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

**Avantage** : HPA réagit aux pics inattendus. Capacity planning donne les limites min/max.

##### Vertical Pod Autoscaler (VPA)

VPA analyse l'historique et recommande des ajustements de `requests/limits`. Complément utile au capacity planning : il détecte les over-allocations au niveau du pod.

##### Auto Scaling Groups (cloud)

```yaml
# AWS ASG
MinSize: 3
MaxSize: 10
DesiredCapacity: 5
TargetTrackingScalingPolicy:
  TargetValue: 75 # CPU target %
```

**Règle** : Capacity planning fixe min/max ; autoscaling gère les fluctuations intraday.

---

#### Perspectives complémentaires

##### Monitoring & alerting avancé

Pour aller plus loin, consultez notre guide [Prometheus, Grafana et Alertmanager en production](https://www.shpv.fr/blog/prometheus-grafana-alertmanager/) qui couvre la setup éprouvé des métriques.

##### FinOps en détail

L'article [FinOps : maîtriser les coûts d'infrastructure](https://www.shpv.fr/blog/finops-production/) détaille la gouvernance budgétaire et les mécanismes de chargeback.

##### Observabilité Kubernetes

Capacity planning en Kubernetes nécessite une excellente observabilité. Voir [Monitoring Kubernetes à grande échelle](https://www.shpv.fr/blog/kubernetes-monitoring/) pour les patterns de scaling automatisé.

##### SLOs et budgets d'erreurs

Enfin, lier le capacity planning aux **Service Level Objectives** : un SLA 99.9 % exige une marge de capacité différente d'un SLA 99 %. Détails dans [SLO, SLI et error budgets](https://www.shpv.fr/blog/slo-sli-error-budget/).

---

#### Perspectives : le capacity planning au-delà des métriques

Le capacity planning n'est pas qu'une question technique. C'est une **conversation continue** entre engineering et finance.

- **Quarterly business review** (QBR) : Partager trends, projections, budgets demandés
- **Incident post-mortem** : "Pourquoi cette saturation n'a-t-elle pas été prédite ?"
- **Chargebacks internes** : Facturer aux équipes leur part de l'infrastructure pousse à l'optimisation

---

#### Sources

- Brendan Gregg, "Systems Performance: Enterprise and the Cloud" (2nd ed., 2020)
- "Prometheus: The Definitive Guide" par Julien Pivotto et Björn Rabenstein (2021)
- Google Cloud, "Right Sizing Recommendations" (2024) - [https://cloud.google.com/docs/onboarding-checklist/compute#right-sizing](https://cloud.google.com/docs/onboarding-checklist/compute#right-sizing)
- "FinOps Foundation Handbook" (2023) - [https://www.finops.org/framework/](https://www.finops.org/framework/)
- Kubernetes official docs, "Horizontal Pod Autoscaler" - [https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- "How To Achieve Cost and Performance at Scale" par Emily Omier (Forrester, 2023)

---

#### Conclusion

Le capacity planning n'est pas du luxe de grandes orga. Même une PME de 10 personnes avec 5-10 serveurs gagne à :

1. **Tracer ses métriques** (Prometheus, exporters)
2. **Définir des seuils clairs** (warning, critical)
3. **Modéliser la croissance** (trending, saisonnalité)
4. **Provisionner proactivement** (30-90 jours d'avance)

Chez **SHPV**, où nous gérons depuis 2012 des infrastructures critiques avec un SLA 99.9 %+, cette discipline est le pilier de la fiabilité. Elle permet à nos clients de **grandir sans peur de la saturation**, tout en maîtrisant les coûts.

Une infrastructure bien dimensionnée, c'est une infrastructure qui **vieillit gracieusement**.

Pour discuter de votre stratégie capacity planning ou explorer nos services d'infogérance, [contactez-nous](https://www.shpv.fr/services/infogerance/), nos équipes sont à votre écoute.