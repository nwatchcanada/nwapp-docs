The following instructions will help setup ``HTTP/2`` with our app.

Special thanks to:

* https://www.howtoforge.com/how-to-enable-http-2-in-nginx/

# Instructions

1. Open up our virtual host file.

        root# vi /etc/nginx/sites-available/nwapp.ws.conf

2. Add ``http2`` to the specific sections. When you finish it should look like the following:

```
server {
   if ($host ~ ^[^.]+\.markingcloud\.org$) {
       return 301 https://$host$request_uri;
   } # managed by Certbot


   if ($host = nwapp.ws) {
       return 301 https://$host$request_uri;
   } # managed by Certbot


    listen 80;
    server_name nwapp.ws;
   return 404; # managed by Certbot
}

server {
    listen  80;
    server_name *.nwapp.ws;
    charset     utf-8;
    access_log off;
    gzip on;
    client_max_body_size 0;  # Unlimited Upload File Size

    location / {
        add_header 'Access-Control-Allow-Origin' '*';
        root  /var/www/nwapp.ws/build;
        index  index.html index.htm;
        try_files $uri /index.html =404;
    }

    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root  /usr/share/nginx/html;
    }

    listen 443 ssl http2; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/nwapp.ws/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/nwapp.ws/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    ssl_protocols TLSv1.3;
}
```


(5) Confirm and restart the server.

```bash
$ sudo nginx -t
$ sudo systemctl restart nginx
```

(6) Confirm

```bash
curl -I http://nwapp.ws
curl -I http://london.nwapp.ws
```

(7) (OPTIONAL) Useful tutorial:

* https://www.digitalocean.com/community/tutorials/how-to-redirect-www-to-non-www-with-nginx-on-centos-7

* https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7
