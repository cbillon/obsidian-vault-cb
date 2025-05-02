---
link: https://medium.com/@tpierrain/le-ddd-est-il-de-droite-2b8c695393aa
byline: Thomas Pierrain. (υѕe caѕe drιven)
site: Medium
date: 2022-10-14T09:36
excerpt: Derrière cette affirmation provocante (idéale pour pitcher une session d’unconf et avoir du monde), Simon (Chaveneau) a voulu se poser la question de l’impact du DDD sur notre organisation…
twitter: https://twitter.com/@tpierrain
slurped: 2024-08-26T18:30
title: Le DDD est-il de droite ? - Thomas Pierrain. (υѕe caѕe drιven) - Medium
tags:
  - ddd
---

## **“Le DDD c’est de droite !”**

Derrière cette affirmation provocante (idéale pour pitcher une session d’unconf et avoir du monde), [Simon (Chaveneau)](https://www.google.fr/url?sa=t&rct=j&q=&esrc=s&cd=&ved=2ahUKEwict-zdmN_6AhXIw4UKHYUDAaUQ6F56BAgOEAE&url=https%3A%2F%2Ftwitter.com%2Faafsicha%3Fref_src%3Dtwsrc%255Egoogle%257Ctwcamp%255Eserp%257Ctwgr%255Eauthor&usg=AOvVaw3n6Da-QBMCaQJ0qC4_-S3u) a voulu se poser la question de l’impact du DDD sur notre organisation (Produit-Tech) pour produire du logiciel. Cela s’est passé lors de notre première non conférence à Agicap, organisée pour permettre au Produit et à la Tech de se retrouver pour discuter, apprendre, partager, rire, mais surtout prendre de la hauteur par rapport à notre quotidien.

Notre 1ere unconference à Agicap, le 13 Oct 2022

Lors d’une unconference, ce sont les gens présents qui forgent le programme durant une session initiale de place du marché (“Market Place”). Lors de cette première plénière de la journée, chacune et chacun peut se saisir d’un post-it, d’un feutre, et de pitcher ensuite devant tout le monde ses sessions, avant de les positionner sur le programme du jour (pour plus d’info sur cette unconference, il y a [**ce fil twitter**](https://twitter.com/massbath/status/1580477223420964864?s=21&t=VTNOWLvhXZvV8XUK1PtndQ)**).**

Mais revenons au sujet et à la session proposée par Simon.

## Le règne de la propriété privée ?

En substance pour Simon : si des pans entiers de notre SI appartiennent à des équipes -chacune responsable d’un ou plusieurs Bounded Context (BC)- ne sommes nous pas en face d’une version logicielle de la propriété privé (avec barbelés autours des BCs, etc) ? D’où le “DDD, c’est de droite !” 😜

Et avec comme question subsidiaire : “ne serions nous pas plus efficace avec une organisation à base de feature teams ? (avec tout le monde qui peut toucher à tous les BCs se trouvant sur la route de ses use cases)”

Et bien je dois avouer que cette session s’est révélée beaucoup plus fructueuse que je ne l’avais espéré initialement car s’en est ensuivit une discussion vraiment passionnante à propos de notre mode d’organisation Produit/Tech.

Mais plutôt que de détailler le cheminement qu’à pris cette session où nous étions très nombreux (sûrement intéressant à décrire dans le cadre d’un autre billet), je vais directement décrire ce que j’en ai retenu et pense sur le sujet.

## Quelques observations

- **Un logiciel efficace est un logiciel aligné par rapport aux enjeux métier**. Par aligné, on entend un logiciel qui emprunte les bons termes du domaine, qui articule correctement les concepts clés du métier et convoque le moins possible la complexité accidentelle apportée par les enjeux techniques. [**C’est un des principaux enjeux du Domain Driven Design (DDD), comme expliqué ici**](https://medium.com/@tpierrain/domain-driven-design-explained-in-3-minutes-d45ed92fcc1c).
- **La loi de Conway n’est pas une option qu’on pourrait décider de ne pas prendre**. Elle agit un peu à l’instar de certaines lois physiques comme l’attraction de la pesanteur. On peut lutter contre, mais elle est toujours valable sur terre. La loi de Conway est cette loi de 1967 qui formule que _“les organisations qui conçoivent des systèmes […] tendent inévitablement à produire des designs qui sont des copies de la structure de communication de leur organisation”_. Pour le dire autrement, l’architecture d’un logiciel est forcément la résultante des modes de communication des gens impliqués pour le concevoir. Par exemple, un compilateur développé par 3 équipes fonctionnera très vraisemblablement en 3 passes ou aura 3 modules bien distincts.
- **La manœuvre inversée de Conway permet d’arrêter de subir la loi de Conway**. Celle-ci recommande de définir l’organisation (nombre d’équipes) en fonction de l’architecture logicielle que l’on souhaite obtenir.
- **Le DDD n’impose aucune organisation. Il donne des clés pour permettre de survivre**, y compris dans des cas d’organisations dysfonctionnelles (avec des équipes qui se font la guerre ou qui s’ignorent). Il nous aide à décrypter les enjeux de pouvoir et les enjeux sociaux entre équipes. De ce fait, c’est plutôt un outil de libération contre nos déterminismes (de gauche donc ;-)
- En revanche, **le DDD nous donne des outils pour pouvoir concevoir un logiciel efficace et le plus aligné possible par rapport au domaine**. Parmi eux le concept de Bounded Context (BC), qui nous propose de concevoir des modèles pour des usages, et de les encercler et protéger contre les confusions/faux amis/incompréhensions issus des autres contextes.
- Pour plus d’alignement et d’efficacité, **le DDD nous recommande aussi vivement de challenger régulièrement notre vision et notre modélisation** de la solution quitte à révolutionner et à changer de modèle de temps en temps suite à un eureka moment (appelé _Design Breakthrough_). C’est pour ça qu’on fait régulièrement des sessions de Context map et qui raffinent en permanence notre vision du domaine avec les produits.
- **Les équipes de dev ont des équilibres fragiles.** Il faut à peu près 6 mois de vie commune et de relations interpersonnelles pour qu’une équipe de dev soit réellement efficace. On ajoute ou retire une seule personne à cette équipe et ce n’est plus la même équipe (cf. [**Dynamic Reteaming**](https://www.amazon.fr/Dynamic-Reteaming-Wisdom-Changing-Teams/dp/1492061298)). En terme d’efficacité, je préfère des équipes qui restent ensemble et qui changent de sujets, que des équipes qu’on vient exploser/redistribuer au gré des sujets. Bien entendu, si quelqu’un en a marre de son équipe ou de ses sujets, tout est fait pour rendre possible son changement d’équipe (être bien personnellement est encore plus important pour notre efficacité collective)
- Notre organisation actuelle et nos équipes sont plutôt liées à un ou plusieurs Bounded Context afin de tenir compte de la complexité du problème et de capitaliser un minimum sur nos expertises métier respectives. De temps en temps, il se peut que certaines équipes aient un périmètre trop large et qu’il nous faille mieux découper et attribuer nos Bounded Context. Par contre, cette découpe des BCs doit rester liée à des concepts métier (remember DDD)
- **Rien n’interdit d’avoir des features teams dans une organisation qui ferait du DDD**, du moment que tout le monde respecte les frontières des Bounded Contexts.
- **Pour l’heure nous préférons largement la pratique du Swarming** (sorte de task forces montées avec des gens de plusieurs équipes mais où la responsabilité et l’ownership du chantier final est défini dès le début; pour éviter le syndrome du “Bon, on a fait quelque chose ensemble, mais qui va le maintenir maintenant ?!?”). **Au delà de son efficacité** à faire collaborer les bonnes personnes en fonction du sujet, **le swarming apporte énormément en capital social** en favorisant temporairement les relations inter-personnelles cross teams (super utile pour la suite quand tout le monde retourne dans son équipe).
- Notre organisation Produit actuelle est calée sur 1 PM = 1 équipe de dev, mais peut être serait il intéressant d’avoir des PMs plus associés aux sujets métiers qu’à leur équipe, quitte à ce qu’ils se voient associer à une autre équipe en fonction des sujets qu’ils ou elles maîtrisent ?

## There is no silver bullet

Pour finir, je dirai qu’il n’y a pas plus de silver bullet en organisation qu’en dev, et que le feedback, les questionnements et les expérimentations en permanence sont nos compagnons de route sur cette voie de l’amélioration continue pour faire efficacement un logiciel qui soit pertinent pour nos utilisateurs.