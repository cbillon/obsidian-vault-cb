---
link: https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/
site: Metal3d
date: 2024-06-27T17:18
excerpt: Et si on utilisait une IA pour analyser l'audio d'un débat politique ? Voici comment mon "IA" a retranscrit les propos de chaque candidat, et comment je  peux l'interroger pour extraire des informations.
tags:
slurped: 2026-03-28T10:39
title: J'ai créé une IA qui analyse un débat politique à la télévision
---

Et si on utilisait une IA pour analyser l'audio d'un débat politique ? Voici comment mon "IA" a retranscrit les propos de chaque candidat, et comment je peux l'interroger pour extraire des informations.

**Et cerise confite sur le gâteau au chocolat noir, je vais tout faire avec des outils open-source.**

C’est important, c’est très important… Nous allons voter dimanche 30 juin pour les législatives. Il y a trois grands groupes politiques qui sont très présents et il se trouve qu’un débat a eu lieu, et qu’on retrouve [le replay sur TF1](https://www.tf1info.fr/elections/en-direct-legislatives-suivez-sur-tf1-le-premier-debat-televise-entre-attal-bardella-et-bompard-2305539.html), où plusieurs sujets ont été abordés.

Ce que je voulais savoir, par cette approche, c’est comment, à de partir d’une donnée très compliquée à analyser, à savoir une vidéo (disons plutôt le son de la vidéo), nous pourrions proposer un outil qui analyse et synthétise le contenu. L’idée étant de pouvoir extraire des propos, les vérifier, résumer, ou même simplifier ce qui est dit.

Vous pourrez adapter les questions à votre guise, et même poser des questions plus complexes. Mais il va falloir passer de la vidéo à quelque chose de compréhensible par une machine.

Pour cela, il y a quelques étapes à suivre. Je vais donc tenter de vous expliquer, pas à pas, comment j’ai procédé. Je vais vous expliquer des concepts en les **vulgarisant** autant que je peux. Car si l’on se penche un peu plus dans les détails cela peut paraitre abstrait, imbitable, et cela peut même décourager.

J’espère que je vais réussir à vous expliquer tout ça de manière simple et claire. Même si certains concepts ne sont clairement pas évidents, j’ai à cœur de vous démontrer que c’est finalement assez accessible. Et si les blocs de code vous rebutent, je vous invite à ne vous pencher que sur les concepts.

Bref, j’espère que je vais être utile.

## Mais avant tout, je dois **impérativement vous mettre en garde**

> Ce que je vous propose ici est avant tout une approche didactique à des fins de recherches et de tests.
> 
> **En aucun cas** une intelligence artificielle ne doit vous orienter votre vote.
> 
> Les débats politiques sont complexes et ont des enjeux propres à chacun. Vous avez aussi votre propre sensibilité aux sujets abordés.
> 
> Cette approche peut générer des données biaisées, erronées ou opposées aux convictions des candidats. Je vous répèterai souvent qu’il faut vérifier les réponses et les données. Malgré tout le soin que j’ai apporté à ces tests, il est évident que je ne peux pas garantir l’exactitude des réponses qui sont générées.

## Les étapes

Il va falloir passer par plusieurs étapes. Ce travail prend du temps. Si vous voulez reproduire ce que je vous propose, il faudra s’armer de patience, certainement corriger les données, faire plusieurs tests (notamment changer de modèle, adapter les méthodes de réponses, lire de la documentation…).

Les étapes sont donc les suivantes :

- Récupération de l’audio du débat et édition du fichier pour supprimer ce qui n’est pas nécessaire (la pub…)
- **Transcription automatisée** de l’audio en texte, qui se déroule en deux étapes :
    - segmenter par phrase et par locuteur (en gros, “qui dit quoi” et quand ?),
    - transcrire chaque phrase en texte (traduire le son en texte)
- **enregistrer les textes en vecteurs** dans une base d’indexation vectorielle (je vais vous expliquer le concept)
- **poser des questions à un LLM** (je vais vous expliquer…), en sortir des analyses, et des idées pour aller encore plus loin

Pour y arriver, on aura besoin de pas mal d’outils.

> Je vais passer outre le téléchargement de l’audio/vidéo pour une raison simple : on est à la limite de l’égalité, car le contenu est potentiellement sous droits d’auteurs. Je me refuse à donner des moyens d’effectuer des téléchargements illégalement depuis des sites de chaines de télévision. Pour autant, croyez-moi, c’est une étape très simple à réaliser. Un coup de Google et vous allez trouver des outils pour cela (genre, cherchez “video downloader”, voilà, j’en ai trop dit).

Par contre, pour la suite, aucun souci. Tout va passer par des outils IA :

- `pyannote.audio` pour le découpage des segments de l’audio, afin d’avoir une série de phrases, et de savoir qui parle et quand
- `pydub` pour la manipulation de l’audio
- `whisper` pour la transcription de l’audio en texte
- `llama-index` pour toute la partie que l’on appelle “RAG” (Retrieval Augmented Generation), en d’autres termes pour la recherche d’informations dans un corpus de texte (pas de panique, je vais vous expliquer)

On utilisera un modèle pour que l’IA réponde à nos questions plus naturellement. J’apprécie beaucoup le modèle [`Qwen2`](https://huggingface.co/Qwen/Qwen2-7B-Instruct), qui est bien entrainé pour répondre en français. Localement, avec LLamaCPP, c’est viable. Mais je n’ai pas des foudres de guerre et donc je vais passer par [l’API de HuggingFace](https://huggingface.co/docs/api-inference/index) pour gagner du temps.

Il faudra utiliser, bien entendu, le modèle au format GGUF que vous trouverez ici [Qwen2-7B-Instruct-GGUF](https://huggingface.co/Qwen/Qwen2-7B-Instruct-GGUF). Les modèles dans ce format sont “quantizés” de différentes manières. Si j’en crois mes tests, c’est le modèle `q4_k_m` qui est le plus probant. Mais vous pouvez tester les autres…

Par chance, `llama-index` propose un package pour utiliser ce service.

> Si vous préférez OpenAI ou avoir un modèle local, voyez ma [note](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/#note-%C3%A0-propos-de-openai-ou-llamacpp) en bas de l’article.

Cette API est gratuite pour un usage modéré, mais personnellement, j’ai un accès “pro” pour 9 €/mois qui repousse la limite d’utilisation. Par contre, dans tous les cas, pour utiliser l’API ou récupérer des modèles, il vous faudra vous enregistrer (gratuitement) sur le site et générer une clef d’API.

Je vous laisse suivre les étapes expliquées sur la page [de Hugging Face](https://huggingface.co/docs/hub/en/security-tokens). Je vous conseille d’ailleurs d’installer le client en ligne de commande [comme indiqué ici](https://huggingface.co/docs/huggingface_hub/guides/cli) pour enregistrer le “token” sur votre poste et ne pas avoir à le mettre dans votre code source. La commande `huggingface-cli login` vous demandera la clef d’API et cela sera plus simple pour la suite.

Vous pouvez aussi adapter tout ce que je vous explique ici pour utiliser les accès à OpenAPI (avec GPT donc), ou même charger un modèle localement selon plusieurs méthodes.

Allez lire la documentation du site [de llama-index](https://docs.llamaindex.ai/en/stable/) qui vous donnera les explications nécessaires pour comprendre comment ça fonctionne.

## Prérequis et préparation

Normalement, tout devrait à peu près fonctionner sur un _notebook_ (comme Google Colab, Jupyter, Kaggle…) - pour ma part, j’utilise des scripts Python directement sur mon poste. Cela me permet de mieux contrôler les étapes et d’éviter des écrasements de variables.

Il vous faut absolument Python sur le poste, **de préférence sur Linux**, et avoir un **GPU** ayant, a minima, **8 Go de VRAM**. Malheureusement, pour la partie “transcription”, les modèles ne supportent que des GPU NVIDIA. Vous pouvez passer par votre CPU, mais vous allez sentir passer le temps[1](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/#fn:1).

## Transcription de l’audio

On part du principe que vous avez récupéré l’audio de l’émission. Vous devez le transformer au format `.wav`, car les autres formats semblent poser un souci. N’oubliez pas de virer les publicités pour ne pas avoir des voix inutiles à classifier. Un outil comme Audacity peut vous aider à éditer l’audio.

Il est temps de se lancer.

Créez un environnement virtuel, installez les dépendances :

```
# on va travailler dans un répertoire dédié
mkdir -p Projects/ML/Débat
cd Projects/ML/Débat

python -m venv venv
source venv/bin/activate
pip install pyannote.audio pydub whisper-openai
```

Le code `segmentazie.py` suivant va créer un fichier `audio.rttm` qui contiendra les segments de l’audio, avec le locuteur. C’est une étape très importante, elle va nous permettre de savoir qui parle et quand.

Avant toutes choses, notez que j’ai supprimé les publicités du fichier audio pour éviter d’avoir à identifier un nombre conséquent de voix.

```
import numpy
import torch
from pyannote.audio import Pipeline
from pyannote.audio.pipelines.utils.hook import ProgressHook

# numpy.NAN is not defined, so we define it here
# I fixed this in a PR: https://github.com/pyannote/pyannote-audio/pull/1732
setattr(numpy, "NAN", numpy.nan)

INPUT_FILE = "input.wav"

pipeline_model = "pyannote/speaker-diarization-3.1"

# check if cuda is available
device: torch.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

pipeline = Pipeline.from_pretrained(pipeline_model).to(device)

# apply the pipeline on the input file
with ProgressHook() as hook:
    diarization = pipeline(file=INPUT_FILE, hook=hook)

# dump the diarization output to disk using RTTM format
with open("audio.rttm", "w", encoding="utf-8") as rttm:
    diarization.write_rttm(rttm)
```

Le fichier `audio.rttm` est un fichier texte qui contient ce genre de lignes :

```
SPEAKER input_truncated 1 0.115 10.598 <NA> <NA> SPEAKER_01 <NA> <NA>
SPEAKER input_truncated 1 10.713 1.316 <NA> <NA> SPEAKER_04 <NA> <NA>
SPEAKER input_truncated 1 12.029 3.493 <NA> <NA> SPEAKER_01 <NA> <NA>
SPEAKER input_truncated 1 21.293 12.521 <NA> <NA> SPEAKER_01 <NA> <NA>
SPEAKER input_truncated 1 34.135 20.672 <NA> <NA> SPEAKER_04 <NA> <NA>
SPEAKER input_truncated 1 55.094 15.711 <NA> <NA> SPEAKER_01 <NA> <NA>
SPEAKER input_truncated 1 71.311 12.572 <NA> <NA> SPEAKER_04 <NA> <NA>
SPEAKER input_truncated 1 83.782 19.862 <NA> <NA> SPEAKER_01 <NA> <NA>
SPEAKER input_truncated 1 103.660 12.690 <NA> <NA> SPEAKER_04 <NA> <NA>
SPEAKER input_truncated 1 117.093 13.601 <NA> <NA> SPEAKER_02 <NA> <NA>
```

On ne va pas détailler, mais il faut retenir que (en partant de la colonne 0) :

- La colonne 3 est le début du segment
- La colonne 4 est la durée du segment
- La colonne 7 est le “locuteur” - évidemment que c’est un identifiant “anonyme” car le script ne connait pas les voix, il sait juste que c’est une personne différente qui parle.

Pour ma part, j’ai écouté quelques segments en me basant sur les temps de début et j’ai pu trouver que (dans mon cas) :

- `SPEAKER_01` est Anne-Claire Coudray
- `SPEAKER_04` est Gilles Bouleau
- `SPEAKER_02` est Manuel Bompard
- `SPEAKER_05` est Gabriel Attal
- `SPEAKER_08` est Jordan Bardella

Les autres segements sont généralement les moments où les candidats parlaient en même temps. Je les omets volontairement pour la suite.

## Transcription de l’audio en textes

Maintenant que je sais qui parle et quand, je vais pouvoir transcrire les phrases en texte. On va utiliser `whisper`, un modèle open-source de OpenAI, très bien entrainé et franchement très bon.

J’ai tenté d’utiliser le modèle “de base” (`base`) mais les résultats étaient assez mauvais. Le modèle `medium` a été plus adapté. J’aurais adoré tester avec un modèle plus large, et par conséquent plus précis, mais ma machine a un peu de mal à tout gérer.

Ce que je vais faire, ici, c’est de créer des fichiers texte dans un répertoire par candidat. Cela permettra d’indexer les propose plus facilement par la suite.

Voilà donc le script `transcription.py` :

```
import os

import torch
import shutil
import whisper
from pydub import AudioSegment

INPUT_FILE = "input.wav"

# model to use
model_type = "medium"  # tiny, base, small, medium, large

# open the audio file
audio = AudioSegment.from_file(INPUT_FILE, format="wav")
# open the diarization file
dz = open("audio.rttm", "r").read().splitlines()

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# load the model
model = whisper.load_model(model_type, device=device)

# only the candidates
speakers = {
    "SPEAKER_02": "Manuel Bompard",
    "SPEAKER_05": "Gabriel Attal",
    "SPEAKER_08": "Jordan Bardella",
}

# delete and recreate transcriptions/xxx
shutil.rmtree("transcriptions", ignore_errors=True)
for speaker in speakers.values():
    os.makedirs("transcriptions/" + speaker, exist_ok=True)

current = ""
for line in dz:
    line = line.split()

    # get the speaker and the segment
    speaker = line[7]
    if speaker not in speakers:
        # not a candidate
        continue

    # get the real name of the speaker
    speaker_name = speakers.get(speaker)

    # get the start and the duration and convert to milliseconds
    start = float(line[3]) * 1000
    duration = float(line[4]) * 1000

    # get the segement
    audio_slice = audio[start : (start + duration)]

    # one channel, 16kHz
    audio_slice = audio_slice.set_channels(1).set_frame_rate(16000)

    if audio_slice.raw_data is None:
        continue

    # transcribe with raw data (converted)
    text = whisper.transcribe(
      model,
      audio=np.frombuffer(audio_slice.raw_data, dtype=np.int16).astype(np.float32)/32768.0,
      language="fr",
    )

    # print (optional)
    if speaker != current:
      # new speaker...
      print(f"--- {speaker_name} ---")

    print(text["text"])

    # append the text to the file in the speaker directory
    with open(f"transcriptions/{speaker_name}/propos.txt", "a", encoding="utf-8") as f:
        f.write(text["text"] + "\n")

    current = speaker
```

Cela m’a généré 3 fichiers (dans le répertoire “transcriptions”) :

- `Manuel Bompard/propos.txt`
- `Gabriel Attal/propos.txt`
- `Jordan Bardella/propos.txt`

J’aurais pu, bien entendu, aller plus loin, en ajoutant des contextes de question des journalistes, des réponses, voire structurer les données. Mais je n’ai pas un temps infini[2](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/#fn:2).

> **Attention** whisper se trompe de temps en temps, et parfois `pyannote` attribue des propos à la mauvaise personne. Donc, clairement **tout ce qui va suivre** doit être pris avec des pincettes. Partez du principe qu’il faut normalement **tout vérifier** et corriger les données. Or, je n’ai pas le temps, et là je vous propose uniquement une explication dans un contexte d’actualité. Mon IA finale n’est pas précise, elle demanderait des heures de travail pour être plus pertinente et moins dans l’erreur. J’espère avoir été clair : attention à ce que vous allez lire !

## Les LLM

Avant d’aller plus loin, un petit rappel sur ce que sont les “LLM” (Large Language Models).

Ce sont des modèles, c’est-à-dire des gros fichiers qui représentent des calculs à faire (des milliers, voir des millions ou des milliards de paramètres) qui sont adaptés pour “traiter et générer du texte”. On parle de “modèles de langage”.

Vous avez forcément entendu parler de ChatGPT, Gemini, Claude, Mistral… ce sont des outils qui utilisent ces modèles pour vous donner des réponses qui semblent cohérentes et naturelles.

> Ils ont, par contre, un énorme défaut !

Ils sont “entrainés” à un instant T, sur des données qui sont “statiques”. Cela veut dire que, si vous leur posez une question sur un sujet dont elle n’a pas connaissance, ce qui est très vrai en ce qui concerne l’actualité, eh bien, il peut avoir trois réactions possibles :

- dire qu’il ne sait pas… ce qui est très rare
- donner une réponse qui n’est pas à jour
- raconter n’importe quoi, on dit qu’il _hallucine_

Par exemple, si je demande à ChatGPT (en 2024) la liste des “femmes premières ministres” que nous avons eu en France…

![ChatGPT n’est pas à jour](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/chatgpt-ministre.png)

Et oui… son entrainement date de 2021, il n’est donc pas au courant…

En ce qui concerne les cas d’hallucination, par exemple, je lui ai demandé de me donner les trois premiers ministres “femmes” qu’on a eu en France. Selon d’où vient le vent, il me sort trois noms. Alors qu’en 2024, nous n’avons eu que deux femmes premières ministres (E. Cresson et E. Borne).

Pour la blague, il m’a cité “Ségolène Royal” dans la liste. Il a clairement halluciné la réponse 😉.

> Mais pourquoi il “hallucine” ? Il est bête ou quoi ? Il ment ouvertement ?

Parce que le modèle cherche à donner une réponse _coute que coute_. Il va potentiellement commencer une phrase dont la suite va être fausse. Qu’à cela ne tienne, il va continuer pour être cohérent.

> Hors, la cohérence n’est pas forcément vraie.

Comprenez bien que son but n’est pas de donner une vérité, mais de “fournir une réponse”. Il ne cherche pas l’information, il génère du texte qui semble bon. Si l’information est biaisée, ou si elle parait ouverte, il va inventer sans s’en rendre compte. Il faudrait qu’il ait vu une information viable pour donner une réponse viable.

> Alors pourquoi on ne les entraine pas tous les jours ? En continu ?

Parce que c’est long. Entrainer un modèle GPT prend **des semaines, voire des mois !** Et ça coûte cher. Très cher. Il faut des machines très puissantes, qui avalent de l’électricité comme Gérard enquille les litres de rouge. Vraiment, c’est catastrophique.

> Et ça, c’est l’erreur que beaucoup font aujourd’hui : penser qu’un modèle LLM est une “source de vérité”. Un modèle LLM ne sert par à donner de l’information, il sert à donner une réponse “naturelle”. La question qu’on lui pose oriente la réponse. Si la question est mal posée, la réponse sera fausse.
> 
> **Arrêtez d’utiliser ChatGPT comme si c’était un moteur de recherche !**

C’est pour cela qu’on utilise de plus en plus la méthode “RAG”, dont on va parler plus tard, qui consiste à donner de l’information à jour, contextualisée, au modèle.

## Vectorisation de termes

Cette phase est la plus “compliqué”. Même si, clairement, `llama-index` est absolument génial et va vous mâcher tout le travail, il faut prendre le temps de comprendre comment ça fonctionne.

> Si vous n’avez jamais entendu parler de “vectorisation de texte”, que l’on appelle aussi `embeding`, je vous fais un topo rapide.

En français, et en anglais aussi, on utilise un terme pour définir “comment comprendre un mot”, et c’est assez fou que ce terme soit exactement la définition dont on a besoin. **On dit qu’un mot à un “sens”.** Vous ne vous rendez pas compte à quel point dire cela est juste. D’ailleurs, on parle de direction quand on évoque “où veut aller le discours”, et on dit clairement “je ne vois pas où tu veux en venir”.

> Car, en réalité, un “sens” indique une direction.

Alors prenons le mot “sens” au pied de la lettre. Imaginons que l’on dessine une flèche pour définir un mot.

![Aimer, vecteur](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/aimer.svg)

Donc, deux mots “proches” iront à peu près dans la même direction, tandis qu’un mot opposé ira dans le sens inverse.

![Vecteur inverse](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/aimer-adorer-d%C3%A9tester.svg)

Et vous savez quoi ? Un sens, une flèche, en géométrie, on appelle ça un **vecteur**. On peut le définir mathématiquement. Un vecteur, en mathématique, c’est une direction et une longueur. On peut définir la longueur du vecteur à partir des coordonnées.

![Vecteur avec coordonnées](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/vecteur-x-y.svg)

Chose encore plus dingue, si on se débrouille bien, une addition de vecteurs peut donner un autre vecteur. Ce vecteur résultant aura aussi un sens. Cela veut dire que, si on trouve une méthode pour savoir la direction de chaque mot, alors :

- on peut additionner les vecteurs de chaque mot d’une phrase, et avoir le sens de la phrase
- des vecteurs proches auront des sens proches

Cela veut dire que la “sémantique” est mathématiquement représentable. C’est ce que l’on appelle la “vectorisation de texte”.

![Phrase vectorisée](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/sentence.svg)

> Ici, pour vulgariser, j’ai utilisé des flèches sur un plan. Donc en deux dimensions (2D). On peut imaginer qu’un vecteur soit défini par trois dimensions (3D)…

Par exemple, “Chat” et “Chien” sont proches, mais “Serpent” l’est moins. Si les axes ont une signification propre au modèle (par exemple s’il approche les concepts de “mammifère”, “animal”, “domestique”, etc.), alors on peut imaginer ce genre de vecteur en trois dimensions :

![vecteur en 3 dimensions](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/vector-3d.svg)

> Sauf que mathématiquement, on n’a aucune limite. Les modèles utilisés pour la sémantique gèrent des vecteurs en 128, 256, 512, 1024 dimensions… Le cerveau humain ne peut pas imaginer ce genre de flèche, de vecteur. Mais on peut le représenter mathématiquement. Cela donne énormément d’informations sur le **sens** d’un mot.

Ainsi, un mot comme “chat” peut être représenté comme ceci:

```
[-0.255, 1.25, 6.32, -0.23, 1.2, 6.3, ...]
```

> Comment on choisit ces valeurs ?

Eh bien ce n’est pas à nous, humains derrière le clavier, de décider. On utilise des algorithmes qui vont trouver tout seul les valeurs qui rapprochent ou éloignent les concepts. Ce sont des “régressions statistiques” qui vont trouver ces données, et c’est bien assez compliqué pour que je vous dise “ayez confiance, ça marche”.

Nous, de toutes manières, on ne va pas se prendre la tête, on va utiliser des modèles déjà entrainés qui savent donner un vecteur à chaque mot.

Par contre, c’est bien beau de savoir vectoriser des mots, mais ça sert à quoi au juste ?

Le but, c’est d’être capable, à terme, de pouvoir trouver des parties d’un document qui “semble parler d’un sujet en particulier”. Cela veut dire que, si je cherche des documents qui parlent de “chien”, je vais chercher les vecteurs qui se rapprochent le plus du vecteur “chien”. Et je vais certainement tomber sur des documents qui parlent de “canidés”, de “toutous”, de “cabots”, etc. Parce que les vecteurs qui représentent ces mots sont proches de celui de “chien”. On dit que leur “distance vectorielle est proche”.

La méthode la plus simple est donc de découper mon texte en petits morceaux (en phrases, ou en sections de texte), et d’enregistrer chaque morceau et ses vecteurs dans une base de données adaptée.

![Indexation vectorielle](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/RAG-Vector.-index.svg)

Cela veut dire que, si j’ai une “question” à poser, je peux la transformer en un vecteur et chercher dans la base tous les bouts de texte qui ont un sens proche. Alors une fois de plus, on a des outils pour ça, on ne va pas faire de maths bien énervés nous-même. Reposons-nous sur le travail fait par des gens qui nous ont facilité la tâche.

Maintenant, comprenez l’intérêt dans notre exemple. On va pouvoir faire ce genre de choses :

- chercher les propos de chaque candidat dans une base pour un sujet donné, une question, etc.
- construire une question à donner à un modèle de génération de texte pour avoir une réponse “naturelle”

> C’est ce que l’on appelle, le **RAG (Retrieval Augmented Generation).**

**Je réponds à une question qu’on m’a posé dernièrement :**

> Mais pourquoi passer par des vecteurs tout compliqués alors que j’ai un moteur de base de données qui sait faire de la recherche “full text” (recherche de mots) ?

Eh bien parce que le but n’est pas de trouver des textes qui contiennent des “mots clefs” trouvés dans la question. Le but est de trouver des bouts de textes qui ont un sens proche de la question. Cela veut dire que, si je pose une question “Trouves moi un truc à me mettre sur la tête”, je ne veux pas trouver les textes qui parlent de “porter” et “tête”. Je veux qu’il me trouve des textes qui parlent de bonnets, de chapeaux ou de couronnes. Même si je n’ai pas utilisé ces mots. Seule la sémantique, le sens, compte. Et la vectorisation de texte est la seule technique qui permet de faire ça efficacement.

En fait, on pourrait se passer de la vectorisation, mais les résultats seraient moins précis. On aurait des résultats parasites, des réponses ambivalentes.

Ah, une précision tout de même, on pourrait utiliser Elastic Search ou TypeSense qui propose déjà une vectorisation des termes indexés. Mais, à mon sens, c’est utile si vous avez besoin de la base pour d’autres choses. Nous, ici, on va faire du “one shot”, du test ponctuels, donc on va passer par une base en mémoire, adaptée à la recherche sémantique pour ce cas de figure.

## Et donc, t’as parlé de RAG, c’est quoi ?

Eh bien, c’est une “technique” qui consiste à utiliser un moteur de recherche pour extraire des informations à partir d’une question, puis de compiler une question avec ce contexte qu’on va poser à un modèle LLM. Et par conséquent d’avoir une réponse “naturelle” et cohérente basée sur des informations à jour dans un contexte bien déterminé.

> En gros, on oriente la réponse que donne une IA en lui donnant des informations.

C’est, en fait, assez “simple” conceptuellement parlant.

La question posée par un utilisateur est utilisée deux fois :

- pour trouver les bouts de texte dans la base, afin d’avoir un “contexte” d’informations
- pour demander à l’IA de répondre à la question **en prenant en compte le contexte** qu’on lui injecte

![RAG création d’un prompt](https://www.metal3d.org/blog/2024/ia-pour-analyser-un-d%C3%A9bat/RAG-query.svg)

Cela a pour effet d’éviter les “hallucinations” des modèles, car on leur donne des informations à jour, contextualisées.

Le LLM en bout de chaine, il sert à deux choses finalement :

- arriver à analyser pour résumer la réponse de manière cohérente
- donner une réponse “naturelle” à l’utilisateur

> Comme un “LLM” est un modèle “génératif”, et qu’on “retrouve” des informations qu’on injecte à notre question, vous comprenez que RAG (Retrieval Augmented Generation) veut bien dire “Génération Augmentée par la Recherche”. Eh ouais 😄

Dans les faits, on peut pousser le concept très loin. Par exemple en envoyant la réponse une seconde fois dans le moteur avec d’autres informations issue d’une autre recherche basée sur la réponse. On peut ainsi affiner la réponse (“méthode Refine”).

llama-index propose d’ailleurs plusieurs techniques de recherche, et de génération de résultat. On peut par exemple faire de la recherche en mode “refine”, “compact” ou “summarize”, et avoir des arbres de réponses pour connaitre le document sur lequel il trouve les réponses, etc.

Pour l’heure, on fait simple. Mais vous pouvez vous amuser un peu si vous voulez.

Je vous invite à lire leur documentation, vous verrez que c’est très complet. Mais **vraiment** très complet.

## Implémentation du RAG

Nous allons créer un second environnement virtuel parce que LLamaIndex utilise une version de `numpy` incompatible avec celle que nous utilisons pour `pyannote.audio`.

Nous installons `chromadb` pour indexer les documents, on va l’utiliser en mode `in memory`, car nous n’avons pas beaucoup de documents à indéxer, ce sera donc rapide et nous n’aurons pas à stocker tout ça sur le disque.

On pourrait utiliser le système d’indexation par défaut, intégré à `llama-index`. Cependant, `chromadb` est plus adapté et mieux optimisé pour les recherches sémantiques.

Il nous faudra aussi quelques sous-paquets pour accéder à des modèles LLM. Dans mon cas, je vais utiliser l’API d’inférence de HuggingFace (parce que c’est rapide, simple et efficace). Mais si vous désirez utiliser un modèle local, vous pouvez changer un peu le code et utiliser `llama-index-llms-llamacpp` pour charger un modèle au format `GGUF` par exemple.

```
# quittez l'environnement virtuel précédent
deactivate

# créez un nouvel environnement virtuel
python -m venv venv-llama
source venv-llama/bin/activate

# installez les dépendances
pip install chromadb
pip install llama-index llama-index-embeddings-huggingface llama-index-llms-huggingface llama-index-vector-stores-chroma
```

Le code suivant est à peu près clair.

- Nous créons une base chroma en mémoire
- Pour chaque candidat, je crée une collection séparée (comme ça je peux trouver les réponses pour un candidat)
- Ensuite, je boucle sur les candidats :
    - J’indexe les propos.
    - Je stocke les vecteurs dans la collection du candidat.
    - Je garde le moteur de requête pour pouvoir poser des questions.

Il me reste enfin à avoir une base de questions. Pour chaque question à poser, je la pose à chaque candidat.

Lisez les commentaires dans le code, c’est important si vous voulez comprendre ce que je fais, et comment l’adapter à vos besoins.

```
from llama_index.llms.huggingface import HuggingFaceInferenceAPI
from llama_index.embeddings.huggingface import HuggingFaceEmbedding
from llama_index.vector_stores.chroma import ChromaVectorStore
from llama_index.core import (
    SimpleDirectoryReader,
    StorageContext,
    VectorStoreIndex,
    PromptTemplate,
)
import chromadb

# une base ChromaDB épheémère, en mémoire
chroma_client = chromadb.EphemeralClient()

# Une lsite de candidats avec leur collection associée. Bien entendu,
# j'aurais pu automatisé en listant les répertoires, mais pour vous clarifier
# le code... je fais comme ça
collections: dict[str, dict] = {
    "Jordan Bardella": {
        "chroma": chroma_client.create_collection("bardella"),
        "query": None,
    },
    "Manuel Bompard": {
        "chroma": chroma_client.create_collection("bompard"),
        "query": None,
    },
    "Gabriel Attal": {
        "chroma": chroma_client.create_collection("attal"),
        "query": None,
    },
}

# le prompt "de base" pour contextualiser la manière dont me répondra le LLM.
# Je force un peu la main pour avoir des réponses en français et contextualisées sur
# le domaine politique.
system_prompt = " ".join(
    [
        "Tu est un électeur qui cherche des réponses à des question sur les candidats à l'élection legislative.",
        "Tu réponds en français.",
    ]
)

# LLM à utiliser. J'utilise ici l'API de HuggingFace, mais vous pouvez
# passer par OpenAI, ou charger un modèle avec LLamaCPP. La doc est assez
# fournie
# model = "mistralai/Mistral-7B-Instruct-v0.3" # peu précis et contradictoire
model = "HuggingFaceH4/zephyr-7b-alpha" # le meilleur durant mes tests
# model = "HuggingFaceH4/zephyr-7b-beta" # me donne souvent des textes vides...
llm = HuggingFaceInferenceAPI(
    model_name=model,
    system_prompt=system_prompt,
)
# Voir la note dans l'article, plus bas, pour OpenAI ou LLamaCPP

# le moteur de vectorisation de texte. J'utilise ici un modèle léger, largement suffisant
# pour ce genre de tâche (il tournera en local, donc il va être téléchargé)
embeddings = HuggingFaceEmbedding(model_name="BAAI/bge-base-en-v1.5")

# les templates pour le LLM. Parce que je veux des réponses en francais, je dois
# adaper les templates. Si vous voulez bosser en anglais, c'est inutile. llama-index
# a déjà des templates utilisés par défaut.
text_qa_template_str = (
    "Le contexte de l'information est le suivant :"
    "\n---------------------\n{context_str}\n---------------------\n"
    "En utilisant à la fois ces informations de contexte et tes propres connaissances, "
    "réponds à la question suivante : {query_str}\n"
    "Si le contexte n'est pas utile, tu peux également répondre à la question par toi-même.\n"
)
text_qa_template = PromptTemplate(text_qa_template_str)

# seulement utile si on veut changer le mode de réponse en mode "refine"
refine_template_str = (
    "La question originale est la suivante : {query_str}\nNous avons fourni une"
    " réponse existante : {existing_answer}\nNous avons la possibilité d'affiner"
    " la réponse existante (uniquement si nécessaire) avec un peu plus de contexte"
    " ci-dessous.\n------------\n{context_msg}\n------------\nEn utilisant à la fois"
    " le nouveau contexte et tes propres connaissances, mets à jour ou répètes la réponse"
    " existante.\n"
)
refine_template = PromptTemplate(refine_template_str)

# C'est parti. Puor chaque candidat, on va indexer les propos qui se trouvent dans le répertoire
# qui porte le nom du candidat.
for candidate, col in collections.items():
    path = f"transcriptions/{candidate}" # répertoire qui contient les propos du candidat
    storage = StorageContext.from_defaults(
        vector_store=ChromaVectorStore(col["chroma"]), # collection du candidat
    )
    # on lit les documents (les propos dans le fichier texte)
    documents = SimpleDirectoryReader(path).load_data(show_progress=True)
    # on indexe les documents (vectorisation et enregistrement dans la base)
    vector_store = VectorStoreIndex.from_documents(
        documents=documents,
        storage_context=storage,
        llm=llm,
        embed_model=embeddings,
        show_progress=True,
    )
    # on crée un moteur de recherche sémanitque qui répondra avec le LLM et en utilisant
    # le ou les templates de questions
    query_engine = vector_store.as_query_engine(
        llm=llm,
        embed_model=embeddings,
        text_qa_template=text_qa_template,
        refine_template=refine_template,
    )
    col["query"] = query_engine # on garde ce moteur pour poser des questions

# donc là on a "collections" qui contient, pour chaque candidat, un moteur de "requête"
# qui va nous permettre de poser des questions et d'avoir des réponses contextualisées

# les questions à poser
questions = [
    "Que propose le candidat pour lutter contre le chômage ?",
    "Que propose le candidat pour augmenter le pouvoir d'achat ?",
    "Que propose le candidat pour lutter contre le réchauffement climatique, l'environnement, les énergies fossiles et renouvelables ?",
    "Que propose le candidat pour améliorer le système de santé, les déserts médicaux, l'accès aux soins ?",
    "Que propose le candidat pour améliorer le système éducatif, scolaire, écoles ?",
]

# et c'est parti... on pose les questions
for question in questions:
    print(f"=== Question: {question} ===")
    for candidate, col in collections.items(): #pour chaque candidiat on pose la question
        print(f"=> Candidat: {candidate}")
        response = col["query"].query(question)
        print(response)
        print()
```

En exécutant ce script, nous avons des réponses clarifiées, propres, contextualisées et plus condensées.

Voici un extrait de la sortie dans le terminal, **attention, je vous rappelle que l’IA peut se tromper, mal interpréter, voir halluciner une réponse**. Ne prenez pas ces réponses pour argent comptant, ne les utilisez pas pour prendre votre décision dimanche 30 juin. Ceci est un exemple, rien d’autre !

Avec le modèle `MisralAI/Mistral-7B-Instruct-v0.3`, voici un exemple de sortie (que je trouve moyen) :

```
=== Question: Que propose le candidat pour lutter contre le réchauffement climatique, l'environnement, les énergies fossiles et renouvelables ? ===

=> Candidat: Jordan Bardella
Le candidat propose de mettre en place un moratoire sur la construction de tout nouveau chantier éolien, car il estime
que 25% du temps, les énergies éoliennes tournent à vide et que l'énergie ne se stocke pas, ce qui oblige à brader l'énergie.
Il souhaite également refaire de la France un paradis énergétique en défendant le nucléaire, qui permet à la France d'avoir
une énergie pas chère, décarbonée et bonne pour les factures et pour les entreprises. Il souhaite également déverrouiller
les contraintes qui pèsent sur la croissance pour les entreprises, réindustrialiser, relocaliser et simplifier l'économie pour
créer les conditions d'une diminution de l'empreinte carbone de la France sur la planète et de celle de l'Union européenne.
Il estime également que la gestion stratégique de l'eau pose des enjeux très concrets pour les agriculteurs.

=> Candidat: Manuel Bompard
Le candidat Manuel Bompard propose de faire des investissements publics massifs pour la transition écologique, notamment
dans la rénovation thermique des logements. Il considère que la construction de nouveaux réacteurs nucléaires ne peut
pas être une solution à l'urgence climatique, car les efforts de réduction des émissions de gaz à effet de serre qu'il
faut faire doivent être faits dans un temps qui ne permet pas la construction de ces réacteurs nucléaires.

=> Candidat: Gabriel Attal
Le candidat propose de continuer à investir pour décarboner notre industrie, mais pas via la décroissance comme le propose monsieur Bonpard.
Il a engagé un plan pour relancer le nucléaire avec 14 nouveaux réacteurs et les chantiers ont déjà démarré.
En 2035, on aura des nouveaux réacteurs qui pourront entrer.
Il a également promis de continuer à investir dans les énergies renouvelables, mais il a souligné que si on veut baisser
nos émissions de CO2, si on veut être vraiment indépendant en matière d'énergie, il faut continuer à installer des éoliennes.
Il a également souligné que les éoliennes nouvelles qui sont installées chaque année, c'est l'équivalent d'un réacteur nucléaire supplémentaire.
```

En utilisant, cette fois, le modèle `HuggingFaceH4/zephyr-7b-alpha`, voici un autre exemple de sortie, que je trouve plus pertinent et cohérent :

```
=== Question: Que propose le candidat pour lutter contre le réchauffement climatique, l'environnement, les énergies fossiles et renouvelables ? ===

=> Candidat: Jordan Bardella
Le candidat propose de lutter contre le réchauffement climatique en faisant de la France un pays de producteurs
et en relocalisant les industries. Il souhaite refaire de la France un paradis énergétique en soutenant
le nucléaire, qui est un atout incontesté pour la France et l'Union européenne.
Il pense que les énergies renouvelables doivent être sécurisées, mais il souhaite également arrêter les éoliennes
et mettre en place un moratoire sur la construction de nouveaux chantiers éoliennes en raison de leur faible efficacité
énergétique. Il est opposé à l'importation de véhicules électriques trop chers et à leur coût de réparation trop élevé
, qui rendraient les véhicules électriques inaccessibles pour les classes populaires et moyennes. Il souhaite
donc simplifier l'économie et déverrouiller les contraintes qui pèsent sur la croissance pour les entreprises.

=> Candidat: Manuel Bompard
Le candidat Manuel Bompard propose de faire des dépenses significatives, très importantes, de plusieurs dizaines de milliards
d'euros pour la transition écologique, parce que c'est le défi du siècle, et que ce défi du siècle, on ne pourra y répondre
que par des investissements publics massifs.
Il considère que face à l'urgence climatique que nous avons devant nous, c'est une très mauvaise décision de faire des coupes budgétaires
 sur le budget de la transition écologique et en particulier sur la rénovation thermique des logements. Il estime que
pour lutter contre le réchauffement climatique, il faut réduire massivement les émissions de gaz à effet de serre d'ici à 2030,
d'ici à 2035, d'ici à 2040. Il estime que présenter la construction de nouveaux réacteurs nucléaires comme une solution à
l'urgence climatique est tout simplement faux et inexact.

=> Candidat: Gabriel Attal
Le candidat propose de continuer à investir pour décarboner notre industrie et non pas via la décroissance comme le
propose le candidat de l'autre parti. Il soutient la construction de 14 nouveaux réacteurs nucléaire
s et les chantiers ont déjà démarré. Il a également promulgué une loi de fermeture centrale nucléaire, mais il affirme
que les premières centrales nucléaires seront en place en 2035. Il soulève la question de savoir
comment les Français pourront répondre à la demande croissante d'électricité si les éoliennes sont arrêtées alors que les
nouveaux réacteurs nucléaires ne seront pas en place avant 2035. Il affirme que les éoliennes
nouvelles qui sont installées chaque année, c'est l'équivalent d'un réacteur nucléaire supplémentaire. Les parcs actuels
arrivent à échéance dans combien de temps ?
```

En fonction du modèle, nous avons des divergences de résultats. Pour autant, le sens reste à peu près le même et les informations extraites sont assez cohérentes à ce que j’ai pu entendre en direct.

Mais je le répète : **je n’ai pas nettoyé les données d’extraction audio, et il est possible d’ailleurs que certains propos ne soient pas attribués à la bonne personne.**

Il faudrait aussi que j’adapte les prompts pour les rendre plus précis, plus ouverts, ou au contraire plus strict sur la réponse attendue.

> Mais, est-ce que cela est pertinent ?

Alors… si je pars “en connaissance de cause”, en sachant qu’il manque certainement des informations, que la transcription demande des corrections, etc. Oui, c’est pertinent. Oui cela a bien résumé les propos, et oui, je trouve à peu près ce que je cherche dans le débat qui a duré plus d’une heure et demie.

Aussi, je retrouve à peu près les mêmes informations que j’ai pu entendre. Clairement, avec peu de moyens et de temps, j’ai déjà un résultat qui est assez satisfaisant.

Oui, cela est utile, oui, on peut s’en servir. Mais il est **impératif** de prendre du recul et de **vérifier** les propos extraits par l’IA. Car clairement, elle se trompe de temps en temps.

> La règle est claire : doutez de l’IA. Elle est comme un pote à qui vous parlez, elle sort un avis, basé sur ses compétences d’analyse, mais elle a des biais, et en plus vos données sont potentiellement cassées.

## Note à propos de OpenAI ou LLAmaCPP

Si vous ne voulez pas utiliser l’API de HuggingFace, vous pouvez utiliser OpenAI ou LLamaCPP.

**OpenAI**, installez le paquet `llama-index-llms-openai` et utilisez le modèle `OpenAI/gpt-3.5-turbo`. Remplacez la variable `llm` par :

```
llm = OpenAI(
    system_prompt=system_prompt,
)
```

**LLamaCPP**, vous devez installer, en premier lieu, le package [LamaCPP-python](https://github.com/abetlen/llama-cpp-python) en précisant le “backend” que vous voulez.

Pour ma part, j’en ai marre d’utiliser CUDA, donc je passe par Vulkan :

```
CMAKE_ARGS="-DLLAMA_VULKAN=on" pip install llama-cpp-python
```

Puis, installez le paquet `lla-index-llms-llamacpp` et utilisez le modèle `GGUF`. Remplacez la variable `llm` par :

```
llm = LLamaCPP(
    model_url="ici l'url d'un modèle GGUF",
    system_prompt=system_prompt,
    # model_kwargs={"n_gpu_layers": 28}, # si vous avez un GPU avec peu de mémoire
    # model_kwargs={"n_gpu_layers": -1}, # si vous voulez tout charger dans le GPU
)
```

L’intérêt de LLamaCPP, c’est que vous pouvez charger des modèles GGUF qui débordent de la VRAM sur la RAM du système et de répartir le travail en partie sur le CPU. On appelle cela du “offloading”. C’est très utile si vous avez un GPU trop léger. Par contre, **c’est plus lent**. Pour une RTX 3070 avec 12 Go de VRAM, je suis obligé de réduire le nombre de couches (28 sur les 31 de Qwen2) à charger dans la carte. Cela demande donc au CPU de gérer 2 couches. Ça reste utilisable, mais on entend les ventilateurs tourner…

Par exemple, pour un modèle Qwen2-7B-Instruct-GGUF, j’utilise `https://huggingface.co/Qwen/Qwen2-7B-Instruct-GGUF/resolve/main/qwen2-7b-instruct-q4_k_m.gguf`

Le compte “The Block” propose un grand nombre de modèles GGUF, vous pouvez les tester : [https://huggingface.co/TheBloke](https://huggingface.co/TheBloke), par exemple les modèles Llama2 en 7B se trouvent ici : [https://huggingface.co/TheBloke/Llama-2-7B-GGUF/tree/main](https://huggingface.co/TheBloke/Llama-2-7B-GGUF/tree/main)

## Encore plus loin

Je ne vous colle pas le code ici, car l’article devient long. Mais j’ai ajouté pas mal de traitements supplémentaires à mon script :

- pour chaque réponse, je fais demande à l’IA de me sortir une requête que je pourrais donner à un moteur de recherche,
- cette requête, je l’envoie à Google (ou autre) pour trouver des pages d’information,
- ces pages, je les indexe
- ensuite, je lance une recherche sémantique sur ces pages pour avoir des informations,
- je demande au LLM de me dire si le candidat a dit vrai ou faux, en partie ou totalement

Eh bien, sans surprise, les trois candidats ont beaucoup (mais alors, beaucoup hein) raconté de choses ambigües, ont forcé le trait ou menti. Un candidat a même excellé dans les contre-vérités et l’imprécision (en répondant “en dehors de la question”).

J’ai aussi réussi, tant bien que mal, à analyser la “teneur des propos”, et deux candidats sont perçus comme “agressifs”, un est souvent sur la défensive, et un autre est souvent “dans le flou”.

Alors, encore une fois, tout sort de ce que l’IA “semble comprendre”. Et c’est pour cela que je ne vous montre pas ces résultats. Ils pourraient être mal interprétés, et utilisés à mauvais escient. Mais sachez que, franchement, j’ai trouvé l’analyse assez proche de mon ressenti.

> Pour information, elle m’a même démontré que j’avais tort sur un sujet. J’ai vérifié son propos, j’ai revu le passage du débat en question et cherché sur le net… et bon sang ! Elle avait raison.

Je me dis que, avec un peu de travail (de temps, d’argent…) ce genre de travaux pourraient aider les analystes, et à terme aider des gens à faire leur choix. Mais pour cela, il faudrait fiabiliser et rendre plus factuel les réponses. Et malheureusement, c’est à ce jour une tâche trop lourde pour le temps que j’ai à disposition.

## Conclusion

Je n’ai pas passé beaucoup de temps à coder tout ça. J’ai dû passer, en gros, 2 heures de mon temps, le soir, au lieu de regarder un film. Donc, **évidemment que le résultat n’est pas parfait, loin de là**. Cependant, cela montre à quel point nous avons sous la main des outils d’une puissance incroyable.

C’est avant tout un test, un amusement. Impossible à ce jour, avec le matériel que j’ai à disposition et surtout le manque de temps que je peux consacrer à ces travaux, de faire un travail plus abouti.

J’ai déjà travaillé sur d’autres implémentations RAG avant cela, et ce fut souvent plus précis, plus pertinent. Pour cet exemple, le résultat est moins fiable à cause de nombre de transformations à effectuer sur les données qui conduisent forcément à un paquet d’erreurs. Il faudrait relire, corriger, adapter…

Mais une chose est certaine : la transcription automatique avec `pyannote` et `whisper` combinée, afin de détecter l’interlocuteur et ses propos fonctionne plutôt bien. `whisper-openai` est clairement un bel outil qui fonctionne vite et bien. `llama-index` est un outil qui permet de faire de la recherche sémantique plus simplement, et simplifie le requêtage des modèles LLM.

ChromaDB et `llama-index` se marient bien. D’ailleurs, ayant aussi testé le RAG avec des PDF, en utilisant les plugins intégrés à `llama-index`, je vous assure que cela donne de très bons résultats.

Le RAG, c’est clairement l’avenir de l’IA dans beaucoup de domaines.

Tout ce que je vous ai montré utilise des outils opensource. Merci encore à la communauté.