---
link: https://dolys.fr/forums/topic/comprendre-les-tests-de-fuite-dns-et-depannage-dns-dot-et-dns-doh/
byline: Auteur
site: l'Almanet doLys de nam1962 et ses amis
excerpt: "Après avoir configuré vos systèmes DNS sécurisés, vous voudrez naturellement vérifier qu'ils fonctionnent correctement. Voici quelques précisions importantes : Si vous utilisez des outils comme dnsleaktest.com, ipleak.net ou browserleaks.com/dns, vous pourriez être surpris de voir des serveurs Google apparaître dans vos résultats, même avec votre configuration DoT ou DoH parfaitement…"
twitter: https://twitter.com/https://twitter.com/e_dolys
slurped: 2026-03-29T16:41
title: Comprendre les tests de fuite DNS et dépannage DNS-DoT et DNS-DoH | l'Almanet doLys de nam1962 et ses amis
tags:
  - dns
---

Après avoir configuré vos systèmes DNS sécurisés, vous voudrez naturellement vérifier qu’ils fonctionnent correctement. Voici quelques précisions importantes :

**1. Les tests de fuite DNS peuvent être trompeurs**

Si vous utilisez des outils comme dnsleaktest.com, ipleak.net ou browserleaks.com/dns, vous pourriez être surpris de voir des serveurs Google apparaître dans vos résultats, même avec votre configuration DoT ou DoH parfaitement fonctionnelle.

_Explication technique_ : Ces tests ne montrent pas vos serveurs DNS, mais les serveurs d’hébergement des domaines que le test contacte. C’est comme si vous demandiez l’adresse d’un restaurant à un ami de confiance (votre serveur DNS), puis le test montrait que vous êtes allé au restaurant (serveurs Google).

**2. Comment vérifier que votre configuration fonctionne réellement**

La véritable confirmation est visible dans les logs de vos conteneurs :

```

# Pour voir quels serveurs DNS sont utilisés par DNSCrypt-Proxy
docker logs dnscrypt-proxy

# Pour vérifier les connexions TLS d'Unbound
docker logs unbound
```

**3. Outils de test recommandés**

Pour une vérification complète, utilisez plusieurs outils qui se complètent :  
– [](https://ipleak.net/)IPLeak.net[/url] : Vue d’ensemble incluant IP, DNS et WebRTC  
– [](https://dnsleaktest.com/)DNSLeakTest.com[/url] (test étendu) : Vérification DNS approfondie  
– [BrowserLeaks.com/dns](https://browserleaks.com/dns) : Détails techniques sur le comportement DNS de votre navigateur  
– [](https://dnsleaktest.org/)DNSLeakTest.org[/url] : Confirmation supplémentaire

**4. Pour les puristes qui s’interrogent sur Docker**

Certains puristes pourraient questionner l’utilisation de Docker pour ces configurations DNS. Voici pourquoi c’est un choix pratique :

– **Isolation des dépendances** : Unbound et Pi-hole ont leurs propres exigences qui restent isolées du système  
– **Portabilité** : Même configuration sur Debian, Arch, ou n’importe quelle distribution  
– **Mises à jour simplifiées** : Images maintenues par des équipes spécialisées  
– **Facilité de bascule** : Passer de DoT à DoH sans risquer de casser votre résolution DNS système  
– **Reconstruction rapide** : En cas de problème, tout peut être rétabli en quelques minutes

L’overhead est négligeable pour ce type d’application, et les avantages pratiques l’emportent largement. Si vous préférez une installation native, les configurations peuvent être facilement adaptées – seuls les chemins devront être ajustés.

Bonne navigation, libre et privée !
