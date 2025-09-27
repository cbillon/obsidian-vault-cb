---
link: https://rodolphe.breard.tf/article/signature-electronique/
byline: Rodolphe Bréard
site: Accueil
date: 2025-02-13T01:00
excerpt: Savez-vous précisément ce qu’est une signature électronique et quel est son cadre légal ? Le sujet étant plus complexe qu’il n’y parait, c’est assez flou pour la plupart des gens, ce qui est fort dommage car ce mode de signature permet généralement d’économiser du papier, du temps et bien d’autres choses. Voyons donc ensemble ce qu’il en est vraiment.
slurped: 2025-09-13T19:53
title: La signature électronique
tags:
  - signature-electronique
---

Savez-vous précisément ce qu’est une signature électronique et quel est son cadre légal ? Le sujet étant plus complexe qu’il n’y parait, c’est assez flou pour la plupart des gens, ce qui est fort dommage car ce mode de signature permet généralement d’économiser du papier, du temps et bien d’autres choses. Voyons donc ensemble ce qu’il en est vraiment.

## Les types de signature électroniques [#](https://rodolphe.breard.tf/article/signature-electronique/#les-types-de-signature-%c3%a9lectroniques)

Si on parle généralement de la signature électronique au singulier, la réglementation distingue en réalité 4 niveaux de signature :

1. La signature simple : c’est le fait d’insérer une image de signature manuscrite sur un document ou, d’une manière plus générale, tout autre procédé qui y ressemble.
2. La signature avancée : là on parle d’une signature au sens cryptographique du terme, le signataire détient une clé secrète (certificat) dont il se sert pour ajouter une signature publique sur un document.
3. La signature avancée reposant sur un certificat de signature électronique qualifié : c’est la même chose que la signature avancée, sauf que dans ce cas le certificat du signataire a été émis par un prestataire de confiance reconnu après que ce dernier a vérifié l’identité du signataire.
4. La signature qualifiée : c’est la même chose que la signature avancée reposant sur un certificat de signature électronique qualifié, sauf que pour signer le signataire est en plus contraint d’utiliser un dispositif physique qui a fait l’objet d’une qualification. En règle générale le certificat est incorporé dans ce dispositif et ne peut ni en être extrait ni copié.

Concrètement, ça signifie que si vous générez votre propre certificat de signature et que vous l’utilisez pour signer, c’est du niveau 2 (avancé). Si c’est une autorité de certification qualifiée qui vous délivre un certificat et que vous utilisez un logiciel de votre choix pour signer, c’est du niveau 3 (avancé reposant sur un certificat de signature électronique qualifié). Et enfin, si c’est une autorité de certification qui vous délivre un dispositif physique qualifié (souvent sous forme d’une clé USB ou d’une carte à puce) et que vous utilisez ce dispositif pour signer, alors c’est du niveau 4 (qualifié).

Du point de vue du vocabulaire on distinguera d’un côté la signature faite au nom d’une personne physique que l’on appelle une signature et de l’autre côté la signature faite au nom d’une personne morale (association, entreprise, etc.) que l’on appelle un cachet. C’est techniquement identique, seul le vocabulaire change.

## La portée légale [#](https://rodolphe.breard.tf/article/signature-electronique/#la-port%c3%a9e-l%c3%a9gale)

Les signatures électroniques sont reconnues par la loi et il n’est donc pas possible de les refuser au seul motif qu’elles sont électroniques. En revanche, il est possible de les refuser au motif que le niveau de la signature n’est pas suffisant compte tenu de l’importance du document signé. C’est ce que l’on retiendra des articles [1366](https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000032042461) et [1367](https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000032042456) du Code civil :

> L’écrit électronique a la même force probante que l’écrit sur support papier, sous réserve que puisse être dûment identifiée la personne dont il émane et qu’il soit établi et conservé dans des conditions de nature à en garantir l’intégrité.

> La signature nécessaire à la perfection d’un acte juridique identifie son auteur. Elle manifeste son consentement aux obligations qui découlent de cet acte. Quand elle est apposée par un officier public, elle confère l’authenticité à l’acte.
> 
> Lorsqu’elle est électronique, elle consiste en l’usage d’un procédé fiable d’identification garantissant son lien avec l’acte auquel elle s’attache. La fiabilité de ce procédé est présumée, jusqu’à preuve contraire, lorsque la signature électronique est créée, l’identité du signataire assurée et l’intégrité de l’acte garantie, dans des conditions fixées par décret en Conseil d’Etat.

Il est tout à fait normal que ces deux textes soient assez vagues, cela permet de ne pas avoir besoin de changer la loi à chaque avancée technologique. À la place, les détails sont pris soit par décret soit par arrêté, ce qui permet au gouvernement d’adapter la réglementation sans avoir besoin de refaire tout le processus législatif.

L’autre raison de cette rédaction très généraliste est de ne pas entrer en conflit avec le règlement [eIDAS](https://eur-lex.europa.eu/eli/reg/2014/910/oj?locale=fr). Pour rappel, contrairement aux directives européennes qui doivent êtres retranscrites dans le droit de chaque État membre, les règlements européens sont, eux, directement applicables. Vous remarquerez au passage que le règlement eIDAS est lui aussi assez lacunaire et que les détails sont pris dans différents [actes d’exécution](https://cyber.gouv.fr/referentiel-documentaire-lie-au-reglement-eidas).

Ceci dit, comme vous l’aurez deviné, tous les niveaux de signature électronique ne se valent pas. On dit généralement que seul le niveau 4 (qualifié) est l’équivalent de la signature manuscrite. Pour être plus précis, il s’agit du seul niveau pour lequel [la réglementation](https://www.legifrance.gouv.fr/loda/id/JORFTEXT000035676246) prévoit que c’est à la personne contestant la signature de prouver que la signature n’est pas valable. Pour tous les autres niveaux, si quelqu’un conteste une signature, c’est à la personne ne la contestant pas de prouver que la signature est valable.

Forcément, plus le niveau de signature est élevé, plus il est facile de prouver que la signature est valide. La signature de niveau 1 (simple) a donc une valeur juridique très limitée. Un cas de jurisprudence célèbre où elle a été jugée suffisante est celui où le directeur général de l’Urssaf Île-de-France a utilisé un tel procédé afin de signer une lettre de déclaration de créance auprès d’une entreprise en situation de cessation des paiements. L’entreprise a alors contesté cette créance en argumentant que la signature n’était pas valide. Si en première instance un juge a donné raison à la société, l’Urssaf a fait appel et obtenu de la cour d’appel qu’elle reconnaisse la signature (et la créance) comme valide, infirmant ainsi le premier jugement ([CA Paris, 25 janvier 2018, no 17/01273](https://juricaf.org/arret/FRANCE-COURDAPPELDEPARIS-20180125-1701273)).

Si la signature de niveau 2 (avancé) est plus difficile à contester, il faut garder en têtes qu’elles ne se valent pas toutes. En effet, n’importe qui avec un minimum de connaissances techniques sur le sujet peut créer des certificats permettant de signer avec le nom que l’on souhaite. On se retrouve alors dans une situation similaire de celle avec une signature simple : tant que vous signez un document que vous avez émis, en cas de contestation il vous suffit d’affirmer que c’est bien vous qui avez signé pour que ça passe. Mais si c’est un tiers qui nie avoir signé, ce sera à vous de prouver qu’il ment et qu’il a bien signé. C’est pour cela qu’en général on passe par des prestataires de confiance qui conserve un dossier de preuve (souvent les traces de validation par SMS) plutôt que de faire ça soi-même.

Pour la signature de niveau 3, là on est sur du sérieux et c’est assez facile à prouver et donc difficile à contester. Je ne l’aborderai donc pas spécialement.

La signature de niveau 4 (qualifiée) est bien différente. En effet, comme nous l’avons vu, en cas de contestation c’est à la partie contestant la signature de prouver que cette signature n’est pas valable. Vu le niveau de sécurité de la chose, c’est très difficile voir impossible. Le seul exemple valable qui me vient en tête est le cas où c’est le signataire qui conteste sa propre signature au motif qu’elle lui a été extorquée par violence, menaces ou chantage.

## Un mot sur le RGS [#](https://rodolphe.breard.tf/article/signature-electronique/#un-mot-sur-le-rgs)

La France dispose d’une réglementation spécifique, le [référentiel général de sécurité](https://cyber.gouv.fr/le-referentiel-general-de-securite-rgs) (RGS). Il s’agit de règles de sécurités spécifiques aux autorités administratives au sens large. Cette réglementation leur impose notamment l’usage de certificats de sécurité disposant d’une qualification particulière propre au RGS avec un niveau de 1, 2 ou 3 étoiles.

C’est pour cela que, si vous regardez les produits des différentes autorités de certifications, vous verrez plein de types de certificats différents, certains pouvant être labellisés « eIDAS » (délivrés par une AC qualifiée intégrée à l’EUTL, voir ci-après) et/ou RGS. Vous en trouverez également d’autres qui permettent l’authentification après de certains services, notamment le [SIV](https://fr.wikipedia.org/wiki/Plaque_d%27immatriculation_fran%C3%A7aise#SIV_:_syst%C3%A8me_d%27immatriculation_des_v%C3%A9hicules), mais c’est une autre histoire.

## Comment ça se passe techniquement ? [#](https://rodolphe.breard.tf/article/signature-electronique/#comment-%c3%a7a-se-passe-techniquement)

C’est là que les choses se corsent car il y a de multiples moyens de signer numériquement un document. Certains formats, tels que PDF, intègrent nativement un processus de signature. Pour les autres formats de fichiers, il peut y avoir un fichier de signature séparé.

D’une manière générale, aucune méthode n’est imposée mais une des [décisions d’exécution d’eIDAS](https://eur-lex.europa.eu/eli/dec_impl/2015/1506/oj?locale=fr) reconnaît explicitement les méthodes [XAdES](https://en.wikipedia.org/wiki/XAdES), [CAdES](https://en.wikipedia.org/wiki/CAdES_%28computing%29) et [PAdES](https://en.wikipedia.org/wiki/PAdES). Cependant, dans certains contextes, notamment les [marchés publics](https://www.legifrance.gouv.fr/loda/id/JORFTEXT000038318621/), certaines méthodes et niveaux de sécurité peuvent être imposés. La bonne nouvelle c’est que PAdES est la méthode de signature la plus courante car nativement intégrée dans le format PDF.

En revanche, il faut absolument garder un point d’attention en tête, celui des autorités de certification. Dès que l’on souhaite faire une signature de niveau 3 ou 4, il est nécessaire que le certificat ait été émis par une autorité de certification (AC) qualifiée. Chaque État membre dispose d’une agence officielle (l’ANSSI en France) en charge de qualifier les AC de confiance et de remonter cette information au niveau européen. L’ensemble des AC qualifiées de chacun des pays de l’UE forment donc la « European Union Trusted Lists » (EUTL), ce qui permet d’avoir la liste de l’ensemble des certificats racine reconnus au niveau de l’UE.

Cette notion d’AC qualifiée est primordial pour la vérification de la validité des signatures de niveau 3 et 4. Il pourrait être tentant de se dire que si votre lecteur PDF, par exemple Adobe Acrobat Reader, affiche que la signature d’un fichier PDF est valide et reconnue, alors il s’agit d’une signature de niveau 3 ou 4 tandis qu’une signature valide mais non-reconnue serait du niveau 2. Or, certains logiciels, [notamment ceux d’Adobe](https://helpx.adobe.com/fr/acrobat/kb/trust-services.html), incluent l’EUTL comme source de confiance aux côtés d’autres sources qui ne sont pas qualifiées au niveau de l’UE et affichent donc comme valides et reconnues des signatures qui ne sont que du niveau 2 car le certificat n’est pas qualifié.

## En pratique, comment signer ? [#](https://rodolphe.breard.tf/article/signature-electronique/#en-pratique-comment-signer)

Je vous ferai pas l’affront de vous expliquer comment scanner votre signature manuscrite et intégrer cette image dans un document. En revanche, vous serez peut-être surpris d’apprendre que, si vous disposez d’un certificat, [LibreOffice peut l’utiliser pour signer des documents](https://help.libreoffice.org/latest/fr/text/shared/01/digitalsignatures.html). Oui, ce logiciel peut meme signer des PDF générés par un autre moyen.

À noter que [Stirling PDF](https://www.stirlingpdf.com/) permet de signer et vérifier la signature des PDF. Pour la signature, il faut cependant bien sélectionner l’option « Signer avec un certificat » et faire bien attention au résultat car cette fonctionnalité est, à l’heure où j’écris ces lignes, indiquée comme un « travail en cours ».

Et voilà comment je viens de vous éviter de dépenser de l’argent pour vous offrir la version payante de certains logiciels de PDF tels qu’Acrobat Reader ou Foxit.

Ceci dit, les logiciels précités ne sont utiles que si l’on dispose déjà d’un certificat et que l’on est la seule personne à signer le document. Dans la mesure où un certificat auto-généré n’aura que peu de valeur et sera très certainement refusé par les autres parties et qu’un certificat qualifié peut s’avérer coûteux pour un particulier, ce n’est pas forcément la solution qui va rendre la signature électronique à la portée de tout le monde. En plus, si vous souhaitez faire de la signature qualifiée, non seulement vous ne pourrez sans doute pas vous passer d’un logiciel payant, mais en plus les dispositifs nécessitent le plus souvent un middleware qui n’est disponible que sous Windows, les autres OS ayant de fortes limitations ou n’étant tout simplement pas supportés.

Afin de ne pas vous laisser sur une note négative, je tiens à signaler que la société Goodflag (anciennement Lex Persona) a lancé le portail communautaire [Lex Community](https://lex.community/). Il s’agit d’une plateforme destinée à fournir gratuitement des services basiques de signature électronique. En pratique, vous créez un « parapheur » de documents qui sera signé par une ou plusieurs personnes. Vous pouvez donc vous ajouter vous-même en signataire ainsi que d’autres personnes. Bien entendu, vous pouvez définir le niveau des signatures, ce qui nécessitera un niveau de vérification plus ou moins élevé.

La bonne nouvelle c’est que Lex Community permet actuellement de faire de la signature qualifiée à condition de disposer d’une [identité numérique La Poste](https://lidentitenumerique.laposte.fr/). Cette méthode permet à l’AC de vérifier votre identité avec fiabilité et donc de générer un certificat qualifié dans un dispositif également qualifié (le tout restant sous leur contrôle exclusif bien entendu) afin de signer le document que vous avez téléversé.