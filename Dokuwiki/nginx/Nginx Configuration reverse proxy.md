---
tags:
  - nginx
  - reverse-proxy
---

====== Nginx Configuration dun reverse proxy ======

[[https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-on-ubuntu-22-04
]]

Configuration exemple

  server {
      listen 80;
      listen [::]:80;
      server_name dokuwiki.cbillon.ovh;        
      location / {
          proxy_pass http://192.168.122.27:80;
        
          proxy_set_header Host $http_host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }
