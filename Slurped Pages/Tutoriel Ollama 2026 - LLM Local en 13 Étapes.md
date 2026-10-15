---
link: https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/
byline: Nadia Dubois
site: Tech Insider
date: 2026-05-29T18:31
excerpt: "Tutoriel Ollama v0.6.2 : installer, exécuter Llama 3.3 et Mistral en
  local, créer un RAG RGPD en 13 étapes testées sur Linux, macOS, Windows."
slurped: 2026-10-15T10:19
title: "Tutoriel Ollama 2026 : LLM Local en 13 Étapes"
---

29 May 2026

31 min de lecture

**Publié le 29 avril 2026 — Mise à jour : Ollama v0.32.1 (juillet 2026), patchée en v0.32.14 (août 2026)**

1. [Qu’est-ce qu’Ollama et pourquoi l’adopter en 2026](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-0)
2. [Prérequis matériels et logiciels (versions exactes)](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-1)
3. [Étape 1 : Installer Ollama sur Linux, macOS et Windows](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-2)
4. [Étape 2 : Télécharger votre premier modèle avec ollama pull](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-3)
5. [Étape 3 : Lancer votre premier prompt avec ollama run](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-4)
6. [Étape 4 : Comprendre les commandes CLI essentielles](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-5)
7. [Étape 5 : Interagir avec l’API HTTP native (/api/generate)](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-6)
8. [Étape 6 : Utiliser la compatibilité OpenAI (/v1/chat/completions)](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-7)
9. [Étape 7 : Créer un Modelfile personnalisé](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-8)
10. [Étape 8 : Choisir la bonne quantification (Q4_K_M, Q5_K_M, Q8_0)](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-9)
11. [Étape 9 : Générer des embeddings pour un RAG local](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-10)
12. [Étape 10 : Déployer Ollama dans un conteneur Docker](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-11)
13. [Étape 11 : Intégrer Ollama avec LangChain et LlamaIndex](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-12)
14. [Étape 12 : Optimiser les performances (Flash Attention, GPU, threading)](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-13)
15. [Étape 13 : Projet complet — Assistant RAG pour PME française](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-14)
16. [Comparatif Ollama vs alternatives en 2026](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-15)
17. [Sécurité, RGPD et déploiement en entreprise](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-16)
18. [Dépannage : 8 problèmes fréquents et solutions](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-17)
19. [Astuces avancées : LoRA, function calling, vision](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-18)
20. [Benchmarks 2026 : tokens/seconde par GPU](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-19)
21. [Foire aux questions](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-20)

1. [Ollama est-il vraiment gratuit ?](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-21)
2. [Peut-on faire tourner Llama 3.3 70B sur un MacBook ?](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-22)
3. [Quelle différence entre Ollama et LM Studio ?](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-23)
4. [Ollama envoie-t-il des données vers le cloud ?](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-24)
5. [Quel modèle choisir pour le français en 2026 ?](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-25)
6. [Comment fine-tuner un modèle avec Ollama ?](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-26)
7. [Ollama remplace-t-il l’API OpenAI ?](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-27)
8. [Comment migrer du Modelfile vers Hugging Face ?](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-28)

23. [Conclusion : Ollama en 2026, le standard du LLM local](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-29)

1. [Couverture associée](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-30)
2. [Sources externes](https://tech-insider.org/fr/ollama-tutoriel-llm-local-13-etapes-2026/#toc-31)

Ollama est devenu en 2026 l’outil de référence pour exécuter des grands modèles de langage (LLM) en local, sans envoyer une seule requête vers le cloud. Selon **TechCrunch** (juillet 2026), Ollama revendique désormais **8,9 millions de développeurs actifs par mois** dans le monde et une adoption chez **85 % des entreprises du Fortune 500** — une dynamique confirmée par le blog officiel d’Ollama qui, en août 2026, annonce à son tour **8,9 millions de développeurs** et un total de **88 millions de dollars levés**. Avec plus de **27 100 recherches mensuelles en France** selon DataForSEO (avril 2026), le runtime cumule désormais le support natif de **Llama 3.3 70B**, **Qwen 3.6 27B** — annoncé à **77,2 % sur le benchmark SWE-bench** en juillet 2026 comme nouvelle référence de la gamme Qwen —, **Gemma 3** (1B, 4B, 12B, 27B), le modèle multimodal open source **Meta Muse Glimmer** (30B, licence Apache 2.0, arrivé sur Ollama en août 2026 selon Ollama) et l’API compatible OpenAI sur le port `11434`. Le classement PromptQuorum de juillet 2026 place d’ailleurs Qwen3.6 27B aux côtés de **Llama 4 Scout 17B** et **Phi-4 14B** parmi les modèles Ollama les plus recommandés du moment, et dès septembre 2026 Ollama a étoffé sa gamme avec des modèles de décision de style **Jev**, allant de **0,8B à 9B** de paramètres, d’après les annonces officielles d’Ollama. Ce tutoriel pas-à-pas vous guide de l’installation jusqu’au déploiement d’un assistant RAG complet, avec 13 étapes testées sur Ubuntu 24.04 LTS, macOS Sonoma et Windows 11.

Contrairement aux API payantes (OpenAI à 5 $/Mtok input, Anthropic à 3 $/Mtok), exécuter Llama 3.3 70B en local coûte uniquement votre électricité — même si Ollama a lancé en juillet 2026 un **tier cloud hybride facturé au temps GPU**, qui revendiquait déjà **8,9 millions d’utilisateurs actifs mensuels** selon Open Source For You. Ce virage commercial suit une levée de **65 millions de dollars en Series B** bouclée en juillet 2026, portant le total levé par Ollama à **88 millions de dollars** d’après TechCrunch — un pari renforcé par l’arrivée sur Ollama Cloud, dès juin 2026, du modèle **NVIDIA Nemotron 3 Ultra** (550B de paramètres dont 55B actifs), selon Ollama. Depuis la v0.32.0, sortie le **11 juillet 2026** selon la mise à jour de juillet 2026 de PromptQuorum, lancer la commande `ollama` sans argument ouvre directement un **agent interactif** plutôt qu’un simple message d’aide, et la commande `ollama launch` propose désormais une **fenêtre de session de codage de 5 heures** à usage étendu, selon le blog Ollama (mai 2026). Le rythme de publication reste soutenu : après la v0.32.1 (**16 juillet 2026**, selon LocalAIMaster) et la v0.33.2 (**27 août 2026**), c’est désormais la **v0.35.1**, sortie en **septembre 2026** d’après GitHub, qui fait référence, avec la prise en charge des modèles de décision **Clef**. Cette cadence accompagne une croissance rapide de la base d’utilisateurs actifs, passée d’environ **1,5 million en mai 2024** à environ **5,0 millions en mai 2026** selon le rapport écosystème de Presenc AI. Pour un développeur français soumis au RGPD ou un consultant qui manipule des données clients, c’est l’option par défaut en 2026.


## Qu’est-ce qu’Ollama et pourquoi l’adopter en 2026

Ollama est un runtime open source écrit en Go qui simplifie radicalement l’exécution locale de modèles de langage au format GGUF. Là où llama.cpp exige de compiler les sources, télécharger manuellement les poids quantifiés et configurer chaque paramètre, Ollama propose une expérience proche de Docker : `ollama pull llama3.3`, `ollama run llama3.3`, et le modèle répond. Cette philosophie « batteries incluses » explique son adoption massive auprès des développeurs européens qui doivent composer avec les contraintes du RGPD et de l’AI Act.

En avril 2026, Ollama supporte officiellement trois plateformes : **Linux** (Ubuntu, Debian, Fedora, Arch), **macOS** (Apple Silicon avec Metal 3) et **Windows 10/11**. Le moteur d’inférence s’appuie sur llama.cpp pour le calcul, mais ajoute un serveur HTTP exposant deux surfaces d’API : l’API native (`/api/generate`, `/api/chat`, `/api/embed`, `/api/tags`) et une couche de compatibilité OpenAI (`/v1/chat/completions`) qui permet de réutiliser les SDK existants sans modification.

Les versions récentes ont apporté plusieurs améliorations clés, portées par un rythme de publication soutenu qui a franchi la barre des **216 releases cumulées sur GitHub** dès mai 2026. Ce rythme se reflète aussi dans la bibliothèque de modèles elle-même, qui comptait déjà plus de **1 200 modèles référencés** dans la registry Ollama en mai 2026, selon les données d’usage compilées par Presenc AI. La v0.6.2 introduit Flash Attention v2.7 pour AMD ROCm 6.3+, ce qui débloque enfin les Radeon RX 7900 XTX comme alternative crédible aux RTX 4090. Côté vitesse pure, la **v0.23.1** (5 mai 2026) avait déjà introduit le décodage spéculatif pour **Gemma 4 31B**, doublant son débit d’inférence selon Fazm.ai. Sur Apple Silicon, la préversion MLX livrée avec la v0.19 (mars 2026) exploite désormais nativement la mémoire unifiée d’Apple et vise environ **2× le débit d’inférence** — Ollama l’a d’ailleurs testée dès mars 2026 avec le modèle MoE **Qwen3.5-35B-A3B** —, ce qui permet de faire tenir Llama 3.3 70B sur un MacBook Pro M3 Max 128 Go. Côté contexte, la variable `OLLAMA_CONTEXT_LENGTH` permet de définir une fenêtre par défaut (exemple : `OLLAMA_CONTEXT_LENGTH=8192 ollama serve`) sans toucher au Modelfile.

## Prérequis matériels et logiciels (versions exactes)

Avant d’installer Ollama, vérifiez que votre machine satisfait aux exigences. Les besoins en RAM varient drastiquement selon la taille du modèle et le niveau de quantification choisi. Un modèle 7B en Q4_K_M occupe environ 4,5 Go de RAM, contre 40 Go pour un 70B en Q4_K_M et plus de 140 Go pour le même modèle non quantifié en FP16. Pour 90 % des usages français (rédaction, RAG documentaire, génération de code), un GPU de 8 à 12 Go de VRAM suffit pour faire tourner les modèles 7B-13B avec des performances correctes.

|Composant|Minimum|Recommandé (13B)|Idéal (70B)|
|---|---|---|---|
|OS|Ubuntu 22.04 / macOS 12 / Win 10|Ubuntu 24.04 / macOS Sonoma / Win 11|Ubuntu 24.04 LTS|
|RAM système|8 Go|16 Go|64 Go|
|VRAM GPU|4 Go (3B-7B)|12 Go (RTX 4070)|48 Go (RTX 6000 Ada)|
|CPU (fallback)|4 cœurs AVX2|8 cœurs AVX-512|16 cœurs Threadripper|
|Stockage SSD|20 Go|100 Go NVMe|500 Go NVMe Gen4|
|Connexion (pull initial)|10 Mb/s|100 Mb/s|1 Gb/s|

Côté logiciel, vous aurez besoin de Python 3.11 ou supérieur pour les scripts d’intégration (Python 3.12 recommandé en 2026), de `curl` pour les installeurs, et optionnellement de Docker 25.0+ si vous préférez containeriser le serveur. Pour les utilisateurs NVIDIA, les pilotes CUDA 12.4 ou supérieurs sont obligatoires depuis la v0.5.0. Sur AMD ROCm, la version 6.3 minimum est requise pour activer Flash Attention v2.7. Vérifiez votre version GPU avant de continuer.

```
# Vérifier la version du pilote NVIDIA
nvidia-smi | grep "Driver Version"
# Attendu : Driver Version: 550.xx ou supérieur

# Vérifier CUDA
nvcc --version
# Attendu : Cuda compilation tools, release 12.4+

# Vérifier Python
python3 --version
# Attendu : Python 3.11.x ou supérieur

# Vérifier l'espace disque libre
df -h ~ | tail -1
```

## Étape 1 : Installer Ollama sur Linux, macOS et Windows

L’installation d’Ollama varie selon votre système. Sur Linux, le script officiel détecte automatiquement votre distribution et installe le binaire dans `/usr/local/bin`, ainsi qu’un service systemd qui démarre Ollama au boot. Sur macOS, vous avez le choix entre l’application native (.dmg) qui intègre une icône dans la barre de menus, ou la formule Homebrew. Sur Windows 11, l’installeur PowerShell est devenu la méthode officielle depuis la v0.5.0.

```
# --- LINUX (Ubuntu, Debian, Fedora, Arch) ---
curl -fsSL https://ollama.com/install.sh | sh

# Vérifier l'installation
ollama --version
# Attendu : ollama version is 0.6.2

# Activer le service systemd
sudo systemctl enable ollama
sudo systemctl start ollama

# --- macOS ---
# Option A : Homebrew
brew install ollama

# Option B : Application native
# Télécharger https://ollama.com/download/Ollama-darwin.zip

# --- WINDOWS 11 (PowerShell admin) ---
irm https://ollama.com/install.ps1 | iex
```

Après installation, le serveur Ollama écoute par défaut sur `http://localhost:11434`. Pour vérifier qu’il fonctionne, lancez `curl http://localhost:11434` : la réponse doit être « Ollama is running ». Si vous voyez une erreur de connexion, démarrez manuellement le serveur avec `ollama serve` dans un terminal. Sur Linux, vérifiez le statut systemd avec `systemctl status ollama` : le service doit être actif et activé.

**Piège fréquent #1** : sur Windows, l’antivirus Defender peut bloquer le téléchargement du binaire CUDA pendant l’installation. Désactivez temporairement la protection en temps réel ou ajoutez `%LOCALAPPDATA%\Programs\Ollama` aux exclusions. Sur Linux, si vous utilisez ZFS ou Btrfs pour `/usr`, le service peut échouer à démarrer avec une erreur de permission ; déplacez le binaire vers `/opt/ollama` et ajustez le fichier service.

## Étape 2 : Télécharger votre premier modèle avec ollama pull

Une fois le serveur opérationnel, l’étape suivante consiste à télécharger un modèle depuis la bibliothèque officielle. Ollama utilise un système de tags inspiré de Docker : `llama3.3:70b` spécifie la version 70 milliards de paramètres, tandis que `llama3.3:70b-instruct-q4_K_M` précise la quantification. Les nouveautés se propagent vite : le tag `qwen3.8:27b`, sorti mi-août 2026, a déjà cumulé environ **501 800 pulls** lors de sa première semaine de disponibilité, selon les chiffres publiés par MorphLLM — un signe de l’appétit des développeurs pour la gamme Qwen, dont la version **Qwen3.6 27B** atteint **84 % sur le benchmark MMLU** tout en tenant dans environ **17 Go de RAM** en quantification Q4_K_M, d’après le classement de juillet 2026 de PromptQuorum. D’autres arrivées confirment cette dynamique : le modèle MoE **Poolside Laguna XS 2.1** (33B, avec un routage actif de 3B) a atteint **70,9 % sur SWE-bench** dès son intégration à Ollama le **2 juillet 2026**, toujours selon PromptQuorum, suivi en août 2026 par **NVIDIA Nemotron 3.5 Lightning**, un autre MoE de **30B de paramètres avec 3B actifs**, selon Ollama. La commande `ollama pull` télécharge le modèle, vérifie son hash SHA-256, puis le décompresse dans `~/.ollama/models` (Linux/macOS) ou `%USERPROFILE%\.ollama\models` (Windows).

```
# Modèles populaires en 2026 (vérifiés sur ollama.com/library)

# Pour débuter — 4,7 Go, 8 Go RAM suffisent
ollama pull llama3.1:8b

# Modèle français-friendly — 4,1 Go
ollama pull mistral:7b-instruct-v0.3

# Code & raisonnement — 4,1 Go
ollama pull qwen3:7b

# Petit, rapide, multilingue — 815 Mo
ollama pull gemma3:1b

# Top tier — 40 Go, GPU 48 Go requis
ollama pull llama3.3:70b

# Vérifier les modèles installés
ollama list
# NAME              ID            SIZE      MODIFIED
# llama3.1:8b       42182419e950  4.7 GB    2 minutes ago
# mistral:7b        2ae6f6dd7a3d  4.1 GB    5 minutes ago
```

Le téléchargement peut être long sur une connexion ADSL : prévoyez 8 à 15 minutes pour un modèle 7B sur une fibre 100 Mb/s, et jusqu’à 2 heures pour Llama 3.3 70B (40 Go). Ollama gère le téléchargement par couches comme Docker, ce qui signifie que si vous tirez plusieurs variantes du même modèle de base, les couches communes ne sont téléchargées qu’une fois. Cela économise typiquement 30 à 50 % d’espace disque pour qui expérimente avec plusieurs quantifications.

**Piège fréquent #2** : si votre dossier `~/.ollama` est sur un disque chiffré (LUKS, FileVault), les performances de chargement initial peuvent chuter de 40 à 60 %. Configurez `OLLAMA_MODELS` pour pointer vers un SSD NVMe non chiffré dédié aux modèles : `export OLLAMA_MODELS=/mnt/nvme/ollama`. Cette variable doit être définie avant le démarrage du service, pas dans une session interactive.

## Étape 3 : Lancer votre premier prompt avec ollama run

La commande `ollama run` charge le modèle en mémoire (RAM ou VRAM selon votre GPU) et ouvre une session de chat interactive. C’est la manière la plus rapide de tester un modèle. Pour un usage scripté, on utilisera plutôt l’API HTTP que nous verrons à l’étape 5. La session interactive supporte plusieurs commandes spéciales préfixées par `/` : `/bye` pour quitter, `/clear` pour réinitialiser le contexte, `/show` pour afficher les paramètres, et `/set` pour ajuster température, top_p, system prompt en cours de session.

```
# Lancer une session interactive
ollama run llama3.1:8b

>>> Bonjour, présente-toi en français en 2 phrases.
Je suis Llama 3.1, un modèle de langage open source développé par Meta.
Je peux répondre à des questions, rédiger du texte et discuter en plusieurs langues, dont le français.

>>> /set parameter temperature 0.3
Set parameter 'temperature' to '0.3'

>>> /show parameters
Model defined parameters:
temperature                    0.3
top_k                          40
top_p                          0.9
num_ctx                        4096

>>> /bye

# Mode "one-shot" : prompt direct depuis la ligne de commande
ollama run llama3.1:8b "Écris un haïku sur la pluie à Paris"
# Sortie : Pluie sur les toits / Le métro file en silence / Avril déguisé
```

Sur un RTX 4090 (24 Go VRAM), Llama 3.1 8B en Q4_K_M génère typiquement 80 à 110 tokens/seconde après le warm-up initial. Sur un MacBook Pro M3 Max, comptez 45 à 65 tokens/seconde avec le backend Metal 3 classique, mais un benchmark publié en mars 2026 avec la préversion MLX rapporte un débit passant d’**environ 58 à environ 112 tokens/seconde** sur le même type de modèle, selon Local AI Master. Sur un Ryzen 9 7950X sans GPU, le même modèle plafonne à 8-12 tokens/seconde, ce qui reste utilisable pour des prompts courts mais devient pénible pour de la génération longue. Pour mesurer votre propre débit, utilisez la commande `ollama run llama3.1:8b --verbose` qui affiche le temps de génération en fin de chaque réponse.

## Étape 4 : Comprendre les commandes CLI essentielles

Ollama propose une dizaine de commandes principales que tout utilisateur sérieux doit connaître. Au-delà de `pull` et `run`, la gestion quotidienne s’appuie sur `list`, `ps`, `rm`, `show` et `cp`. La commande `ollama ps` est particulièrement utile : elle affiche quels modèles sont actuellement chargés en mémoire et combien de VRAM ils consomment. Par défaut, Ollama décharge un modèle après 5 minutes d’inactivité ; vous pouvez ajuster ce timeout avec la variable `OLLAMA_KEEP_ALIVE`.

|Commande|Description|Exemple|
|---|---|---|
|`ollama serve`|Démarre le serveur API sur le port 11434|`OLLAMA_HOST=0.0.0.0:11434 ollama serve`|
|`ollama pull`|Télécharge un modèle depuis la registry|`ollama pull qwen3:14b`|
|`ollama run`|Lance une session interactive ou un prompt|`ollama run mistral "Résume ce texte..."`|
|`ollama list` ou `ls`|Liste les modèles téléchargés|`ollama list`|
|`ollama ps`|Affiche les modèles chargés en RAM/VRAM|`ollama ps`|
|`ollama show`|Affiche les détails (paramètres, Modelfile)|`ollama show llama3.1:8b`|
|`ollama rm`|Supprime un modèle local|`ollama rm mistral:7b`|
|`ollama cp`|Crée une copie alias d’un modèle|`ollama cp llama3.1 mon-llama`|
|`ollama create`|Crée un modèle à partir d’un Modelfile|`ollama create assistant -f Modelfile`|
|`ollama push`|Publie un modèle sur la registry|`ollama push user/mon-modele`|

La variable `OLLAMA_HOST` mérite une attention particulière. Par défaut, le serveur n’écoute que sur `127.0.0.1`, ce qui empêche l’accès depuis d’autres machines du réseau. Pour exposer Ollama à votre LAN (par exemple depuis un laptop vers un serveur d’inférence sous Ubuntu), définissez `OLLAMA_HOST=0.0.0.0:11434`. Sur systemd, éditez `/etc/systemd/system/ollama.service.d/override.conf` avec `sudo systemctl edit ollama` et ajoutez `Environment="OLLAMA_HOST=0.0.0.0:11434"`. Attention : exposer Ollama publiquement sans authentification est une faille de sécurité majeure.

## Étape 5 : Interagir avec l’API HTTP native (/api/generate)

L’API HTTP est le cœur d’Ollama pour toute intégration sérieuse. Elle expose quatre endpoints principaux : `/api/generate` pour la complétion simple, `/api/chat` pour les conversations multi-tours avec historique, `/api/embed` pour générer des embeddings vectoriels, et `/api/tags` pour lister les modèles disponibles. Chaque endpoint accepte des requêtes POST JSON et peut renvoyer soit une réponse complète, soit un flux SSE (Server-Sent Events) si vous activez `"stream": true`.

```
# Complétion simple (non-streaming)
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.1:8b",
  "prompt": "Explique le RGPD en 3 points",
  "stream": false,
  "options": {
    "temperature": 0.4,
    "num_ctx": 4096,
    "top_p": 0.9
  }
}'

# Réponse JSON typique :
# {
#   "model": "llama3.1:8b",
#   "created_at": "2026-04-29T10:23:45.123Z",
#   "response": "Le RGPD impose...",
#   "done": true,
#   "total_duration": 4823000000,
#   "load_duration": 12000000,
#   "prompt_eval_count": 12,
#   "eval_count": 142,
#   "eval_duration": 4500000000
# }

# Conversation multi-tours avec historique
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.1:8b",
  "messages": [
    {"role": "system", "content": "Tu es un expert juridique français."},
    {"role": "user", "content": "Article 5 RGPD ?"}
  ],
  "stream": false
}'

# Lister les modèles
curl http://localhost:11434/api/tags | jq '.models[].name'
```

Les champs `total_duration`, `prompt_eval_count` et `eval_count` de la réponse sont précieux pour mesurer les performances réelles de votre déploiement. En divisant `eval_count` par `eval_duration` (en secondes), vous obtenez le débit en tokens/seconde. Le champ `load_duration` indique le temps de chargement du modèle en VRAM : il est non nul uniquement lors de la première requête après un déchargement. Pour un service production qui sert des milliers de requêtes, gardez le modèle chaud avec `"keep_alive": "24h"`.

**Piège fréquent #3** : par défaut, `num_ctx` est limité à 2048 tokens même si le modèle supporte 128k. Cette limite par défaut sert à économiser la VRAM, mais provoque une troncature silencieuse des prompts longs. Pour profiter de la pleine fenêtre de Llama 3.1 (128k), ajoutez explicitement `"options": {"num_ctx": 131072}` dans chaque requête, ou définissez `OLLAMA_CONTEXT_LENGTH=131072` avant `ollama serve`. Vérifiez que vous avez la VRAM nécessaire : un contexte de 128k peut consommer 30 à 50 Go supplémentaires.

## Étape 6 : Utiliser la compatibilité OpenAI (/v1/chat/completions)

L’une des features les plus utiles d’Ollama est sa couche de compatibilité OpenAI exposée sur `/v1`. Cela permet de remplacer un appel `OpenAI()` dans votre code Python ou JavaScript par un appel à Ollama local sans changer une seule ligne de logique métier. Le SDK officiel `openai` reconnaît le paramètre `base_url` qui redirige toutes les requêtes vers `http://localhost:11434/v1`. C’est l’intégration la plus rapide pour migrer un projet existant.

```
# Installation du SDK OpenAI (Python 3.11+)
pip install openai==1.54.3

# script openai_compat.py
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # Clé factice, requise mais ignorée
)

response = client.chat.completions.create(
    model="llama3.1:8b",
    messages=[
        {"role": "system", "content": "Réponds en français concis."},
        {"role": "user", "content": "Quelle est la capitale de la Belgique ?"}
    ],
    temperature=0.2,
    max_tokens=150
)

print(response.choices[0].message.content)
# Sortie : Bruxelles est la capitale de la Belgique.

# Mode streaming
stream = client.chat.completions.create(
    model="llama3.1:8b",
    messages=[{"role": "user", "content": "Compte de 1 à 5"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

La compatibilité OpenAI couvre `/v1/chat/completions`, `/v1/completions`, `/v1/embeddings` et `/v1/models`. Quelques différences subsistent : Ollama ignore les paramètres `logit_bias`, `logprobs` et `n > 1` (génération multiple). Les `tool_calls` sont supportés depuis la v0.4 pour les modèles compatibles (Llama 3.1, Qwen 2.5+, Mistral Nemo), et la v0.24.0 (mai 2026) a ajouté la prise en charge de l’app Codex, élargissant l’intégration côté outils de codage assistés par IA. Pour le SDK JavaScript, le package officiel `ollama` (`npm install ollama`) offre une API typée TypeScript plus idiomatique que le détournement du SDK OpenAI.

## Étape 7 : Créer un Modelfile personnalisé

Le Modelfile est l’équivalent Ollama du Dockerfile : un fichier texte qui décrit comment construire un modèle personnalisé à partir d’un modèle de base. Les mots-clés principaux sont `FROM` (modèle parent), `PARAMETER` (température, top_p, num_ctx, etc.), `SYSTEM` (prompt système permanent), `TEMPLATE` (format de prompt Jinja), `MESSAGE` (exemples few-shot) et `ADAPTER` (chargement d’un LoRA). Cette mécanique transforme Ollama en plateforme de packaging réutilisable pour distribuer des assistants spécialisés.

```
# Fichier : Modelfile
FROM llama3.1:8b

# Paramètres d'inférence
PARAMETER temperature 0.3
PARAMETER top_p 0.85
PARAMETER num_ctx 8192
PARAMETER repeat_penalty 1.1
PARAMETER stop "<|eot_id|>"

# Prompt système permanent
SYSTEM """Tu es Juris-FR, un assistant juridique français spécialisé
en droit du numérique. Réponds toujours en français professionnel.
Cite l'article exact du Code applicable. Si tu n'es pas sûr,
indique-le clairement plutôt que d'inventer."""

# Exemples few-shot
MESSAGE user "Quel article RGPD pour le droit à l'oubli ?"
MESSAGE assistant "Article 17 du RGPD (Règlement UE 2016/679)."

# Construire le modèle
ollama create juris-fr -f Modelfile
# transferring model data
# creating layer
# writing manifest
# success

# Utiliser le modèle personnalisé
ollama run juris-fr "Que dit l'article 5 du RGPD ?"
```

L’avantage d’un Modelfile bien construit est de garantir un comportement reproductible entre environnements de développement et production. Vous pouvez versionner votre Modelfile dans Git, le distribuer à votre équipe, et chacun obtiendra le même modèle après `ollama create`. Pour les déploiements à plusieurs nœuds, vous pouvez aussi pousser votre modèle vers ollama.com (compte gratuit requis) avec `ollama push user/juris-fr`, puis le récupérer ailleurs avec `ollama pull user/juris-fr`. Cette mécanique facilite la collaboration interne sans exposer publiquement vos prompts système.

## Étape 8 : Choisir la bonne quantification (Q4_K_M, Q5_K_M, Q8_0)

La quantification est le processus qui réduit la précision des poids du modèle (par exemple de FP16 16-bits à 4-bits) pour économiser drastiquement la mémoire au prix d’une légère perte de qualité. Ollama distribue par défaut les modèles en Q4_K_M, considéré comme le meilleur compromis qualité/taille pour la majorité des usages. Comprendre les différentes quantifications GGUF est essentiel pour optimiser votre déploiement selon votre matériel.

|Quantification|Bits effectifs|Taille (8B)|Perplexité vs FP16|Recommandation|
|---|---|---|---|---|
|Q2_K|~2,6|3,0 Go|+15 % (dégradé)|Éviter sauf urgence VRAM|
|Q3_K_M|~3,4|3,8 Go|+5 %|Acceptable, 4 Go VRAM|
|Q4_K_M|~4,5|4,7 Go|+1,5 %|Sweet spot par défaut|
|Q5_K_M|~5,5|5,4 Go|+0,7 %|Si VRAM disponible|
|Q6_K|~6,5|6,1 Go|+0,2 %|Quasi-FP16|
|Q8_0|~8,5|8,0 Go|+0,05 %|Production critique|
|F16|16|14,9 Go|Référence|Recherche, fine-tuning|

En pratique pour un développeur français, la règle est simple : si votre modèle Q4_K_M tient en VRAM avec votre fenêtre de contexte habituelle, restez sur Q4_K_M. Si vous avez 30 à 50 % de VRAM libre, montez à Q5_K_M ou Q6_K pour un gain de qualité mesurable sur les tâches de raisonnement. Réservez Q8_0 aux cas où la qualité est critique (génération de contrats juridiques, code médical) et où vous pouvez vous permettre 70 % de RAM en plus. Pour télécharger une quantification spécifique :

```
# Tags spécifiques par quantification
ollama pull llama3.1:8b-instruct-q4_K_M    # par défaut
ollama pull llama3.1:8b-instruct-q5_K_M    # +15 % qualité
ollama pull llama3.1:8b-instruct-q8_0      # quasi-FP16
ollama pull llama3.1:8b-instruct-fp16      # référence

# Comparer les tailles
ollama list | grep llama3.1
# llama3.1:8b-instruct-q4_K_M    4.7 GB
# llama3.1:8b-instruct-q5_K_M    5.4 GB
# llama3.1:8b-instruct-q8_0      8.0 GB
# llama3.1:8b-instruct-fp16     14.9 GB
```

## Étape 9 : Générer des embeddings pour un RAG local

L’endpoint `/api/embed` introduit en v0.6.2 permet de générer des embeddings vectoriels en batch, brique fondamentale pour construire un système RAG (Retrieval-Augmented Generation) 100 % local. Les modèles d’embeddings recommandés en 2026 sont `nomic-embed-text` (768 dimensions, 137 Mo) pour les usages généraux et `mxbai-embed-large` (1024 dimensions, 670 Mo) pour la production. Pour le français, `jina-embeddings-v2-base-fr` donne d’excellents résultats sur les documents juridiques et techniques.

```
# Télécharger un modèle d'embeddings
ollama pull nomic-embed-text

# Générer un embedding (batch)
curl http://localhost:11434/api/embed -d '{
  "model": "nomic-embed-text",
  "input": [
    "Le RGPD est entré en vigueur le 25 mai 2018.",
    "La CNIL est l autorité de contrôle française.",
    "Le DPO est le délégué à la protection des données."
  ]
}'

# Réponse :
# {
#   "model": "nomic-embed-text",
#   "embeddings": [[0.123, -0.456, ...], [...], [...]],
#   "total_duration": 234000000,
#   "load_duration": 1200000,
#   "prompt_eval_count": 36
# }

# Avec Python + ChromaDB
pip install chromadb==0.5.13 ollama==0.4.2

# rag_setup.py
import ollama
import chromadb

client = chromadb.PersistentClient(path="./rag_db")
collection = client.get_or_create_collection("docs_juridiques")

docs = [
    "Article 5 RGPD : licéité, loyauté, transparence du traitement.",
    "Article 17 RGPD : droit à l effacement (droit à l oubli).",
    "Article 25 RGPD : protection des données dès la conception."
]

for i, doc in enumerate(docs):
    emb = ollama.embed(model="nomic-embed-text", input=doc)
    collection.add(
        ids=[f"doc_{i}"],
        embeddings=[emb["embeddings"][0]],
        documents=[doc]
    )

# Requête
query = "Comment supprimer mes données personnelles ?"
q_emb = ollama.embed(model="nomic-embed-text", input=query)
results = collection.query(query_embeddings=[q_emb["embeddings"][0]], n_results=2)
print(results["documents"])
```

Une fois les embeddings stockés dans ChromaDB (ou Qdrant, ou Milvus), le pattern RAG complet consiste à : (1) embeddifier la question utilisateur, (2) récupérer les _k_ documents les plus proches en similarité cosinus, (3) injecter ces documents dans le prompt système d’un modèle de génération, (4) répondre à l’utilisateur. Cette architecture permet de répondre sur des documents privés sans les envoyer dans le cloud, conformité RGPD garantie par construction.

## Étape 10 : Déployer Ollama dans un conteneur Docker

Pour un déploiement production reproductible, containeriser Ollama est la voie royale. L’image officielle `ollama/ollama` est maintenue activement et supporte CPU, GPU NVIDIA (via nvidia-container-toolkit) et GPU AMD (via ROCm). Le volume persistant `/root/.ollama` stocke les modèles téléchargés et survit aux recréations du conteneur. Cette pratique évite de re-télécharger 40 Go à chaque redéploiement.

```
# docker-compose.yml — déploiement GPU NVIDIA
version: "3.9"
services:
  ollama:
    image: ollama/ollama:0.6.2
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    environment:
      - OLLAMA_HOST=0.0.0.0:11434
      - OLLAMA_KEEP_ALIVE=24h
      - OLLAMA_NUM_PARALLEL=4
      - OLLAMA_MAX_LOADED_MODELS=2
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:11434/api/tags"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  ollama_data:

# Démarrer
docker compose up -d

# Vérifier que le GPU est détecté
docker compose exec ollama nvidia-smi

# Tirer un modèle dans le conteneur
docker compose exec ollama ollama pull llama3.1:8b

# Tester l API depuis l hôte
curl http://localhost:11434/api/tags
```

Les variables `OLLAMA_NUM_PARALLEL` et `OLLAMA_MAX_LOADED_MODELS` méritent d’être ajustées selon votre charge. `OLLAMA_NUM_PARALLEL=4` autorise 4 requêtes simultanées par modèle (utile pour servir une équipe), tandis que `OLLAMA_MAX_LOADED_MODELS=2` limite à 2 le nombre de modèles distincts en VRAM. Si vous servez à la fois un modèle de génération (Llama 3.1 8B) et un d’embeddings (nomic-embed-text), cette valeur doit être ≥ 2 sinon le système swappe en permanence entre les deux, écroulant les performances.

**Piège fréquent #4** : sur Docker Desktop Windows avec WSL2, le passthrough GPU exige des étapes supplémentaires : installer le pilote NVIDIA pour WSL côté Windows, vérifier `wsl --update`, et activer « Use NVIDIA GPU acceleration » dans les paramètres Docker Desktop. Sans ces étapes, Ollama tombe silencieusement sur le CPU et vous voyez des performances 10 à 20× plus lentes que prévu, sans message d’erreur explicite.

## Étape 11 : Intégrer Ollama avec LangChain et LlamaIndex

Les frameworks d’orchestration LLM comme LangChain et LlamaIndex offrent des intégrations Ollama de première classe. Ces librairies abstraient la mécanique des prompts, gèrent la mémoire conversationnelle, et permettent de chaîner plusieurs appels LLM (« agents », « chains »). Pour un développeur français qui construit un chatbot interne ou un assistant de support, c’est généralement le bon niveau d’abstraction : ni le bas niveau de l’API HTTP, ni la rigidité d’une plateforme SaaS.

```
# Installation
pip install langchain==0.3.7 langchain-ollama==0.2.0 langchain-chroma==0.1.4

# rag_langchain.py — RAG complet en 30 lignes
from langchain_ollama import ChatOllama, OllamaEmbeddings
from langchain_chroma import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

# 1. Charger et découper le document
with open("rgpd.txt", "r", encoding="utf-8") as f:
    text = f.read()
splitter = RecursiveCharacterTextSplitter(chunk_size=800, chunk_overlap=100)
chunks = splitter.split_text(text)

# 2. Embeddings et vector store
embeddings = OllamaEmbeddings(model="nomic-embed-text")
vectorstore = Chroma.from_texts(chunks, embeddings, persist_directory="./db")
retriever = vectorstore.as_retriever(search_kwargs={"k": 4})

# 3. LLM et prompt
llm = ChatOllama(model="llama3.1:8b", temperature=0.2, num_ctx=8192)
prompt = ChatPromptTemplate.from_template(
    """Tu es un assistant juridique français.
Réponds uniquement avec le contexte fourni.

Contexte :
{context}

Question : {question}

Réponse :"""
)

# 4. Chaîne RAG
chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# 5. Question
print(chain.invoke("Que dit l article 17 sur le droit à l oubli ?"))
```

LlamaIndex propose une approche similaire mais orientée « moteur de recherche augmenté ». Pour un projet d’analyse documentaire massive (par exemple indexer 10 000 PDF de jurisprudence), LlamaIndex offre des structures d’index hiérarchiques plus efficaces que LangChain. Pour un chatbot conversationnel ou un agent multi-étapes, LangChain garde l’avantage. Les deux frameworks supportent Ollama nativement via leurs packages dédiés `llama-index-llms-ollama` et `langchain-ollama`.

## Étape 12 : Optimiser les performances (Flash Attention, GPU, threading)

Une fois Ollama opérationnel, plusieurs leviers permettent d’extraire 20 à 40 % de performance supplémentaire. Le premier est **Flash Attention**, activé par défaut depuis v0.6.2 sur GPU compatibles, mais qui peut être forcé via `OLLAMA_FLASH_ATTENTION=1`. Le second est le KV-cache quantifié (`OLLAMA_KV_CACHE_TYPE=q8_0`), qui réduit de moitié la VRAM consommée par le contexte, libérant de la place pour des fenêtres plus larges.

```
# Variables d'optimisation à définir avant ollama serve
export OLLAMA_FLASH_ATTENTION=1
export OLLAMA_KV_CACHE_TYPE=q8_0
export OLLAMA_NUM_PARALLEL=4
export OLLAMA_KEEP_ALIVE=24h
export OLLAMA_CONTEXT_LENGTH=8192
export OLLAMA_MAX_LOADED_MODELS=2

# Sur Linux avec systemd
sudo systemctl edit ollama
# Ajouter dans la section [Service] :
# Environment="OLLAMA_FLASH_ATTENTION=1"
# Environment="OLLAMA_KV_CACHE_TYPE=q8_0"

sudo systemctl daemon-reload
sudo systemctl restart ollama

# Vérifier l utilisation GPU pendant inférence
watch -n 1 nvidia-smi

# Mesurer le débit réel
time curl http://localhost:11434/api/generate -d '{
  "model": "llama3.1:8b",
  "prompt": "Compte de 1 à 100",
  "stream": false,
  "options": {"num_predict": 500}
}' | jq '.eval_count / (.eval_duration / 1000000000)'
```

Pour un usage CPU-only (machines sans GPU), la commande `num_thread` dans les options de requête, ou la variable `OLLAMA_NUM_THREADS`, contrôle le parallélisme. La règle empirique : utilisez le nombre de cœurs physiques (pas logiques avec hyperthreading) moins 1, pour laisser un cœur au système. Sur un Ryzen 9 7950X (16 cœurs physiques), `OLLAMA_NUM_THREADS=15` est optimal. Au-delà, le surcoût de synchronisation annule les gains.

**Piège fréquent #5** : sur les machines hybrides (CPU + GPU faible), Ollama peut décider de répartir le modèle entre VRAM et RAM (mode « partial offload »), entraînant des performances catastrophiques. Forcez tout le modèle sur GPU avec `"options": {"num_gpu": -1}` (toutes les couches), ou choisissez explicitement le nombre de couches GPU avec `"num_gpu": 32` pour les 32 premières couches. Si vous voyez « offloading X/33 layers » dans les logs et que X < nombre total, soit votre VRAM est insuffisante, soit votre quantification est trop lourde.

## Étape 13 : Projet complet — Assistant RAG pour PME française

Pour boucler ce tutoriel, voici un projet complet qui assemble toutes les briques précédentes : un assistant interne pour PME française qui répond aux questions des employés sur les procédures RH internes, en s’appuyant uniquement sur les documents PDF stockés localement. Le projet utilise FastAPI pour l’interface HTTP, Ollama pour le LLM et les embeddings, ChromaDB pour le vector store, et PyPDF pour l’extraction de texte. L’ensemble tourne sur une machine avec 16 Go RAM et un GPU RTX 4060 (8 Go VRAM).

```
# Structure du projet
# assistant-rh/
# ├── docs/                  # PDF des procédures RH
# ├── main.py                # API FastAPI
# ├── ingest.py              # Indexation initiale
# ├── requirements.txt
# └── docker-compose.yml

# requirements.txt
fastapi==0.115.5
uvicorn==0.32.0
pypdf==5.1.0
chromadb==0.5.13
ollama==0.4.2
python-multipart==0.0.17

# ingest.py — Indexer tous les PDF du dossier docs/
import os
from pypdf import PdfReader
import ollama
import chromadb

client = chromadb.PersistentClient(path="./db")
collection = client.get_or_create_collection("rh")

def chunk_text(text, size=600, overlap=80):
    chunks = []
    start = 0
    while start < len(text):
        chunks.append(text[start:start+size])
        start += size - overlap
    return chunks

for pdf_file in os.listdir("docs"):
    if not pdf_file.endswith(".pdf"):
        continue
    reader = PdfReader(f"docs/{pdf_file}")
    full_text = "\n".join(p.extract_text() for p in reader.pages)
    chunks = chunk_text(full_text)
    for i, chunk in enumerate(chunks):
        emb = ollama.embed(model="nomic-embed-text", input=chunk)
        collection.add(
            ids=[f"{pdf_file}_{i}"],
            embeddings=[emb["embeddings"][0]],
            documents=[chunk],
            metadatas=[{"source": pdf_file, "chunk": i}]
        )
    print(f"Indexé : {pdf_file} ({len(chunks)} chunks)")

# main.py — API HTTP
from fastapi import FastAPI
from pydantic import BaseModel
import ollama
import chromadb

app = FastAPI(title="Assistant RH")
client = chromadb.PersistentClient(path="./db")
collection = client.get_collection("rh")

class Question(BaseModel):
    text: str
    k: int = 4

@app.post("/ask")
def ask(q: Question):
    q_emb = ollama.embed(model="nomic-embed-text", input=q.text)
    results = collection.query(
        query_embeddings=[q_emb["embeddings"][0]],
        n_results=q.k
    )
    context = "\n\n".join(results["documents"][0])
    sources = list({m["source"] for m in results["metadatas"][0]})

    response = ollama.chat(
        model="llama3.1:8b",
        messages=[
            {"role": "system", "content":
             "Tu es l assistant RH. Réponds uniquement avec le contexte fourni. "
             "Si la réponse n est pas dans le contexte, dis-le."},
            {"role": "user", "content":
             f"Contexte :\n{context}\n\nQuestion : {q.text}"}
        ],
        options={"temperature": 0.2, "num_ctx": 8192}
    )
    return {
        "answer": response["message"]["content"],
        "sources": sources
    }

# Lancer
# python ingest.py
# uvicorn main:app --host 0.0.0.0 --port 8000
# curl -X POST http://localhost:8000/ask -H "Content-Type: application/json" \
#      -d '{"text":"Combien de jours de congés payés par an ?"}'
```

Ce projet de 80 lignes vous donne un assistant entièrement local, conforme RGPD, capable de répondre sur vos documents internes sans qu’aucune donnée ne quitte la machine. Sur un RTX 4060, le temps de réponse moyen est de 2 à 4 secondes pour une question typique. Pour passer en production, ajoutez une authentification (JWT via FastAPI Security), un rate-limit (slowapi), un cache Redis pour les questions fréquentes, et un monitoring Prometheus/Grafana. Vous pouvez aussi remplacer Llama 3.1 8B par Mistral 7B ou Qwen 3 7B selon vos tests de qualité.

## Comparatif Ollama vs alternatives en 2026

Ollama n’est pas seul sur le marché des runtimes LLM locaux. Plusieurs alternatives méritent d’être considérées selon votre cas d’usage. Pour les déploiements à très haute charge (>100 req/s), **vLLM** offre un débit 3 à 9× supérieur grâce à PagedAttention et au continuous batching, au prix d’une complexité d’installation bien plus élevée. Pour les setups graphiques sans code, **LM Studio** propose une UI desktop conviviale mais ferme l’extensibilité. Pour les développeurs Python qui veulent le contrôle total, **llama.cpp** reste la référence bas niveau.

|Critère|Ollama|vLLM|LM Studio|llama.cpp|
|---|---|---|---|---|
|Installation|1 commande|pip + CUDA|App desktop|Compile C++|
|API HTTP native|Oui (/api/*)|Oui (OpenAI)|Oui (OpenAI)|Non (server à part)|
|Format modèles|GGUF + tags|HF safetensors|GGUF|GGUF|
|Débit (8B Q4)|80-110 t/s|200-350 t/s|70-90 t/s|90-120 t/s|
|Multi-modèle|Oui|1 par GPU|1 chargé|1 par instance|
|GPU NVIDIA|Oui (CUDA)|Oui|Oui|Oui|
|GPU AMD (ROCm)|Oui (v0.6.2+)|Limité|Beta|Oui|
|Apple Silicon|Oui (Metal 3)|Non|Oui|Oui|
|OpenAI compat|Oui (/v1)|Oui|Oui|Via wrapper|
|Docker officiel|Oui|Oui|Non|Communauté|
|Cas d’usage|Dev, RAG, équipe|Production scale|Solo no-code|Bas niveau|

Pour 80 % des projets français, Ollama est le bon choix par défaut. Il offre le meilleur ratio simplicité/fonctionnalités, et l’écart de débit avec vLLM ne devient critique qu’au-delà de quelques dizaines de requêtes par seconde simultanées. Pour les déploiements à très grande échelle (chatbot grand public avec des milliers d’utilisateurs concurrents), basculer vers vLLM ou un service géré comme Mistral Le Chat Pro ou Anthropic Claude API devient plus pertinent économiquement.

## Sécurité, RGPD et déploiement en entreprise

Le principal argument d’Ollama en entreprise française est sa nature locale : aucune donnée ne quitte votre infrastructure, ce qui simplifie radicalement la conformité RGPD et AI Act. Cela ne signifie pas pour autant qu’Ollama soit « sécurisé par défaut ». Plusieurs précautions s’imposent en production. D’abord, ne jamais exposer le port 11434 sur Internet sans authentification : il n’y a pas de mécanisme natif d’auth dans Ollama. Placez-le derrière un reverse proxy Nginx ou Traefik avec basic auth ou OAuth2 Proxy.

Ensuite, isolez le réseau : si Ollama tourne sur un serveur dédié, utilisez un firewall (UFW, nftables) pour autoriser uniquement les IP des serveurs applicatifs autorisés. Pour les déploiements multi-tenants où plusieurs équipes partagent un même cluster, considérez un proxy intelligent comme LiteLLM qui ajoute la gestion des clés API, le rate-limiting et l’audit log. Côté logs, Ollama écrit dans `~/.ollama/logs/server.log` sur Linux : vérifiez régulièrement la taille (rotation via logrotate) et auditez les requêtes inhabituelles.

Concernant la conformité AI Act (entré en application en France selon le calendrier du [Digital Omnibus IA reporté au 2 décembre 2027](https://tech-insider.org/fr/digital-omnibus-ia-ai-act-report-2-decembre-2027-2026/)), Ollama lui-même n’est qu’un runtime : ce sont les modèles que vous exécutez qui peuvent tomber sous les obligations de l’AI Act (transparence, évaluation des risques). Llama 3.3 et Mistral sont publiés sous licences permissives mais leurs cartes modèles doivent être archivées si vous les utilisez en production sur des cas à haut risque (RH, santé, éducation).

## Dépannage : 8 problèmes fréquents et solutions

Voici les huit erreurs les plus fréquemment rencontrées par les utilisateurs Ollama, avec leurs solutions vérifiées. Avant de chercher plus loin, consultez toujours `~/.ollama/logs/server.log` ou `journalctl -u ollama -f` qui contiennent presque toujours la cause racine.

|Symptôme|Cause probable|Solution|
|---|---|---|
|« connection refused » sur :11434|Serveur non démarré|`systemctl start ollama` ou `ollama serve`|
|« CUDA out of memory »|Modèle trop gros pour VRAM|Quantification inférieure (Q4 → Q3) ou `num_gpu` réduit|
|Génération < 5 tok/s sur RTX|Modèle sur CPU (partial offload)|Vérifier nvidia-smi, augmenter VRAM libre|
|Pull s’arrête à 99 %|Hash mismatch, registry timeout|`ollama rm` puis `ollama pull` à nouveau|
|« model requires more system memory »|RAM système insuffisante|Activer swap 16 Go ou quantif. plus légère|
|Réponse tronquée à 200 tokens|`num_predict` par défaut bas|Ajouter `"options": {"num_predict": 2048}`|
|Contexte oublié après ~2 000 tokens|`num_ctx` par défaut 2048|Définir `OLLAMA_CONTEXT_LENGTH=8192`|
|Service ne démarre pas au boot|systemd non activé|`sudo systemctl enable ollama`|

Pour les erreurs GPU plus subtiles, l’outil `OLLAMA_DEBUG=1 ollama serve` active des logs verbeux qui montrent exactement combien de couches sont chargées en VRAM versus RAM. Si vous voyez « offload to GPU : 28/33 layers », cela signifie que 5 couches sont restées en RAM système, ralentissant l’inférence. Solution : libérer 1 à 2 Go de VRAM (fermer le navigateur, désactiver l’accélération desktop) ou descendre à une quantification plus légère.

## Astuces avancées : LoRA, function calling, vision

Trois fonctionnalités avancées méritent d’être mentionnées pour les utilisateurs qui dépassent le cas d’usage de base. D’abord les **adaptateurs LoRA** : depuis la v0.4, Ollama peut charger un LoRA fine-tuné par-dessus un modèle de base via `ADAPTER ./mon-lora.gguf` dans le Modelfile. Cela permet de spécialiser un modèle 8B sur votre domaine (jargon médical, juridique, technique interne) sans re-télécharger les 4,7 Go du modèle complet : seul le LoRA de 50 à 200 Mo est ajouté.

Ensuite le **function calling** (tool use) : les modèles compatibles (Llama 3.1+, Qwen 2.5+, Mistral Nemo) acceptent un schéma d’outils JSON dans le prompt, et renvoient des appels de fonction structurés. C’est la brique de base pour construire des agents qui interagissent avec votre SI (consultation CRM, envoi d’emails, requêtes SQL). L’endpoint `/api/chat` accepte le champ `tools` au format OpenAI standard.

```
# Function calling avec /api/chat
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.1:8b",
  "messages": [
    {"role": "user", "content": "Quelle est la météo à Paris ?"}
  ],
  "tools": [{
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "Obtenir la météo actuelle d une ville",
      "parameters": {
        "type": "object",
        "properties": {
          "city": {"type": "string", "description": "Nom de la ville"}
        },
        "required": ["city"]
      }
    }
  }],
  "stream": false
}'

# Réponse contient :
# "tool_calls": [{
#   "function": {"name": "get_weather", "arguments": {"city": "Paris"}}
# }]
```

Enfin la **vision multimodale** : depuis la v0.4, Ollama supporte les modèles vision comme `llava:13b`, `llama3.2-vision:11b` ou `qwen2-vl:7b`. Vous pouvez envoyer une image en base64 dans le champ `images` de la requête. Cas d’usage typiques : OCR de documents, analyse de captures d’écran pour automatisation RPA, description d’images pour accessibilité. Attention, ces modèles consomment 2 à 4 Go de VRAM supplémentaires pour l’encodeur visuel.

## Benchmarks 2026 : tokens/seconde par GPU

Pour vous aider à choisir votre matériel, voici les débits mesurés sur Ollama v0.6.2 avec Llama 3.1 8B Q4_K_M, prompt de 512 tokens, génération de 256 tokens, batch size 1. Ces chiffres proviennent de benchmarks reproductibles publiés par la communauté en avril 2026. Notez que les performances dépendent fortement du driver GPU, du contexte effectif et de la température ambiante (throttling).

|Matériel|VRAM/RAM|Llama 3.1 8B Q4|Llama 3.3 70B Q4|Prix indicatif|
|---|---|---|---|---|
|RTX 4090 24 Go|24 Go|105 t/s|N/A (insuffisant)|1 900 €|
|RTX 4080 Super 16 Go|16 Go|88 t/s|N/A|1 100 €|
|RTX 4070 Ti Super 16 Go|16 Go|82 t/s|N/A|850 €|
|RTX 4060 Ti 16 Go|16 Go|52 t/s|N/A|500 €|
|RX 7900 XTX 24 Go (ROCm 6.3)|24 Go|78 t/s|N/A|1 000 €|
|MacBook Pro M3 Max 128 Go|128 Go unifié|62 t/s|12 t/s|4 200 €|
|MacBook Pro M3 Pro 36 Go|36 Go unifié|48 t/s|N/A|2 600 €|
|Mac Mini M4 Pro 64 Go|64 Go unifié|55 t/s|9 t/s|2 000 €|
|Ryzen 9 7950X (CPU only)|64 Go DDR5|11 t/s|1,8 t/s|1 500 € (config)|
|2× RTX 4090 (NVLink off)|48 Go|180 t/s parallel|22 t/s|3 800 € (GPU seuls)|

Pour un développeur solo, la RTX 4070 Ti Super 16 Go représente le meilleur rapport performance/prix en 2026 : suffisamment de VRAM pour Llama 3.1 8B avec contexte 32k, environ 82 tokens/seconde, et un prix sous les 900 €. Pour une petite équipe, le Mac Mini M4 Pro 64 Go séduit par sa mémoire unifiée qui permet de faire tourner des modèles 32B impossibles sur la plupart des cartes grand public. Pour les budgets serrés, une RTX 4060 Ti 16 Go reste très honnête à 500 €.

## Foire aux questions

### Ollama est-il vraiment gratuit ?

Oui, Ollama est entièrement gratuit et open source sous licence MIT. Le runtime, l’API, le client CLI et l’application desktop sont gratuits. Les modèles eux-mêmes ont leurs propres licences : Llama 3.x est sous licence Meta Community License (gratuit jusqu’à 700 millions d’utilisateurs actifs), Mistral 7B sous Apache 2.0, Qwen sous Tongyi Qianwen License. Vérifiez la licence du modèle si vous comptez l’utiliser commercialement.

### Peut-on faire tourner Llama 3.3 70B sur un MacBook ?

Oui, sur un MacBook Pro M3 Max ou M4 Max avec au moins 64 Go de mémoire unifiée. Le modèle 70B en Q4_K_M occupe environ 40 Go. Le débit observé est de 9 à 12 tokens/seconde, ce qui reste utilisable pour des prompts courts. Sur un Mac Mini M4 Pro 64 Go, c’est aussi possible avec environ 9 tokens/seconde. En dessous de 48 Go de RAM unifiée, Llama 3.3 70B n’est pas réaliste.

### Quelle différence entre Ollama et LM Studio ?

Ollama est orienté CLI et API HTTP, conçu pour s’intégrer dans des scripts, applications et services. LM Studio est une application desktop graphique avec interface chat intégrée, idéale pour les utilisateurs non-développeurs. Les deux utilisent llama.cpp sous le capot et supportent le format GGUF. Pour un développeur qui construit une intégration, Ollama est le bon choix. Pour un usage personnel ponctuel, LM Studio est plus convivial.

### Ollama envoie-t-il des données vers le cloud ?

Non. Une fois un modèle téléchargé via `ollama pull`, toute l’inférence reste 100 % locale sur votre machine. Aucune télémétrie n’est envoyée par défaut. Les seules connexions externes sont : le téléchargement initial des modèles depuis ollama.com, et la vérification optionnelle des mises à jour du binaire. Vous pouvez désactiver cette dernière avec `OLLAMA_NO_AUTOUPDATE=1`.

### Quel modèle choisir pour le français en 2026 ?

Pour un usage généraliste en français, trois modèles se distinguent : **Mistral 7B Instruct v0.3** (excellent niveau de français, contexte 32k, créé en France), **Llama 3.1 8B** (très bon français, contexte 128k, large écosystème), et **Qwen 3 14B** (raisonnement supérieur, multilingue solide). Pour les modèles plus petits (1-3B), `gemma3:4b` reste le plus convaincant en français.

### Comment fine-tuner un modèle avec Ollama ?

Ollama lui-même ne fait pas de fine-tuning. La pratique recommandée est : (1) fine-tuner avec un framework dédié comme **Unsloth** ou **Axolotl** sur un GPU cloud, (2) exporter le LoRA résultant au format GGUF, (3) charger ce LoRA dans Ollama via `ADAPTER ./mon-lora.gguf` dans un Modelfile. Cette séparation permet d’utiliser des GPU H100 pour l’entraînement et votre matériel local pour l’inférence.

### Ollama remplace-t-il l’API OpenAI ?

Pour la plupart des cas d’usage internes, oui. Les modèles open source 8B-70B atteignent maintenant 80-90 % des performances de GPT-4o sur les benchmarks publics, à coût marginal nul après l’investissement matériel. Pour les tâches très avancées (raisonnement multi-étape complexe, code production critique), GPT-5 et Claude 4 restent supérieurs. La stratégie hybride la plus courante : Ollama pour 90 % du trafic interne, API cloud pour les 10 % les plus exigeants.

### Comment migrer du Modelfile vers Hugging Face ?

Vous pouvez télécharger directement un GGUF depuis Hugging Face dans Ollama avec `ollama pull hf.co/utilisateur/modele:Q4_K_M`. Inversement, pour partager un Modelfile public, poussez-le sur ollama.com (`ollama push`) ou exportez le GGUF sous-jacent via `ollama show modele --modelfile`. La compatibilité GGUF est totale : un modèle Ollama est un GGUF standard.

## Conclusion : Ollama en 2026, le standard du LLM local

En treize étapes, ce tutoriel vous a fait passer de zéro à un assistant RAG complet, 100 % local, conforme RGPD par construction. Ollama s’impose en 2026 comme le standard de facto pour exécuter des LLM sans dépendre du cloud, et son écosystème (intégrations LangChain, LlamaIndex, OpenAI SDK, Docker) en fait un choix sans regret pour la majorité des projets français. Les prochaines évolutions attendues — meilleur support multi-GPU, agents natifs, multi-modalité voix — devraient consolider cette position.

### Couverture associée

- [Tutoriel Mistral AI : Chatbot Français en 13 Étapes [2026]](https://tech-insider.org/fr/mistral-ai-tutoriel-chatbot-francais-13-etapes-2026/)
- [Tutoriel CrewAI 2026 : Agents IA Multi-Agents en 13 Étapes](https://tech-insider.org/fr/crewai-tutoriel-agents-ia-python-13-etapes-2026/)
- [Tutoriel Supabase 2026 : Backend Next.js en 13 Étapes](https://tech-insider.org/fr/supabase-tutoriel-backend-nextjs-13-etapes-2026/)
- [Tutoriel Next.js 16 : App Full-Stack en 13 Étapes [2026]](https://tech-insider.org/fr/nextjs-16-tutoriel-app-full-stack-13-etapes-2026/)
- [Mistral Medium 3.5 : 77,6% SWE-Bench, 1,50$ API [2026]](https://tech-insider.org/fr/mistral-medium-3-5-vibe-le-chat-work-mode-avril-2026/)
- [Tutoriel ClickHouse 26.5 : Analytique Temps Réel en 13 Étapes](https://tech-insider.org/fr/clickhouse-26-tutoriel-analytique-temps-reel-13-etapes-2026/)

### Sources externes

- [Bibliothèque officielle des modèles Ollama](https://ollama.com/library)
- [Dépôt GitHub officiel ollama/ollama](https://github.com/ollama/ollama)
- [Documentation officielle de l’API Ollama](https://github.com/ollama/ollama/blob/main/docs/api.md)
- [Modèles GGUF sur Hugging Face](https://huggingface.co/models?library=gguf)
- [Intégration LangChain x Ollama](https://python.langchain.com/docs/integrations/llms/ollama/)

![Nadia Dubois](https://tech-insider.org/wp-content/uploads/2026/03/ti-author-nadia.jpg)

#### [Nadia Dubois](https://tech-insider.org/fr/author/nadia-dubois/)

AI & Innovation Editor

Nadia Dubois est rédactrice IA & Innovation chez Tech Insider, où elle suit l'évolution rapide de l'intelligence artificielle. Elle a précédemment couvert l'IA et les startups pour La Tribune et contribué à la couverture européenne du MIT Technology Review. Nadia est spécialisée dans l'IA générative, la régulation de l'IA et l'intersection entre technologie et politique industrielle européenne. Elle est diplômée de Sciences Po Paris en linguistique computationnelle et journalisme.

[Voir tous les articles](https://tech-insider.org/fr/author/nadia-dubois/)