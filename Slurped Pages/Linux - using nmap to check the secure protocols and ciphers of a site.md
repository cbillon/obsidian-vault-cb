---
link: https://fabianlee.org/2023/02/11/linux-using-nmap-to-check-the-secure-protocols-and-ciphers-of-a-site/
site: "Fabian Lee : Software Engineer"
excerpt: "While enabling HTTPS is a important step in securing your web application, it is critical that you take steps to disable legacy protocols and low strength ciphers that can circumvent the very security you are attempting to implement. The Qualys SSL test is popular for grading the overall security of a public site, but you ... Linux: using nmap to check the secure protocols and ciphers of a site"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/cipher
  - slurp/nmap
  - slurp/protocol
  - slurp/strength
  - slurp/tls
  - slurp/tls1_1
  - slurp/tls1_2
  - nmap
  - ssl-ciphers
  - protocols
slurped: 2025-03-02T10:10
title: "Linux: using nmap to check the secure protocols and ciphers of a site"
---

![](https://fabianlee.org/wp-content/uploads/2018/10/gnu-logo.gif)While enabling HTTPS is a important step in securing your web application, it is critical that you take steps to disable legacy protocols and low strength ciphers that can circumvent the very security you are attempting to implement.

The [Qualys SSL test](https://www.ssllabs.com/ssltest/) is popular for grading the overall security of a public site, but you can also test the secure protocols and ciphers very easily using [nmap](https://nmap.org/) and its [ssl-enum-ciphers](https://nmap.org/nsedoc/scripts/ssl-enum-ciphers.html) script.  This has the advantage of being able to test your internal  corporate sites as well.

The commands below will run an nmap script that checks which protocols and ciphers are offered by a remote site.

# make sure nmap is installed
sudo apt install nmap -y

# get ssl-enum-ciphers script
wget http://nmap.org/svn/scripts/ssl-enum-ciphers.nse

# run TLS level and cipher test using nmap
FQDN=github.com
nmap --script ssl-enum-ciphers.nse -p 443 $FQDN

As an example, here is the output when run against the github.com site, showing its use of TLSv1.2 exclusively, and also listing each cipher available, each with an excellent grade of “A”.

$ nmap --script ssl-enum-ciphers.nse -p 443 github.com
Starting Nmap 7.80 ( https://nmap.org ) at 2023-02-11 11:52 EST
Nmap scan report for github.com (140.82.113.4)
Host is up (0.0088s latency).
rDNS record for 140.82.113.4: lb-140-82-113-4-iad.github.com

PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers: 
|   TLSv1.2: 
|     ciphers: 
|       TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 (ecdh_x25519) - A
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (ecdh_x25519) - A
|       TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256 (ecdh_x25519) - A
|       TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (ecdh_x25519) - A
|       TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 (ecdh_x25519) - A
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (ecdh_x25519) - A
|       TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 (ecdh_x25519) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (ecdh_x25519) - A
|       TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384 (ecdh_x25519) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 (ecdh_x25519) - A
|       TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (ecdh_x25519) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (ecdh_x25519) - A
|       TLS_RSA_WITH_AES_128_GCM_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_GCM_SHA384 (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|     compressors: 
|       NULL
|     cipher preference: server
|_  least strength: A

Nmap done: 1 IP address (1 host up) scanned in 1.26 seconds

This reinforces [the analysis from Qualys](https://www.ssllabs.com/ssltest/analyze.html?d=github.com&hideResults=on), also giving the github.com site an “A+”.

REFERENCES

[ssl-enum-ciphers script doc](https://nmap.org/nsedoc/scripts/ssl-enum-ciphers.html)

[nmap man page](https://linux.die.net/man/1/nmap)

[http://securityevaluators.com/knowledge/blog/20151102-openssl_and_ciphers/](http://securityevaluators.com/knowledge/blog/20151102-openssl_and_ciphers/)

[https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)

[http://blog.rlove.org/2013/12/strong-ssl-crypto.html](http://blog.rlove.org/2013/12/strong-ssl-crypto.html)

[https://securityevaluators.com/knowledge/blog/20150119-protocols/](https://securityevaluators.com/knowledge/blog/20150119-protocols/)

[http://www.acunetix.com/blog/articles/tls-ssl-cipher-hardening/](http://www.acunetix.com/blog/articles/tls-ssl-cipher-hardening/)

[https://www.owasp.org/index.php/Transport_Layer_Protection_Cheat_Sheet](https://www.owasp.org/index.php/Transport_Layer_Protection_Cheat_Sheet)

[fabianlee.org, using openssl to check secure protocols and ciphers](https://fabianlee.org/2017/01/24/openssl-using-openssl-to-enumerate-protocols-and-ciphers-in-use-by-web-applications/)