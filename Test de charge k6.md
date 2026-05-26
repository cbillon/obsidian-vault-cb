---
tags:
  - test
---
#### Sommaire

1. Pourquoi k6 ?
2. Premiers pas : écrire un test
3. Métriques et seuils de performance
4. Scénarios avancés et stages
5. Intégration Grafana et monitoring
6. Comparaison : k6 vs JMeter vs Gatling
7. Perspectives et ressources
8. Conclusion

---

#### Pourquoi k6 ?

k6, édité par Grafana Labs, est un **outil de test de charge open-source** conçu pour les DevOps et les développeurs modernes. Contrairement aux outils GUI lourds, k6 privilégie une **approche code-first** en JavaScript ES6 et possède une **philosophie CLI native**.

##### Points forts

- **Écrit en Go** → Perfs élevées, pas de dépendances JVM
- **Scripts en JavaScript** → Accessibilité pour les équipes web
- **Intégration Grafana/InfluxDB** → Dashboard et alertes en temps réel
- **k6 Cloud** → Tests distribués sur infrastructure Grafana
- **Extensions xk6** → Connecteurs custom (Kafka, MQTT, gRPC…)

Parfait pour valider la **résilience de votre infrastructure** sous charge, qu'elle soit on-premise, cloud ou hybride comme chez SHPV.

---

#### Premiers pas : écrire un test

##### Installation

```bash
# macOS
brew install k6

# Linux
sudo apt-get install k6

# Vérification
k6 version
```

##### Test basique

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
	stages: [
		{ duration: '30s', target: 10 }, // Ramp-up à 10 VUs
		{ duration: '2m', target: 10 }, // Steady à 10 VUs
		{ duration: '30s', target: 0 }, // Ramp-down
	],
	thresholds: {
		http_req_duration: ['p(95)<500'], // p95 < 500ms
		http_req_failed: ['rate<0.05'], // < 5% d'erreurs
	},
};

export default function () {
	const res = http.get('https://api.example.com/users');

	check(res, {
		'status 200': (r) => r.status === 200,
		'response time < 400ms': (r) => r.timings.duration < 400,
	});

	sleep(1);
}
```

##### Lancer le test

```bash
k6 run test.js

# Avec cloud
k6 run --cloud test.js
```

---

#### Métriques et seuils de performance

##### Métriques clés

|Métrique|Sens|
|---|---|
|**VUs**|Virtual Users actifs|
|**RPS**|Requêtes par seconde|
|**http_req_duration**|Latence requête (p50/p95/p99)|
|**http_req_failed**|Taux d'erreurs (4xx, 5xx, timeouts)|
|**group_duration**|Temps groupe (scénarios complexes)|

##### Thresholds (seuils)

```javascript
export const options = {
	thresholds: {
		http_req_duration: ['p(95)<500', 'p(99)<1000'],
		http_req_failed: ['rate<0.01'],
		checks: ['rate>0.95'], // Validations OK > 95%
	},
};
```

Si un seuil échoue, k6 sort avec code d'erreur → **CI/CD compatible**.

##### Checks (assertions)

```javascript
check(res, {
	'authentification OK': (r) => r.status === 200,
	'payload < 100KB': (r) => r.body.length < 100000,
	'header Date présent': (r) => r.headers['Date'] != null,
});
```

---

#### Scénarios avancés et stages

##### Stages : Ramp-up / Steady / Ramp-down

```javascript
export const options = {
	stages: [
		{ duration: '1m', target: 50 }, // Montée progressive
		{ duration: '5m', target: 50 }, // Charge stable
		{ duration: '30s', target: 100 }, // Pic surprise
		{ duration: '5m', target: 100 }, // Tenue du pic
		{ duration: '1m', target: 0 }, // Descente
	],
};
```

##### Scénarios et groupes

```javascript
import { group } from 'k6';

export default function () {
	group('Auth Flow', function () {
		const res = http.post('https://api.example.com/auth', {
			username: 'user@example.com',
			password: 'secret',
		});
		check(res, { 'login OK': (r) => r.status === 200 });
	});

	group('Dashboard', function () {
		const res = http.get('https://api.example.com/dashboard');
		check(res, { 'dashboard loaded': (r) => r.status === 200 });
	});

	sleep(2);
}
```

##### Tags (pour filtrer/regrouper)

```javascript
http.get('https://api.example.com/users', {
	tags: { name: 'GetUsers', category: 'api' },
});
```

---

#### Intégration Grafana et monitoring

##### Output JSON

```bash
k6 run test.js --out json=results.json
```

Exploitable en post-traitement ou versioning pour historique.

##### InfluxDB + Grafana

```bash
# k6 → InfluxDB
k6 run test.js \
  --out influxdb=http://localhost:8086/k6 \
  -e INFLUXDB_ORGANIZATION=myorg \
  -e INFLUXDB_BUCKET=k6
```

Dashboards Grafana pré-construits : latences, erreurs, VUs temps-réel.

##### xk6-prometheus

```bash
# Build avec xk6
xk6 build --with github.com/grafana/xk6-prometheus

# Test avec Prometheus
./k6 run test.js --out prometheus=http://localhost:9090
```

---

#### Comparaison : k6 vs JMeter vs Gatling

|Critère|k6|JMeter|Gatling|
|---|---|---|---|
|**Langage**|JavaScript|Java/XML GUI|Scala/Java|
|**Modèle**|Code-first, CLI|GUI, scriptable|Code-first|
|**Ressources**|Légères (Go)|Lourdes (JVM)|Modérées (JVM)|
|**Courbe d'apprentissage**|Basse|Moyenne|Haute|
|**Cloud distribué**|k6 Cloud|Plugin JMeter Cloud|Gatling Enterprise|
|**Support protocoles**|HTTP, WS, gRPC|HTTP, FTP, MQTT, JMS…|HTTP, WS, SSE|
|**Écosystème**|xk6 extensions|Plugins divers|Gatling Academy|

**k6 gagne** pour les équipes DevOps/Web modernes. **JMeter** pour beaucoup de protocoles. **Gatling** pour benchmarks poussés.

---

#### Perspectives et ressources

Approfondir la culture performance :

- **[Monitoring infrastructure avec Prometheus et Grafana](https://www.shpv.fr/blog/prometheus-grafana-alertmanager/)** → Couplage parfait avec k6 Cloud
- **[Linux performance : tuning kernel et OS](https://www.shpv.fr/blog/linux-performance/)** → Optimiser vos serveurs testés
- **[Caching HTTP : nginx, Redis, CDN](https://www.shpv.fr/blog/nginx-cache/)** → Réduire charge serveur pré-test

---

#### Sources

- [k6.io – Documentation officielle](https://k6.io/)
- [Grafana Docs – k6 Integrations](https://grafana.com/docs/k6/)
- [xk6 Extensions](https://grafana.com/docs/k6/latest/extensions/xk6/)

---

#### Conclusion

k6 est **l'outil de référence** pour valider la tenue de votre infrastructure sous charge réaliste. Ses tests en JavaScript, sa philosophie CLI et son intégration Grafana en font un choix **pragmatique et moderne**.

Que vous opériez une **infrastructure cloud, on-premise ou hybride** (comme SHPV), les tests de charge k6 vous permettent :

✓ **Valider vos SLA** (99.9 %+) ✓ **Détecter les goulots** avant la production ✓ **Intégrer dans la CI/CD** (thresholds automatiques) ✓ **Monitorer en temps réel** (Grafana)

N'attendez pas une panne : **testez votre performance aujourd'hui.**