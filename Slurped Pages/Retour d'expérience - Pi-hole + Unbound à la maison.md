---
link: https://www.linkedin.com/pulse/retour-dexp%C3%A9rience-pi-hole-unbound-%C3%A0-la-maison-cyril-beaufrere-mvone/?trackingId=J63huFhSQ6i9b%2F4XE1RUlQ%3D%3D
byline: Cyril Beaufrere
site: "@LinkedInEditors"
date: 2026-01-05T07:34
excerpt: "Il y a des chantiers que l'on repousse parce qu'on se contente de
  quelque chose qui \"marche à peu près\", jusqu'au moment où l'on décide que
  \"à peu près\" n'est plus un standard acceptable. J'ai pris ce moment pour mon
  DNS domestique : non pas pour courir derrière une promesse de vitesse magique,
  mais"
twitter: https://twitter.com/@LinkedInEditors
slurped: 2026-02-20T10:46
title: "Retour d'expérience : Pi-hole + Unbound à la maison"
---

Il y a des chantiers que l'on repousse parce qu'on se contente de quelque chose qui "marche à peu près", jusqu'au moment où l'on décide que "à peu près" n'est plus un standard acceptable. J'ai pris ce moment pour mon DNS domestique : non pas pour courir derrière une promesse de vitesse magique, mais pour comprendre ce qui se passe réellement quand une machine de la maison ou une VM de mon labo demande "où est ce domaine ?", cesser de subir les choix par défaut de la Livebox, et poser une solution sympa sur le papier avec Pi-hole. Ce petit side project, sans prétention, sert autant les postes clients du foyer que certaines VMs et LXC de mon labo, et depuis qu'il tourne, le web est plus calme et mes diagnostics plus rapides.

### C'est quoi Pi-hole, quelles alternatives, et pourquoi je l'ai choisi

Pi-hole à la base, c'est un DNS "sinkhole" : il intercepte les requêtes des clients, compare les noms demandés à des listes de blocage (et à d'éventuelles regex), puis répond localement (NXDOMAIN/0.0.0.0) pour tout ce qui est indésirable, avant même que le navigateur ou l'application n'émette la moindre connexion. Il peut aussi gérer des groupes de clients avec des politiques différentes, et expose une interface d'admin très lisible (Query Log en temps réel, stats, historiques). Avec Unbound en amont, il ne forwarde plus vers un DNS public : il filtre et résout en local, ce qui ferme proprement la boucle.

Côté alternatives, on peut citer [AdGuard Home](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fadguard-dns%2Eio%2Fkb%2Ffr%2Fadguard-home%2Fgetting-started%2F&urlhash=c6HA&trk=article-ssr-frontend-pulse_little-text-block) (très proche dans l'esprit, interface moderne, quelques fonctions by design plus modernes), [pfBlockerNG](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fwww%2Epandawan%2Efr%2Ffiltrage-dns-et-ip-avec-pfblockerng%2F&urlhash=brKe&trk=article-ssr-frontend-pulse_little-text-block) sur pfSense (puissant si l'on a déjà pfSense comme routeur), et sans doute plein d'autres que je ne connais pas. On peut aussi bricoler avec BIND/Unbound + RPZ ou dnscrypt-proxy, mais on perd souvent la simplicité d'exploitation et la visibilité clé en main.

Pourquoi Pi-hole chez moi ? Parce que je l'avais déjà testé sur Raspberry Pi il y a quelques années et que j'en avais gardé l'idée d'un outil simple et robuste : l'UI fait exactement ce dont j'ai besoin (voir, comprendre, décider), l'intégration avec Unbound est naturelle, la communauté est active, et la friction d'usage est faible.

## Pourquoi j'ai fait ça

Je voulais d'abord reprendre la main sur la résolution elle-même : savoir qui interroge qui, à quel moment, et être capable de l'expliquer quand une page refuse de se charger rapidement ou qu'une application décide de faire des caprices. Mettre Pi-hole en amont me permet de couper court aux domaines de pub et de tracking au niveau DNS, pareil pour des domaines connus pour diffuser des malwares, donc avant que le navigateur ou l'app ne lance quoi que ce soit : un appel qui n'existe pas n'ajoute ni attente ni bruit. Ce que je cherche n'est pas une vitesse de pointe flatteuse mais une sensation de régularité : des chargements qui restent constants, sans à-coups liés à des services tiers qui répondent quand ils veulent, et éviter que mes ados tombent sur des domaines "à risques".

J'y ajoute une exigence de confidentialité et d'indépendance : arrêter d'envoyer toutes les questions du foyer à un résolveur public unique, garder un cache local et valider l'intégrité des réponses via DNSSEC. Enfin, il me fallait de l'observabilité. Le journal des requêtes apporte une vérité opérationnelle : en trente secondes, je peux trancher entre "Pi-hole a bloqué quelque chose d'utile", "un fournisseur est en panne" ou "le problème n'est pas DNS".

Le décor technique est simple : un cluster Proxmox 9.1.x qui sait maintenant lancer des images OCI directement dans des LXC, une Livebox qui reste la passerelle et le DHCP, et parce qu'elle ne pousse pas proprement un DNS personnalisé un DNS saisi à la main sur les machines importantes. Cela vaut pour les postes de la maison comme pour mon labo, où mes VMs et LXC profitent des mêmes règles de résolution et du même niveau de visibilité. Je pourrai rajouter un switch après la livebox qui assumerait le rôle de DHCP et pourrait me propulser le DNS automatiquement, mais j'ai la flemme pour le moment, mes switchs de la maison n'assurant que la distribution, mais ça devrait changer dans le courant de l'année.

## Comment je l'ai fait

Au départ, j'ai voulu profiter de la nouveauté de Proxmox 9.1 : lancer une image OCI directement dans un LXC, sans VM ni moteur Docker. Sur le papier, c'est séduisant, on importe l'image officielle de Pi-hole, on déclare deux montages persistants, on fixe le fuseau horaire, un mot de passe, on démarre et on a un service, mais en pratique, le conteneur démarrait puis s'arrêtait aussitôt. En le lançant au premier plan pour lire l'entrypoint, la vérité tombait en clair : sh: command not found, sed: command not found, grep: command not found, jusqu'à env: bash: no such file or directory. Ce n'était pas juste "un paquet qui manque", c'était structurel. L'image Pi-hole est fabriquée pour un runtime Docker complet avec un certain nombre d'hypothèses implicites, et un LXC-OCI minimal ne fournit pas ce socle. Ni bug Pi-hole, ni bug Proxmox, simplement une incompatibilité de modèle. Toutes les images Docker ne sont pas faites pour s'exécuter "en direct" dans un LXC.

Plutôt que de bricoler un hybride fragile, j'ai choisi la voie de la simplicité via un conteneur Debian standard, Pi-hole exécuté dans le runtime pour lequel il est conçu (Docker), et Unbound installé en natif dans le même LXC.

## LXC Debian et Docker Compose pour Pi-hole

J'ai donc créé un LXC Debian (Debian 13 dans mon cas, unprivileged), avec une IP fixe sur le LAN. Les ressources sont modestes (un vCPU et 512Mo de RAM) mais amplement suffisantes pour un résolveur et un filtre DNS. Côté base système, une mise à jour, quelques utilitaires, puis l'installation de Docker selon la procédure officielle et du plugin Compose pour garder un déploiement lisible, versionné et facile à rejouer.

Dans le LXC, j'ai préparé l'arborescence persistante sous /opt/pihole, avec etc-pihole et etc-dnsmasq.d comme volumes. Le coeur du déploiement tient dans un compose.yaml très simple, et je lance l'image officielle pihole/pihole:latest en network_mode: host. Ce détail est important, car pour un serveur DNS, c'est le mode le plus simple et le plus fiable. Il évite les mappages de ports parfois surprenants et les subtilités de NAT qui compliquent le diagnostic.

Vous noterez l'agent Beszel dont je ne peux plus me passer.

Un docker compose up -d, puis l'interface d'administration est disponible sur l'ip du LXC. À ce stade, Pi-hole filtre déjà… mais il dépend encore d'un DNS en amont, et c'est précisément là qu'entre en scène Unbound.

## Unbound en natif

Le résolveur récursif local Unbound n'est pas un simple relais, il ne forwarde pas vers Google, Cloudflare ou Quad9, il résout vraiment, en local, chaque nom demandé. À la question "où est rudeops.com ?", Unbound remonte la chaîne DNS : il interroge un serveur racine (grâce au fichier root.hints) pour savoir qui gère le TLD concerné (.fr, .com, etc.), interroge ensuite le registre du TLD pour savoir qui fait autorité sur [rudeops.com](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fwww%2Erudeops%2Ecom&urlhash=XtMt&trk=article-ssr-frontend-pulse_little-text-block), puis contacte le serveur faisant autorité pour obtenir la réponse finale. En chemin, il valide DNSSEC lorsque la zone est signée, et en cas d'une une signature incohérente, la réponse est tout simplement rejetée. Il applique la QNAME minimization que j'ai découvert à cette occasion ([RFC 7816](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fdatatracker%2Eietf%2Eorg%2Fdoc%2Fhtml%2Frfc7816&urlhash=lftr&trk=article-ssr-frontend-pulse_little-text-block)) : à chaque étape, il ne révèle que la portion nécessaire du nom .com, puis rudeops.com, puis www, ce qui limite la quantité d'information partagée aux intermédiaires, et il met en cache les résultats, ce qui signifie que la première requête "trace la voie", et les suivantes répondent vite tant que le TTL n'a pas expiré.

Ce rôle de "résolveur maison" mérite d'être souligné parce qu'il change l'ADN du DNS à domicile : la chaîne de confiance est contrôlée sur place (l'ancre DNSSEC est mise à jour automatiquement, les signatures sont validées chez moi), la confidentialité progresse (je ne révèle pas systématiquement mes noms complets à chaque niveau, et je n'externalise pas mes habitudes de consultation), la résilience augmente (si un résolveur public a une panne ou si le FAI se montre capricieux, je ne dépends plus d'un point unique), et la qualité de service devient prévisible grâce au cache (Unbound précharge même certaines entrées populaires avec prefetch, de sorte qu'un nom consulté souvent reste chaud sans pénaliser le premier utilisateur qui passera). Détail pratique : Unbound met aussi en cache les échecs (NXDOMAIN), ce qui évite de reposer cent fois la même question impossible, et, pour mon labo, il accepte des overrides locaux (avec local-zone et local-data) pour donner des réponses rapides à des noms purement internes ou de test, sans jamais sortir sur Internet.

L'installation est directe : paquet unbound, root.hints à jour dans /var/lib/unbound, et une configuration dédiée pour écouter exclusivement sur la boucle locale, port 5335.

Unbound n'est pas exposé au LAN, il n'y a que Pi-Hole qui lui parle.

Je valide la configuration (unbound-checkconf), je lance le service, et je teste directement : dig @127.0.0.1 -p 5335 rudeops.com +dnssec doit répondre proprement. Une fois Unbound en ordre de marche, je branche Pi-hole sur lui.

## Chaîner Pi-hole vers Unbound, puis configurer les clients

Dans l'interface Pi-hole, je désactive les upstreams publics, j'ajoute un unique DNS Custom en IPv4 : 127.0.0.1#5335. Je désactive DNSSEC côté Pi-hole (puisque la validation est déjà faite par Unbound), je redémarre le service, et je vérifie avec un dig depuis le LXC.

La chaîne Pi-hole => Unbound => Internet fonctionne

Reste l'étape clients. Ma Livebox ne pousse pas proprement un DNS personnalisé, je saisis donc à la main l'adresse du LXC comme DNS primaire sur les postes de la maison, mais aussi sur mes VMs et LXC de labo. J'insiste sur un point pratique : je laisse le DNS secondaire vide (ou identique au primaire) pour éviter les bascules silencieuses vers la Livebox qui feraient perdre le bénéfice du filtrage. Sous Windows, j'ai rencontré un comportement étrange : même avec un DNS IPv4 statique, le système continuait d'utiliser le DNS IPv6 annoncé par la box. La solution a été de désactiver l'IPv6 sur l'interface concernée et immédiatement, nslookup a pointé vers pi-hole avec l'IP du LXC, et tout s'est mis à circuler au bon endroit.

## Résultats et bénéfices

Au quotidien, le gain le plus net est la régularité. Les pages n'attendent plus des domaines annexes en panne ou trop lents, l'affichage est plus constant, ce qui se ressent davantage qu'un hypothétique gain de millisecondes. Les interfaces sont moins encombrées d'éléments secondaires. Lire un article, suivre une recette, consulter une documentation, tout est plus fluide. À la maison, les ados voient moins de pré-rolls et de distractions "gratuites" sur les plateformes vidéo. Dans mon labo, mes VMs et LXC arrêtent de bavarder avec des télémétries et analytics dont je n'ai pas l'usage, les tests reposent sur une résolution prévisible, ce qui facilite grandement les comparaisons.

Côté diagnostic, le Query Log de Pi-hole est devenu un réflexe. Quand "ça tourne en rond", je vois en quelques secondes si Pi-hole a bloqué un domaine nécessaire, si un fournisseur tiers est à la peine, ou si le souci n'est pas DNS. Une anecdote l'illustre bien : une application métier utilisée par ma femme s'est mise à mal réagir (formulaires qui perdent l'état, chargements en boucle, messages un peu flous). Le journal a révélé qu'un domaine d'analytics/CDN, paradoxalement indispensable au fonctionnement de l'app, était bloqué. J'ai ajouté ce domaine en liste blanche, et même si elle était furieuse le temps de l'incident, cinq minutes après, tout marchait. Depuis, je limite l'exception aux seules machines qui en ont besoin, histoire de ne pas élargir inutilement la surface d'exemption.

Quant aux chiffres, je ne les sacralise pas, mais ils donnent un ordre d'idée : selon les jours, je vois entre dix et quinze pourcent de requêtes bloquées, davantage côté objets connectés idiots. Le plus important n'est pas le pourcentage, mais la friction en moins et la prévisibilité qui s'installent. La latence "perçue" n'est pas tant une histoire de vitesse de pointe que de constance, effet cumulé du cache d'Unbound et de la disparition de multiples appels à des tiers capricieux.

Le tableau admin est un peu vieillot mais fait le taffe

## Les logs sont vos meilleurs amis

Quand quelque chose "tourne en rond", que ce soit une page un peu têtue, une appli qui refuse de s'authentifier ou une vidéo qui s'arrête net, la différence entre l'impression et le diagnostic tient à une poignée de traces bien lues. Pi-hole et Unbound fournissent exactement ce qu'il faut pour passer du soupçon au fait, à condition d'adopter une petite hygiène d'observation et deux ou trois réflexes de terrain. Ce chapitre est fait pour être lu comme on suit une piste : du symptôme à la cause, en traversant les bons journaux au bon moment.

Commencer par le visible, c'est ouvrir le Query Log de Pi-hole et regarder, en temps réel, la danse des noms. On y voit ce que demande la machine, ce que Pi-hole bloque, ce qu'il laisse passer, et surtout le "pourquoi" de la décision (denylist, regex, groupe, etc.). On lance l'action qui coince (rechargement de la page, clic pour rejouer la séquence) et l'on revient sur le journal : on lit les lignes comme on lirait un storyboard. Si un domaine est marqué "blocked", la partie est presque gagnée : on a le nom exact, l'heure, la règle qui a tiré, et il ne reste qu'à décider s'il faut corriger la règle (par exemple une regex trop large) ou accorder une exception ciblée au groupe concerné.

Lorsque Pi-hole "laisse passer" mais que le symptôme persiste, c'est souvent le moment d'écouter Unbound. Lui parle peu, mais il parle juste. Son journal raconte la résolution pas à pas : l'interrogation des racines, la descente vers le TLD, la réponse du serveur faisant autorité, et, s'il y a un problème, la raison précise, comme la signature DNSSEC qui ne colle pas, délégation incomplète, délai côté autorité. Augmenter temporairement la verbosité, reproduire le problème, puis revenir à un niveau discret est une approche simple et efficace :

Il est parfois plus rapide de poser des questions directes au DNS lui-même, sans passer par l'interface. Depuis le LXC ou votre terminal favori on interroge Pi-hole et Unbound séparément : l'un sur le port 53, l'autre sur 127.0.0.1:5335. On compare les réponses, on active DNSSEC côté client pour demander les signatures, on suit la trace recursive si besoin ; quelques commandes suffisent pour lever l'ambiguïté.

Vérifier un échec DNSSEC

Suivre la chaîne de résolution

Quand un doute subsiste sur ce qui sort vraiment du conteneur, regarder le trafic met fin aux suppositions. On écoute la boucle locale pour les échanges Pi-hole ↔ Unbound, et l'interface LAN pour les dialogues d'Unbound vers l'extérieur. Quelques secondes suffisent à confirmer qu'une requête est bien passée par la chaîne locale ou, au contraire, qu'une application a tenté un contournement.

Ecoute des échanges internes (Pihole => Unbound sur la loopback:5335)

Ecoute de ce qui part vers Internet en DNS classique (UDP/TCP 53)

Sur la durée, Pi-hole garde une mémoire utile. Son historique long terme (SQLite) permet de revenir en arrière, de répondre à la question "depuis quand ?", et d'identifier des motifs récurrents. C'est précieux pour un service qui casse "de temps en temps" à heures fixes, ou pour montrer noir sur blanc qu'une liste trop agressive a commencé à tirer à une date précise. On veille toutefois à un détail qui compte : le niveau de confidentialité. Pi-hole permet d'anonymiser partiellement les requêtes stockées ; on choisit un réglage compatible avec son besoin de diagnostic et sa sensibilité aux données.

Et si vous voulez un diag guidé, utilisez pihole -d

Unbound, de son côté, expose des statistiques et même un bout de son cache via unbound-control. Savoir si un nom est "chaud" ou "froid" explique des écarts de ressenti ; savoir combien de validations DNSSEC ont échoué sur une période courte permet de distinguer une vraie panne en amont d'un simple loupé transitoire.

Il reste enfin le cas particulier mais peu fréquent où les logs sont silencieux alors que le symptôme est bien réel. C'est souvent le signe d'un contournement par DoH côté client : aucune trace côté Pi-hole, rien dans Unbound, et pourtant l'application "résout". La preuve négative devient ici un signal : si les journaux n'ont rien vu, c'est que la requête ne leur est jamais parvenue. On désactive DoH sur le client suspect, on rejoue, et les logs se remettent à parler. À l'inverse, si tout transite bien mais qu'Unbound se tait soudain sur un domaine signé, on pense DNSSEC avant de penser "liste de blocage" : une autorité qui a publié une signature expirée ou une chaîne de confiance rompue explique à elle seule des erreurs apparemment aléatoires.

La routine qui marche tient en quelques pas : reproduire le problème en regardant en parallèle le Query Log et journalctl -u unbound -f, noter l'heure exacte et le nom précis, tester avec dig Pi-hole puis Unbound, écouter rapidement le port 5335 en local, puis arrêter là où la chaîne se coupe. En procédant toujours dans cet ordre (visible (Pi-hole) puis récursif (Unbound), puis réseau) on gagne du temps et on évite les faux positifs. Les logs ne sont pas qu'un outil de forensique, ce sont des garde-fous qui rendent la solution durables, parce qu'ils transforment chaque incident en décision éclairée.

## Un mot sur les métriques

J'ai branché Pi-hole sur mon Prometheus et fait deux-trois panels Grafana avec [l’exporter pihole-exporter d'eko](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fgithub%2Ecom%2Feko%2Fpihole-exporter&urlhash=un34&trk=article-ssr-frontend-pulse_little-text-block). Ça se fait le plus simplement du monde, même si l’interface d’admin de Pi-hole est déjà très bien mais j’ai l’habitude d’agréger mes signaux dans Prom. Concrètement, je lance l'exporter dans le LXC et je le scrape.

## Ce que j'aurais aimé savoir avant (et qui m'aurait évité quelques détours)

Si je devais donner des conseils à mon "moi d'avant", je commencerais par lui dire que multiplier les listes de blocage ne sert à rien : j'ai débuté avec un empilement rassurant, presque gourmand, et j'ai fini par n'en garder que trois, régulièrement tenues, qui font l'essentiel du travail sans casser des usages légitimes, c'est l'exemple type de la fausse bonne idée qui flatte l'instinct mais complique la vie.

Je lui dirais aussi que la fonction OCI en LXC n'est pas un Docker déguisé : certaines images (dont l'officielle Pi-hole pour l'instant) s'attendent à un environnement complet et échoueront dans un LXC-OCI minimal ; mieux vaut un CT Debian simple et un Pi-hole lancé dans son runtime prévu (Docker), plutôt que des heures de rustines.

Je lui glisserais un rappel sur l'IPv6 sous Windows : on croit avoir fixé un DNS IPv4 propre, et l'OS privilégie tranquillement le DNS IPv6 poussé par la Livebox ; on perd du temps à se demander "pourquoi ça ne passe pas par Pi-hole", alors qu'il suffit de désactiver IPv6 sur l'interface concernée pour vérifier le comportement et, au besoin, garder cette configuration si elle simplifie la vie. RAS sur les clients Linux bien entendu...

Enfin, j'insisterais encore sur Unbound : ce n'est pas une case "exotique" qu'on coche pour suivre une tendance, c'est la brique qui transforme Pi-hole en solution autonome ; Unbound résout vraiment (racine → TLD → autorité), valide (DNSSEC), limite ce qu'il révèle (QNAME minimization), stabilise (cache et prefetch), accélère même les échecs (NXDOMAIN en cache), et accepte des raccourcis locaux pour le labo (réponses internes, noms de test) ; c'est lui qui rend l'ensemble compréhensible et reproductible, parce que l'on voit et maîtrise chaque étape de la chaîne.

---

En définitive, ce montage Pi-hole + Unbound apporte exactement ce que j'en attendais : un web plus discret, des chargements réguliers, des VMs et LXC de labo qui cessent de bavarder pour rien, et surtout la capacité d'expliquer rapidement pourquoi un service se comporte mal. Ce n'est pas spectaculaire, mais c'est propre et facilement reproductible et c'est précisément pour ça que ça tient dans le temps.

---

## Ressources utiles

Pour aller plus loin ou simplement valider une intuition, [la documentation officielle de Pi-hole](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fdocs%2Epi-hole%2Enet%2F&urlhash=Jf9e&trk=article-ssr-frontend-pulse_little-text-block) est très bien faite et à jour, je m'y suis forcément beaucoup appuyé, en particulier la page dédiée à Unbound (résolveur récursif local, DNSSEC, QNAME minimization, etc...).

Côté pas-à-pas, j'ai trouvé clairs et pratiques :

- le tutoriel [Pi-hole + Unbound de Crosstalk Solutions](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fwww%2Ecrosstalksolutions%2Ecom%2Fthe-worlds-greatest-pi-hole-and-unbound-tutorial-2023%2F&urlhash=RW0u&trk=article-ssr-frontend-pulse_little-text-block) (bonne vue d’ensemble et écueils courants).
- quelques fils [Pi-hole Discourse](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fdiscourse%2Epi-hole%2Enet%2Ft%2Frecursive-dns-server-unbound-or-dns-over-https%2F11890&urlhash=y9J3&trk=article-ssr-frontend-pulse_little-text-block) qui éclairent des points précis (rôle d'Unbound vs DoH/forwarding, vérifications DNSSEC/QNAME).
- Un super tuto (comme d'habitude) [chez IT Connect](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fwww%2Eit-connect%2Efr%2Fpi-hole-un-bloqueur-de-pubs-pour-tout-votre-reseau%2F&urlhash=dxFl&trk=article-ssr-frontend-pulse_little-text-block).

Enfin, pour une approche "pas à pas" sur d'autres plateformes (Raspberry Pi, Debian/Ubuntu), ces guides restent pertinents pour recouper installation et bonnes pratiques : [tutoriel Raspberry Pi](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fwww%2Eraspberrypi%2Ecom%2Ftutorials%2Frunning-pi-hole-on-a-raspberry-pi%2F&urlhash=RzRU&trk=article-ssr-frontend-pulse_little-text-block) officiel et un guide [Pi-hole + Unbound](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fsecure-bits%2Eorg%2Fen%2Fposts%2Fprivacy%2Fpihole%2Fpihole-unbound%2F&urlhash=Iidl&trk=article-ssr-frontend-pulse_little-text-block) orienté Debian.