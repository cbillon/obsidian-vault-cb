---
link: https://www.glukhov.org/fr/post/2026/01/most-popular-go-projects-on-github/
byline: À propos de Rost Glukhov
site: Rost Glukhov | Site personnel et blog technique
excerpt: Découvrez les projets Go les plus populaires sur GitHub ce mois-ci,
  classés par le nombre d'étoiles gagnées. Des agents de codage IA à la gestion
  de Docker, des applications auto-hébergées aux passerelles LLM - aperçu
  complet avec statistiques, licences et cas d'utilisation.
slurped: 2026-03-22T10:44
title: Top 19 des projets Go tendance sur GitHub - Janvier 2026
---

L’écosystème Go continue de prospérer avec des projets innovants couvrant l’outillage IA, les applications auto-hébergées et l’infrastructure développeur. Ce panorama analyse les [dépôts Go les plus tendance sur GitHub](https://www.glukhov.org/fr/post/2026/01/most-popular-go-projects-on-github/ "top trending Go repositories on GitHub") ce mois-ci.

Si vous débutez avec Go, consultez notre [Fiche Mémo Go](https://www.glukhov.org/fr/post/2022/golang-cheatsheet/ "Golang Cheat sheet and most useful commands") pour un aperçu rapide du langage.
## Aperçu

Basé sur les données de la [page des tendances GitHub](https://github.com/trending/go?since=monthly), voici les 19 projets Go à la croissance la plus active ce mois-ci. Chaque entrée inclut le nombre total d’étoiles, la croissance mensuelle, la licence et une description de ce qui rend le projet remarquable.

---

## 1. Memos — 8 696 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[usememos/memos](https://github.com/usememos/memos)|
|**Étoiles totales**|56 104|
|**Licence**|MIT|
|**Catégorie**|Notes auto-hébergées|

La demande croissante pour des alternatives respectueuses de la vie privée à des services comme Notion et Google Keep, combinée aux faibles exigences en ressources de Memos, a entraîné une croissance explosive.

**Memos** est un service de prise de notes léger et auto-hébergé axé sur la confidentialité et la simplicité. Contrairement aux alternatives basées sur le cloud, vos pensées et données restent entièrement sous votre contrôle — pas de suivi, pas de publicités, pas de frais d’abonnement.

**Fonctionnalités clés :**

- Support Markdown avec édition de texte enrichi
- Organisation et filtrage par tags
- API REST pour les intégrations
- Déploiement en binaire unique avec SQLite
- Support Docker et Kubernetes

---

## 2. Beads — 6 839 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[steveyegge/beads](https://github.com/steveyegge/beads)|
|**Étoiles totales**|13 498|
|**Licence**|Apache-2.0|
|**Catégorie**|Outils de codage IA|

Alors que les assistants de codage IA deviennent courants, la nécessité d’une mémoire persistante entre les sessions et les branches est devenue cruciale. Beads comble élégamment cette lacune.

**Beads** est un système de mémoire basé sur git pour les agents de codage IA créé par Steve Yegge. Il résout un problème fondamental : les agents IA oublient le contexte lorsque l’historique de conversation s’allonge ou lors du changement de branches de code. Si vous travaillez avec des flux de travail basés sur git, notre [fiche mémo des commandes GIT](https://www.glukhov.org/fr/developer-tools/git-and-forges/git-cheatsheet/ "GIT commands cheatsheet") couvre les commandes essentielles dont vous aurez besoin.

**Fonctionnalités clés :**

- Stocke les tâches, plans et dépendances des agents sous forme de fichiers JSONL dans le répertoire `.beads/`
- Git devient la couche de persistance — branchez le code, branchez le contexte
- Identifiants basés sur des hachages pour éviter les collisions dans les flux de travail multi-agents
- Exécution consciente des dépendances avec la commande `bd ready`
- Système de formules pour les modèles de flux de travail déclaratifs

---

## 3. Ollama — 2 966 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[ollama/ollama](https://github.com/ollama/ollama)|
|**Étoiles totales**|161 004|
|**Licence**|MIT|
|**Catégorie**|Runtime LLM|

La dynamique continue du mouvement IA local et les mises à jour régulières ajoutant le support de nouveaux modèles comme DeepSeek et GLM-4.7 maintiennent Ollama à l’avant-garde des outils LLM locaux.

**Ollama** est la manière la plus populaire d’exécuter des grands modèles de langage localement. Avec le support de GLM-4.7, DeepSeek, Qwen, Gemma, Llama et des centaines d’autres modèles, il fournit une CLI et une API simples pour l’inférence IA locale. Si vous souhaitez exécuter des LLMs sur votre machine sans envoyer de données vers le cloud, Ollama rend cela simple — consultez notre [fiche mémo Ollama](https://www.glukhov.org/fr/llm-hosting/ollama/ollama-cheatsheet/ "Ollama cheatsheet") pour des commandes et conseils de configuration.

**Fonctionnalités clés :**

- Téléchargement de modèles en une commande (`ollama run llama3`)
- Point de terminaison API compatible OpenAI
- Gestion multi-modèles
- Support d’accélération GPU
- Personnalisation des modelfiles

---

## 4. Crush — 2 745 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[charmbracelet/crush](https://github.com/charmbracelet/crush)|
|**Étoiles totales**|19 107|
|**Licence**|MIT|
|**Catégorie**|Outils de codage IA|

La combinaison de la réputation de Charmbracelet pour ses outils terminaux esthétiques et de la tendance des agents de codage IA a entraîné une adoption rapide. Pour une comparaison avec les alternatives basées sur le cloud, consultez notre [fiche mémo GitHub Copilot](https://www.glukhov.org/fr/ai-devtools/github-copilot-cheatsheet/ "GitHub Copilot Cheatsheet").

**Crush** est le “codage agentique glamour pour tous” — un assistant de codage IA basé sur le terminal de Charmbracelet. Construit avec Bubble Tea, il fournit une interface TUI élégante pour interagir avec les LLMs afin de générer, refactoriser et déboguer du code.

**Fonctionnalités clés :**

- Interface native terminal avec un style riche
- Support multi-fournisseurs LLM
- Génération et refactorisation de code
- Construit avec le kit d’outils TUI de Charmbracelet
- Assistance de codage consciente du contexte

---

## 5. WeKnora — 2 226 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[Tencent/WeKnora](https://github.com/Tencent/WeKnora)|
|**Étoiles totales**|12 634|
|**Licence**|Apache-2.0|
|**Catégorie**|RAG/IA Documentaire|

La demande des entreprises pour des solutions RAG fonctionnant avec des documents internes, combinée au soutien de Tencent et à sa gamme complète de fonctionnalités, a propulsé la croissance de WeKnora.

**WeKnora** est le framework open-source de Tencent pour la compréhension approfondie des documents et la génération augmentée par récupération (RAG). Il transforme les piles de documents en bases de connaissances interrogeables avec des capacités de recherche sémantique.

**Fonctionnalités clés :**

- Traitement multimodal des documents (PDF, Word, images)
- Récupération hybride : BM25 + vecteur + graphe de connaissances
- Architecture modulaire à quatre couches
- Tableau de bord web et API REST
- Modes de déploiement Docker, développement et Kubernetes

---

## 6. Keploy — 1 736 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[keploy/keploy](https://github.com/keploy/keploy)|
|**Étoiles totales**|15 585|
|**Licence**|Apache-2.0|
|**Catégorie**|Tests|

Le mouvement de décalage à gauche des tests et le désir de réduire les efforts de rédaction manuelle des tests ont rendu la génération automatique des tests de plus en plus attrayante.

**Keploy** est un agent de test API, d’intégration et E2E qui génère des tests et des mocks à partir du trafic API réel. Au lieu d’écrire manuellement des cas de test, Keploy capture les interactions réseau réelles et les rejoue.

**Fonctionnalités clés :**

- Génération automatique de tests à partir des appels API
- Génération de mocks/stubs pour les dépendances
- Support des langages : Go, Java, Node.js, Python
- Intégration avec go-test, JUnit et autres frameworks
- Suivi combiné de la couverture avec les tests unitaires

---

## 7. res-downloader — 1 687 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[putyy/res-downloader](https://github.com/putyy/res-downloader)|
|**Étoiles totales**|14 651|
|**Licence**|MIT|
|**Catégorie**|Téléchargeur multimédia|

**res-downloader** est une application de bureau pour télécharger des médias depuis diverses plateformes chinoises, y compris les chaînes vidéo WeChat, Douyin, Kuaishou, Xiaohongshu, les diffusions en direct, les fichiers m3u8 et les services de musique comme Kugou et QQ Music.

**Fonctionnalités clés :**

- Support multi-plateforme (Windows, macOS, Linux)
- Construit avec Go et le framework Wails
- Support vidéo, audio et images
- Téléchargement de flux m3u8/HLS
- Fonctionnalités de filtrage et de recherche

---

## 8. Arcane — 1 245 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[getarcaneapp/arcane](https://github.com/getarcaneapp/arcane)|
|**Étoiles totales**|4 165|
|**Licence**|BSD-3-Clause|
|**Catégorie**|Gestion Docker|

La demande croissante pour des outils de gestion Docker conviviaux alors que la conteneurisation devient courante au-delà des spécialistes DevOps a alimenté la montée rapide d’Arcane.

**Arcane** est une plateforme de gestion Docker moderne avec une interface web élégante conçue pour rendre la gestion des conteneurs accessible à tous — pas seulement aux experts de la ligne de commande. Si vous cherchez des alternatives à Portainer, Arcane offre une approche fraîche.

**Fonctionnalités clés :**

- Gestion du cycle de vie des conteneurs (démarrer, arrêter, redémarrer, inspecter)
- Gestion des images, volumes et réseaux
- Surveillance des ressources en temps réel avec graphiques
- API REST avec documentation OpenAPI 3.1
- Support Docker Compose

---

## 9. Seanime — 1 197 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[5rahim/seanime](https://github.com/5rahim/seanime)|
|**Étoiles totales**|2 559|
|**Licence**|GPL-3.0|
|**Catégorie**|Serveur multimédia|

Le désir de la communauté anime pour un serveur multimédia dédié qui comprend les conventions de nommage anime et s’intègre avec les services de suivi a fait de Seanime un projet remarquable.

**Seanime** est un serveur multimédia open-source spécifiquement conçu pour l’anime et le manga. Il analyse vos fichiers vidéo locaux, les organise automatiquement avec des métadonnées d’AniList et d’AniDB, et fournit une expérience de visionnage soignée.

**Fonctionnalités clés :**

- Gestion spécifique à l’anime (saisons, épisodes, parties multiples)
- Enrichissement automatique des métadonnées et des œuvres d’art
- Lecteur vidéo intégré avec support des sous-titres
- Synchronisation AniList pour le suivi des épisodes regardés
- Interface web et application de bureau Electron

---

## 10. BubbleTea — 1 169 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[charmbracelet/bubbletea](https://github.com/charmbracelet/bubbletea)|
|**Étoiles totales**|38 879|
|**Licence**|MIT|
|**Catégorie**|Framework TUI|

La croissance continue alors que plus de développeurs construisent des applications terminales et que l’écosystème Charmbracelet s’étend maintient BubbleTea en tendance.

**BubbleTea** est le puissant framework TUI (Terminal User Interface) qui alimente de nombreuses applications terminales élégantes, y compris Crush. Basé sur The Elm Architecture, il fournit une approche fonctionnelle pour construire des programmes terminaux interactifs.

**Fonctionnalités clés :**

- Architecture Elm (Modèle-Mise à jour-Vue)
- Composants composables via la bibliothèque Bubbles
- Style riche avec Lip Gloss
- Support de la souris
- Mises en page réactives

---

## 11. go2rtc — 1 063 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[AlexxIT/go2rtc](https://github.com/AlexxIT/go2rtc)|
|**Étoiles totales**|12 051|
|**Licence**|MIT|
|**Catégorie**|Streaming|

L’adoption des maisons intelligentes et le besoin d’un streaming de caméra local efficace sans services cloud alimentent la popularité de go2rtc.

**go2rtc** est l’application de streaming de caméra ultime, agissant comme un traducteur de protocole universel entre RTSP, RTMP, HTTP-FLV, WebRTC, MSE, HLS, MP4, MJPEG et HomeKit.

**Fonctionnalités clés :**

- Conversion multi-protocole (RTSP vers WebRTC avec ~0,5s de latence)
- Léger — fonctionne sur Raspberry Pi
- Pas de dépendance au cloud
- Intégration avec Home Assistant et Frigate NVR
- Support du transcodage matériel (Intel, AMD, Nvidia)

## 12. NetBird — 960 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[netbirdio/netbird](https://github.com/netbirdio/netbird)|
|**Étoiles totales**|21 346|
|**Licence**|BSD-3-Clause|
|**Catégorie**|Réseau/VPN|

La demande croissante des entreprises pour des solutions de réseau zéro confiance auto-hébergées continue de stimuler la croissance de NetBird.

**NetBird** crée des réseaux superposés sécurisés basés sur WireGuard avec des fonctionnalités d’entreprise comme l’SSO, la MFA et des contrôles d’accès granulaires. C’est une alternative auto-hébergée à Tailscale.

**Fonctionnalités clés :**

- Réseau maillé WireGuard
- Intégration SSO et MFA
- Accès réseau zéro confiance
- Traversée NAT
- Tableau de bord de gestion

---

## 13. go-stock — 634 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[ArvinLovegood/go-stock](https://github.com/ArvinLovegood/go-stock)|
|**Étoiles totales**|4 164|
|**Licence**|MIT|
|**Catégorie**|Finance/IA|

**go-stock** est un outil d’analyse boursière alimenté par l’IA prenant en charge les marchés A, Hong Kong et américain. Il combine les données de marché avec l’analyse LLM pour les sentiments et les perspectives financières.

**Fonctionnalités clés :**

- Prise en charge multi-marchés (A, HK, US)
- Analyse de sentiment IA
- Notifications d’alerte de prix
- Prise en charge de DeepSeek, OpenAI, Ollama, et plus
- Stockage des données localement

---

## 14. wx_channels_download — 573 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[ltaoo/wx_channels_download](https://github.com/ltaoo/wx_channels_download)|
|**Étoiles totales**|4 454|
|**Licence**|MIT|
|**Catégorie**|Téléchargeur de médias|

Un téléchargeur de vidéos pour les Chaînes WeChat, permettant de sauvegarder des vidéos depuis la plateforme de médias sociaux chinoise populaire.

---

## 15. GitHub CLI — 525 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[cli/cli](https://github.com/cli/cli)|
|**Étoiles totales**|42 257|
|**Licence**|MIT|
|**Catégorie**|Outils de développement|

**GitHub CLI** (`gh`) est l’outil en ligne de commande officiel de GitHub pour gérer les dépôts, les demandes de tirage, les problèmes et les GitHub Actions depuis le terminal. Pour automatiser vos flux de travail, consultez notre [fiche mémo GitHub Actions](https://www.glukhov.org/fr/developer-tools/ci-cd/github-actions-cheatsheet/ "Fiche mémo GitHub Actions").

**Fonctionnalités clés :**

- Gestion des PR et des problèmes
- Contrôle des flux de travail GitHub Actions
- Opérations sur les dépôts
- Prise en charge des Codespaces
- Écosystème d’extensions

---

## 16. Bifrost — 483 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[maximhq/bifrost](https://github.com/maximhq/bifrost)|
|**Étoiles totales**|1 922|
|**Licence**|Apache-2.0|
|**Catégorie**|Infrastructure LLM|

À mesure que l’adoption des LLM se généralise, le besoin de passerelles haute performance capables de gérer le trafic de production devient critique.

**Bifrost** est la passerelle LLM la plus rapide — prétendument 50 fois plus rapide que LiteLLM — avec équilibrage de charge adaptatif, mode cluster, garde-fous et prise en charge de 1 000+ modèles avec un overhead inférieur à 100 µs.

**Fonctionnalités clés :**

- Overhead interne <15 µs à 5 000 RPS
- Équilibrage de charge adaptatif entre fournisseurs
- Basculement automatique et réessais
- Métriques Prometheus et OpenTelemetry
- Clés virtuelles avec budgets et limites de débit

---

## 17. Semantic Router — 430 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[vllm-project/semantic-router](https://github.com/vllm-project/semantic-router)|
|**Étoiles totales**|3 023|
|**Licence**|Apache-2.0|
|**Catégorie**|Infrastructure LLM|

Les organisations exploitant plusieurs LLM ont besoin d’un routage intelligent pour optimiser le coût et la qualité, ce qui rend le Semantic Router de plus en plus pertinent.

**Semantic Router** est le système de routage intelligent de vLLM pour les déploiements de Mixture-of-Models. Il route automatiquement les requêtes vers le modèle le plus adapté en fonction de la classification sémantique, améliorant ainsi la précision tout en réduisant les coûts. Pour configurer le backend vLLM qui alimente ce routage, consultez notre [guide de démarrage rapide vLLM](https://www.glukhov.org/fr/llm-hosting/vllm/vllm-quickstart/ "Guide complet de configuration vLLM").

**Fonctionnalités clés :**

- Route automatiquement les requêtes mathématiques, de code, créatives et générales
- Détection des PII et prévention des contournements
- Mise en cache sémantique pour réduire les tokens
- Fonctionne sur CPU (pas de GPU requis)
- Natif Kubernetes avec intégration Envoy

---

## 18. Cilium — 427 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[cilium/cilium](https://github.com/cilium/cilium)|
|**Étoiles totales**|23 564|
|**Licence**|Apache-2.0|
|**Catégorie**|Réseau Kubernetes|

L’adoption continue des entreprises Kubernetes et le passage à une infrastructure basée sur eBPF maintiennent la croissance régulière de Cilium.

**Cilium** fournit un réseau, une sécurité et une observabilité basés sur eBPF pour les applications cloud natives. C’est le CNI (Container Network Interface) leader pour les environnements Kubernetes nécessitant des politiques réseau avancées.

**Fonctionnalités clés :**

- Chemin de données alimenté par eBPF pour une haute performance
- Politiques réseau L3-L7
- Capacités de service mesh
- Observabilité Hubble
- XDP pour la protection contre les DDoS

---

## 19. Listmonk — 399 ⭐

|Métrique|Valeur|
|---|---|
|**Dépôt**|[knadh/listmonk](https://github.com/knadh/listmonk)|
|**Étoiles totales**|18 890|
|**Licence**|AGPL-3.0|
|**Catégorie**|Email auto-hébergé|

L’intérêt croissant pour les alternatives auto-hébergées à Mailchimp et autres plateformes de marketing par email a boosté la visibilité de Listmonk.

**Listmonk** est un gestionnaire de newsletters et de listes de diffusion auto-hébergé haute performance. Il est distribué sous forme de binaire unique avec PostgreSQL comme seule dépendance.

**Fonctionnalités clés :**

- Gérer des millions d’abonnés
- Segmentation basée sur SQL
- Analytics de campagnes et suivi des rebonds
- API d’email transactionnel
- Authentification à deux facteurs TOTP

---

## Tableau récapitulatif

|Rang|Projet|Étoiles/Mois|Étoiles totales|Catégorie|
|---|---|---|---|---|
|1|Memos|8 696|56 104|Notes auto-hébergées|
|2|Beads|6 839|13 498|Mémoire de codage IA|
|3|Ollama|2 966|161 004|Runtime LLM|
|4|Crush|2 745|19 107|Agent de codage IA|
|5|WeKnora|2 226|12 634|Framework RAG|
|6|Keploy|1 736|15 585|Tests API|
|7|res-downloader|1 687|14 651|Téléchargeur de médias|
|8|Arcane|1 245|4 165|Gestion Docker|
|9|Seanime|1 197|2 559|Serveur de médias anime|
|10|BubbleTea|1 169|38 879|Framework TUI|
|11|go2rtc|1 063|12 051|Streaming caméra|
|12|NetBird|960|21 346|VPN WireGuard|
|13|go-stock|634|4 164|Analyse boursière IA|
|14|wx_channels_download|573|4 454|Téléchargeur de vidéos|
|15|GitHub CLI|525|42 257|Outils de développement|
|16|Bifrost|483|1 922|Passerelle LLM|
|17|Semantic Router|430|3 023|Routage LLM|
|18|Cilium|427|23 564|Réseau K8s|
|19|Listmonk|399|18 890|Gestionnaire de newsletters|

---

## Tendances clés

**Les outils de codage IA dominent :** Cinq des dix premiers projets (Beads, Ollama, Crush, WeKnora, Keploy) sont liés au développement IA/LLM, reflétant la concentration de l’industrie sur les outils pour développeurs.

**Renaissance de l’auto-hébergement :** Des projets comme Memos, Seanime, Listmonk et Arcane montrent une forte demande pour des alternatives respectueuses de la vie privée aux services cloud.

**Force infrastructure Go :** Cilium, NetBird et Bifrost démontrent la domination continue de Go dans les logiciels d’infrastructure et de réseau.

**Renaissance des interfaces terminales :** Charmbracelet’s BubbleTea et Crush montrent le retour des applications terminales élégantes.

## Articles connexes

- [Fiche mémo Go](https://www.glukhov.org/fr/post/2022/golang-cheatsheet/ "Fiche mémo Golang et commandes les plus utiles") — Syntaxe du langage et motifs courants
- [Fiche mémo Ollama](https://www.glukhov.org/fr/llm-hosting/ollama/ollama-cheatsheet/ "Fiche mémo Ollama") — Commandes pour exécuter des LLM locaux
- [Démarrage rapide vLLM](https://www.glukhov.org/fr/llm-hosting/vllm/vllm-quickstart/ "Guide complet de configuration vLLM") — Serving haute performance de LLM
- [Fiche mémo commandes GIT](https://www.glukhov.org/fr/developer-tools/git-and-forges/git-cheatsheet/ "Fiche mémo commandes GIT") — Opérations Git essentielles
- [Fiche mémo GitHub Actions](https://www.glukhov.org/fr/developer-tools/ci-cd/github-actions-cheatsheet/ "Fiche mémo GitHub Actions") — Automatisation des flux de travail CI/CD
- [Fiche mémo GitHub Copilot](https://www.glukhov.org/fr/ai-devtools/github-copilot-cheatsheet/ "Fiche mémo GitHub Copilot") — Commandes de l’assistant de codage IA

## Sources

- [Dépôts Go tendance sur GitHub (Mensuel)](https://github.com/trending/go?since=monthly "Page officielle des dépôts Go tendance sur GitHub")
- [Classement GitHub - Top 100 Projets Go](https://evanli.github.io/Github-Ranking/Top100/Go.html "Classement communautaire des meilleurs dépôts Go")
- [Documentation Beads](https://steveyegge.github.io/beads/ "Documentation officielle de Beads par Steve Yegge")
- [Blog Charmbracelet Crush](https://charm.land/blog/crush-comes-home/ "Annonciation de Crush par Charmbracelet")
- [Blog vLLM Semantic Router](https://blog.vllm.ai/2025/09/11/semantic-router.html "Annonciation de Semantic Router par vLLM")
- [Benchmarks de performance Bifrost](https://getmaxim.ai/blog/bifrost-a-drop-in-llm-proxy-40x-faster-than-litellm "Comparaison de performance Bifrost vs LiteLLM")
