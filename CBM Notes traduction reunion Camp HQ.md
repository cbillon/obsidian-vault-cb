
# Camp + Marketplace : une plateforme, deux interfaces

*Suite à la réunion du siège de Moodle du 29 juillet. Document de travail privé ; diffusion interdite avant une annonce conjointe.*

Chaque section le précise deux fois : **En termes simples** pour tous, *Pour le lecteur technique* en italique ci-dessous.

## La proposition en un paragraphe

**En termes simples :** Camp est le catalogue public et le système de contrôle de sécurité des plugins Moodle. Marketplace est une boutique. Une boutique n’a pas besoin de son propre catalogue ; elle a besoin de rayons, de prix et de contrats de support. Nous proposons que Marketplace fonctionne sur la couche d’enregistrement de Camp : Camp conserve les noms, la vérification, les alertes de sécurité et les archives permanentes ; Marketplace gère tout ce qui est commercial. Ainsi, le travail de l’un n’est pas dupliqué et les administrateurs de site bénéficient d’une expérience cohérente au lieu de deux expériences concurrentes.

*Pour le lecteur technique : il s’agit de la mise en œuvre concrète des RFC §3 (objectifs non atteints) et §6.3 : le modèle de boutique sur registre de Packagist / Private Packagist.
Un dépôt est simplement une arborescence de fichiers statique (packages.json,
artefacts, flux d’avis). La Marketplace publie ce format,
les sites activent les deux dépôts, et le client (ou Composer nativement)
les fusionne avec une confiance et une liaison de source spécifiques à chaque dépôt.*

## 1. Un seul espace de noms, respecté conjointement

**En clair :** chaque plugin possède un nom technique unique, et un site Moodle ne peut pas installer deux plugins portant le même nom, quel que soit leur lieu d’achat. Il est nécessaire de gérer cet espace de noms.

L’ancien répertoire s’en chargeait, mais il ne pouvait gérer que les noms qui lui étaient
soumis. Camp a adopté une vision plus globale : il surveille l’ensemble de l’écosystème public, soit plus de 6 000 plugins, dont des milliers n’ont jamais demandé d’inscription. Ainsi, un conflit de noms est détecté dès son apparition publique, et non lors de son enregistrement.

La politique est écrite et publique, et elle a déjà fait ses preuves à plusieurs reprises au cours de ses deux premières semaines, chaque cas étant documenté publiquement. Nous proposons que les deux organisations respectent un espace de noms partagé et une procédure de résolution des conflits unique.

*Pour les lecteurs techniques : voir NAMESPACE.md dans la documentation de Camp. La publication confère un nom, sa découverte ne le réserve pas, une revendication n’est pas un transfert, et les conflits sont résolus publiquement. Les conflits entre dépôts sont résolus
par priorité d’administrateur avec affichage des modifications (RFC § 6.3), jamais
en silence. Des analyses mensuelles classent chaque candidat public par rapport aux noms existants ; des sondages dans l’historique partagé permettent de distinguer les forks,
les copies et les véritables conflits. Environ 6 100 noms sont suivis par rapport aux ~2 900 noms del'ancien répertoire.*

## 2. La Marketplace comme dépôt

**En termes simples :** le format technique de camp est volontairement simple, comme
une prise électrique standard. N'importe qui peut y distribuer des plugins, y compris un magasin vendant des produits à accès contrôlé. La Marketplace ne s'intègre pas *à* camp à proprement parler, mais utilise le même format de prise.

Chaque site Moodle peut ainsi utiliser les deux simultanément, l'administrateur du site
gardant toujours le contrôle sur la source utilisée.

*Pour le lecteur technique : un plugin payant protégé par un jeton d'autorisation est un dépôt protégé par un jeton d'accès, exactement comme un dépôt APT/RPM d'un fournisseur. Identifiants par dépôt et racines de confiance épinglées ; la règle de liaison des sources empêche toute confusion liée aux dépendances entre dépôts ;

le masquage est toujours visible. Les sites gérés par Composer (Moodle 5.2+) bénéficient
de cette sémantique nativement.*

## 3. Un socle de confiance partagé

**En clair :** Camp prouve automatiquement que chaque package de plugin publié a été compilé, octet par octet, à partir du code source public
dont il prétend provenir. Cette preuve est tout aussi précieuse sur une plateforme de vente.

La Marketplace pourrait l'afficher ou l'exiger pour ses propres annonces. La publication en elle-même ne comporte aucun risque de vol : un auteur publie en ajoutant une étiquette de version. Ce qui reste distinct, c'est l'*opinion* de chaque organisation sur un plugin (avis, recommandations,curation) : ces avis restent clairement identifiés par leur auteur.

*Pour le lecteur technique : reconstruction déterministe à l'étiquette,comparaison des hachages,registre des versions en ajout uniquement, analyse antivirus (RFC §4.2), avec signature TUF et journalisation de transparence prévus (§4.3). La publication est une publication de confiance OIDC : l’intégration continue (CI) présente une déclaration d’identité signée,

le service de publication la vérifie par rapport au dépôt source de l’annonce déclarée
grâce à l’identifiant immuable du dépôt, et c’est le service, et non le flux de travail de l’auteur, qui ouvre la demande de tirage (PR) de la version. Les niveaux sont des assertions par dépôt et ne se chevauchent jamais (§ 6.3).*

## 4. Un canal unique d’alerte de sécurité

**En clair :** si un plugin présente une faille de sécurité, tous les sites l’utilisant doivent être avertis, quelle que soit sa provenance. Les alertes sont la seule chose qui ne devrait jamais faire l’objet de concurrence. Le système d’alerte de Camp est opérationnel dès aujourd’hui ; nous proposons un tri coordonné et une règle selon laquelle les deux systèmes alertent sur tout.

*Pour le lecteur technique : les flux d’alertes sont unifiés côté client ; le flux de chaque
dépôt activé est comparé à tous les plugins installés.

La révocation reste Limité au dépôt de publication. CAMP-2026-0001 à 0003 : publiés avec divulgation coordonnée dès la première semaine ;
CAMP-2026-0004 à 0006 : retraits non liés à la sécurité d’enregistrements de publication défectueux le registre d’ajout uniquement se corrige automatiquement publiquement. Les vulnérabilités principales restent du ressort exclusif du siège ; camp gère uniquement les plugins tiers (RFC §3).*

## 5. L’archive permanente

**En clair :** camp conserve chaque version de plugin publiée dans une archive qui ne peut être ni modifiée ni supprimée discrètement, pendant au moins sept ans. Les stocks des boutiques sont renouvelés ; les écosystèmes ont besoin de mémoire.

La Marketplace n’a jamais à gérer cela, et « retiré de la vente » ne signifie jamais « effacé de l’historique ».

*Pour les lecteurs techniques : verrouillage d’objet B2 en mode conformité, conservation de 7 ans par objet, dépôt et audit à chaque publication, servis via un domaine proxy. Rien n'est jamais supprimé ; le retrait est une révocation consultative qui rend une version désinstallable tout en la conservant dans les archives.*

## 6. Installation dans Moodle

**En clair :** l'état final que les utilisateurs méritent : un administrateur ouvre son site Moodle, effectue une recherche et installe des plugins, les plugins de la communauté et de la Marketplace apparaissant côte à côte, clairement identifiés. C'est l'élément que seul le siège social peut pleinement exploiter, et celui qui rend les deux surfaces meilleures que chacune prise séparément.

*Pour le lecteur technique : la surface de mise à jour/installation du noyau Moodle
utilise le format de dépôt, via le modèle de plugin client ou nativement. La prise en charge de plusieurs dépôts est déjà conçue (DESIGN.mdD21) : fusion en PHP, liaison à l'installation, politique de niveau minimum par dépôt.*

## Ce que nous demandons au siège social

1. Les données historiques du répertoire des plugins (historique des versions, avis)
sont archivées, afin que la mémoire de l'écosystème survit à la transition.

2. Coordination du calendrier, avec un lien et un petit mot d'encouragement, lors de la publication de l'annonce sur moodle.org.

3. Alignement sur moodle-plugin-ci comme chaîne d'outils partagée pour la vérification des plugins.

4. Intégration de l'API de mise à jour du noyau, selon un calendrier réaliste.

## Nos principes

- Le camp reste géré par la communauté et indépendant du commerce.
La distribution est assurée ; la gouvernance et la neutralité ne le sont pas.
- Aucune exclusivité. Les auteurs peuvent s'inscrire librement sur les deux plateformes.
Une fiche déplacée conserve son historique, ses archives et son nom.
- Pas de substitution silencieuse. En cas de désaccord entre les deux sources,
l'administrateur du site tranche et le désaccord est visible.

---

# Annexe : deux semaines de tests

Ce registre n'est pas une proposition. Public depuis le 14 juillet, et durant cette période :

- **Plus de 6 170 plugins indexés**, avec recherche, niveaux de confiance, indicateurs de santé et accessibilité aux WCAG 2.2 AA.

- **71 plugins revendiqués par leurs auteurs** (28 mainteneurs différents),

dont les mainteneurs de certains des plugins les plus installés de l'écosystème ;

27 plugins avec des versions entièrement vérifiées au niveau du code source, soit 38 versions vérifiées au total ;
les versions d'auteurs externes sont distribuées selon le flux de travail standard (une étiquette Git suffit).
- **Un pipeline de sécurité fonctionnel** : six avis de sécurité publiés. Trois avis de sécurité ont été publiés en coordination avec la version corrigée, avec des avertissements par version sur le site et dans les métadonnées utilisées par les clients ;
trois retraits non liés à la sécurité ont corrigé des enregistrements de publication défectueux,
ce à quoi ressemble un registre d'ajout uniquement.

- **La politique d'espace de noms a été appliquée avec succès à deux reprises.** Un conflit de noms entre deux plugins non liés est en cours de traitement, conformément à une politique de notification écrite et de délai de préavis. Par ailleurs, un mainteneur a volontairement attribué le nom d'un composant au plugin de production qui le méritait ; le transfert a duré une journée et est documenté publiquement.

- **La communauté effectue le nettoyage spontanément.** Dix-huit listes inactives ont été supprimées à la demande de leurs propres mainteneurs, sans aucune question, par certains des développeurs les plus respectés de l'écosystème, dont plusieurs ont revendiqué leurs plugins actifs le jour même.

- **La publication sans jeton est disponible.** Depuis le 28 juillet, un auteur publie
en poussant une étiquette Git : aucun jeton, fork ou secret n'est utilisé dans le
flux, il n'y a donc rien à divulguer, à faire tourner ou à faire expirer. La première
version publiée de cette manière a produit trois versions indépendantes du
même code : le flux de travail de l'auteur, la vérification du registre et
une reconstruction locale, toutes validées par le hachage. GitHub et gitlab.com sont
pris en charge, avec une solution de repli pour les autres. 
- **Un service indépendant d'évaluation de sécurité** (MDL Shield) a intégré ses évaluations publiées aux pages des plugins dès la première semaine, selon un modèle de confidentialité approuvé par les deux parties.

- **Une infrastructure gérable par une petite équipe** : entièrement statique, répliquable, publication incrémentale avec un coût proportionnel aux nouvelles versions, une surveillance externe du pipeline de publication et détection automatisée, actualisation des indicateurs et surveillance des versions en amont.

En résumé : la couche de registre, complexe et peu attrayante, est construite, opérationnelle et bénéficie déjà de la confiance de la communauté. Développez le magasin de plugins.