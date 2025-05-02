====== Article Let's Encrypt edouard.bzh ======

La première pierre de ce projet est donc ce tutoriel dédié à la mise en place d’un certificat wildcard Let’s Encrypt en utilisant Acme.sh et l’API d’OVH. Ce tutoriel servira également de base à tous les suivants puisque j’utilise Nginx en reverse proxy de tous mes conteneurs Docker.

====== Installation des essentiels ======

Avant de nous lancer dans la génération du certificat, nous allons commencer par installer les paquets essentiels : nginx et curl. Toutes les étapes listées dans ce tutoriel ont été réalisées sur des machines sous Debian 11.

  apt update
  apt install nginx curl
  apt install --reinstall ca-certificates

On peut maintenant passer à l’installation de Acme.sh, un client ACME qui va nous permettre de générer facilement nos certificats Let’s Encrypt et de les renouveler automatiquement. Avec cette solution, nous n’aurons qu’une seule action à faire : la génération.

  curl https://get.acme.sh | sh -s email=votremail@gmail.com

====== Création des accès API OVH ======

Note : dans la suite de ce tutoriel, je vous invite à remplacer domain.tld par votre domaine.

Pour que Acme.sh puisse vérifier que nous sommes bien propriétaires des domaines pour lesquels nous allons générer les certificats, nous allons lui donner un accès à l’API OVH sur notre compte. Ainsi, toutes les étapes de configuration DNS pourront être réalisées automatiquement par le script lors de la génération du certificat.

Pour cela, rendez-vous à cette adresse : [[https://api.ovh.com/createToken/?GET=/domain/zone/domain.tld/*&POST=/domain/zone/domain.tld/*&PUT=/domain/zone/domain.tld/*&GET=/domain/zone/domain.tld&DELETE=/domain/zone/domain.tld/record/*|création d’un accès API OVH]]. Dans la fenêtre qui s’ouvre, renseigner vos identifiants OVH (n’oubliez pas le « - ovh » à la fin de votre identifiant). Définissez un nom et une description au choix et choisissez une validité infinie en sélectionnant « Unlimited ». Modifier ensuite les droits en remplaçant domain.tld par votre vrai domaine.

Cliquez ensuite sur « Create keys » et notez précieusement les informations sur la clé API.

De retour sur votre serveur, ouvrez le fichier /root/.acme.sh/account.conf pour y ajouter à la fin ces 3 lignes qu’il faudra évidemment compléter avec les identifiants générés juste au-dessus.

  SAVED_OVH_AK='app key'
  SAVED_OVH_AS='app secret'
  SAVED_OVH_CK='consumer key'

====== Génération du certificat Let’s Encrypt ======

On peut maintenant passer à la génération du certificat. La commande ci-dessous générera un certificat wildcard Let’s Encrypt avec Acme.sh. Il sera valable pour domain.tld ainsi que pour tous les sous-domaines *.domain.tld.

  /root/.acme.sh/acme.sh --issue --keylength 4096 -d domain.tld -d '*.domain.tld' --dns dns_ovh --server 
  letsencrypt


Si tout se passe bien, le script va tourner pendant plusieurs secondes afin de faire les différentes vérifications DNS. À la fin, si la génération s’est déroulée sans accroc, le script vous précisera l’emplacement des clés et certificats. Notez qu’à partir de ce moment, le certificat sera renouvelé automatiquement par Acme.sh.

====== Mise en place du certificat avec Nginx ======
Maintenant que le certificat a été généré, nous allons pouvoir le mettre en place avec Nginx. Personnellement, j’utilise un fichier qui contient tous les paramètres et configurations liées au certificat. J’ajoute ensuite ce fichier à tous mes vhosts Nginx qui utilisent le domaine associé à ce certificat.

Nous allons commencer par créer un répertoire /etc/nginx/ssl/certs pour y installer les fichiers générés avec Acme.sh. On créer d’abord le répertoire puis on demande à Acme.sh d’y copier le certificat et la clé. On précise ensuite la commande de redémarrage de Nginx.

  mkdir -p /etc/nginx/ssl/certs

  /root/.acme.sh/acme.sh --install-cert -d domain.tld \
  --key-file       /etc/nginx/ssl/certs/key-domain.tld.key  \
  --fullchain-file /etc/nginx/ssl/certs/cert-domain.tld.pem \
  --reloadcmd     "systemctl restart nginx"

On va maintenant créer le fichier /etc/nginx/ssl/ciphers-domain.tld.conf dans lequel on va préciser les paramètres de sécurité. Cette configuration se base sur les préconisations de Mozilla que vous pouvez retrouver [[https://ssl-config.mozilla.org/|ici]]. N’oubliez pas de modifier domain.tld partout dans le fichier.

    # generated 2023-04-06, Mozilla Guideline v5.6, nginx 1.17.7, OpenSSL 1.1.1k,configuration
   
   ssl_certificate_key  /etc/nginx/ssl/certs/cbillon.ovh.key;
   ssl_certificate /etc/nginx/ssl/certs/cbillon.ovh.pem;
   ssl_session_timeout 1d;
   ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
   ssl_session_tickets off;

   # curl https://ssl-config.mozilla.org/ffdhe2048.txt > /path/to/dhparam
   ssl_dhparam /etc/nginx/ssl/dhparam;

   # intermediate configuration 
   #protocols TLSv1.2 TLSv1.3;
   #ssl-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM- 
   SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA- 
   AES128-GCM-SHA2>

   # modern configuration
   ssl_protocols TLSv1.3;

   ssl_prefer_server_ciphers off;

   # HSTS (ngx_http_headers_module is required) (63072000 seconds)
   add_header Strict-Transport-Security "max-age=63072000" always;

   # OCSP stapling
   ssl_stapling on;
   ssl_stapling_verify on;

   # verify chain of trust of OCSP response using Root CA and Intermediate certs
   ssl_trusted_certificate /etc/nginx/ssl/certs/cbillon.ovh.pem;

   # replace with the IP address of your resolver  
   # dns11.ovh.net 213.251.188.130
   #  ns11.ovh.net 213.251.128.130
   resolver 213.251.188.130;


On récupère enfin le fichier dhparam fourni par Mozilla :

  curl https://ssl-config.mozilla.org/ffdhe2048.txt > /etc/nginx/ssl/dhparam

====== Utilisation du certificat dans un vhost Nginx ======

Maintenant que le certificat est complètement installé et que nous avons mis en place la configuration Nginx associée, il ne reste plus qu’à l’utiliser dans nos vhosts. Pour cela, il suffit simplement d’appeler le fichier ciphers-domain.tld.conf que nous avons créé précédemment dans chacun de nos vhosts. Par exemple, voici une configuration classique en reverse proxy où le fichier est appelé :



  server {
    listen              80;
    server_name         omv.domain.tld;
    location / {
      return              301 https://$server_name$request_uri;
    }
  }

  server {
      listen 443 ssl http2;
      server_name omv.domain.tld;

      include /etc/nginx/ssl/ciphers-domain.tld.conf;

      access_log /var/log/nginx/omv.domain.tld.log combined;
      error_log /var/log/nginx/omv.domain.tld.log error;

      location / {
          proxy_pass http://127.0.0.1:8080;
          proxy_set_header Host $host;
          proxy_redirect http:// https://;
          proxy_http_version 1.1;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection $connection_upgrade;
          proxy_set_header X-Real-IP $remote_addr;
      }
  }


Vous savez maintenant comment générer et utiliser un certificat wildcard Let’s Encrypt avec Acme.sh, Nginx et OVH. Comme toujours, je reste disponible en commentaire ou sur Twitter si vous avez la moindre question. Avec cette configuration, vous n’aurez normalement plus à vous soucier de votre certificat puisqu’il sera renouvelé automatiquement.