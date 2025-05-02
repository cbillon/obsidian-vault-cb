---
tags:
  - Lets-Encrypt
---

====== Let's Encrypt ======

[[https://www.edouard.bzh/guide-certificat-wildcard-lets-encrypt-acmesh-nginx-ovh/]]

[[tips:lets-encrypt/edouard|tips:lets-encrypt/edouard]]

[[https://www.cyberciti.biz/faq/find-check-tls-ssl-certificate-expiry-date-from-linux-unix/]]

Pour renouveler un certificat let's Encrypt générer ave cette méthode 

  ~/.acme.sh$ ./acme.sh --renew-all --force
  
  
  [Sun Dec  3 12:55:37 CET 2023] Renew: 'cbillon.ovh'
  [Sun Dec  3 12:55:37 CET 2023] Renew to Le_API=https://acme-v02.api.letsencrypt.org/directory
  [Sun Dec  3 12:55:38 CET 2023] Using CA: https://acme-v02.api.letsencrypt.org/directory
  [Sun Dec  3 12:55:38 CET 2023] Multi domain='DNS:cbillon.ovh,DNS:*.cbillon.ovh'
  [Sun Dec  3 12:55:38 CET 2023] Getting domain auth token for each domain
  [Sun Dec  3 12:55:41 CET 2023] Getting webroot for domain='cbillon.ovh'
  [Sun Dec  3 12:55:41 CET 2023] Getting webroot for domain='*.cbillon.ovh'
  [Sun Dec  3 12:55:41 CET 2023] cbillon.ovh is already verified, skip dns-01.
  [Sun Dec  3 12:55:41 CET 2023] *.cbillon.ovh is already verified, skip dns-01.
  [Sun Dec  3 12:55:41 CET 2023] Verify finished, start to sign.
  [Sun Dec  3 12:55:41 CET 2023] Lets finalize the order.
  [Sun Dec  3 12:55:41 CET 2023] Le_OrderFinalize='https://acme- 
  v02.api.letsencrypt.org/acme/finalize/1311001076/226574245566'
  [Sun Dec  3 12:55:43 CET 2023] Downloading cert.
  [Sun Dec  3 12:55:43 CET 2023] Le_LinkCert='https://acme- 
  v02.api.letsencrypt.org/acme/cert/03e0e659ae5cb7e449888a6f26e2d982c920'
  [Sun Dec  3 12:55:44 CET 2023] Cert success.

Pour afficher la date d'expiration d'un certificat

  openssl x509 -enddate -noout -in /etc/nginx/ssl/certs/cbillon.ovh.pem
  
  notAfter=Mar  2 10:55:42 2024 GMT  

