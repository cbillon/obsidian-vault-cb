---
tags:
  - moodle
  - install
  - reverse-proxy
---
## Moodle 5.2 router behind Nginx reverse proxy

par [Jens Flemming](https://moodle.org/user/view.php?id=5191768&course=5), mardi 28 avril 2026, 18:46

Here come the relevant config snippets (after moodle.org locked my account due to (wrongly) suspected spam; after contacting support they unlocked it two days later...). For some reason I cannot format this reply...  
  
------------------------------------  
Reverse proxy (Nginx) config on the host (note the rewrite!):  
  
location /moodle/ {  
rewrite /moodle/(.*) /$1 break;  
proxy_pass [http://127.0.0.1:9001](http://127.0.0.1:9001/);  
proxy_redirect off;  
proxy_set_header X-Real-IP $remote_addr;  
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
proxy_set_header X-Forwarded-Proto $scheme;  
proxy_set_header Upgrade $http_upgrade;  
proxy_set_header Connection $connection_upgrade;  
}  
  
------------------------------------  
Nginx in the container (note the /moodle in the @rphp location block):  
  
root /var/www/html/moodle/public;  
index index.php;  
  
absolute_redirect off;  
  
location / {  
try_files $uri $uri/ @rphp;  
}  
  
location @rphp {  
fastcgi_pass unix:/run/php/php-fpm.sock;  
include fastcgi_params;  
fastcgi_param SCRIPT_FILENAME $realpath_root/r.php;  
fastcgi_param DOCUMENT_ROOT $realpath_root;  
fastcgi_param PATH_INFO /moodle$uri;  
fastcgi_param DOCUMENT_URI /moodle$uri;  
fastcgi_param REQUEST_URI /moodle$uri$is_args$args;  
fastcgi_param SCRIPT_NAME /moodle$uri;  
}  
  
location ~ \.php(/|$) {  
fastcgi_split_path_info ^(.+\.php)(/.*)$;  
set $path_info $fastcgi_path_info;  
try_files $fastcgi_script_name $fastcgi_script_name/ /r.php$is_args$args;  
fastcgi_pass unix:/run/php/php-fpm.sock;  
include fastcgi_params;  
fastcgi_param PATH_INFO $path_info;  
fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;  
fastcgi_param DOCUMENT_ROOT $realpath_root;  
}  
  
------------------------------------  
Moodle's config.php:  
  
$CFG->wwwroot = '[https://192.168.178.229/moodle'](https://192.168.178.229/moodle');  
$CFG->reverseproxy = true;  
$CFG->routerconfigured = true;  
$CFG->sslproxy = true;  
  
As I said, this config works (that is, problem solved), but the docs suggest a (in my case) non-functional config. Wondering whether my setup is that special that the docs do not cover it?  
  
From my (very limited) point of view paths for both static files and the router should use the same base path assumption. For static files the (implicit) assumption is, that the base path is /. But the router extracts the base path from wwwroot, resulting in /moodle. The webserver config has to distinguish between both cases and provide different URIs...  
  
Any hints on alternative/simpler configs? Or is it a bug, I/someone should report?  
  
Thanks!