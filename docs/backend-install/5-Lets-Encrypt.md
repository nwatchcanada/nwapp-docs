* https://blogs.msdn.microsoft.com/mihansen/2018/03/15/creating-wildcard-ssl-certificates-with-lets-encrypt/

## Instructions
### Let's Encrypt
#### Nginx
The following instructions are used to manually setup `letsencrypt` and automatically integrate with `nginx`.

1. Install our **Lets Encrypt** client.

  ```
  sudo yum install -y certbot-nginx
  ```

2. Generate our certificate.

  ```
  certbot --nginx --agree-tos --server https://acme-v02.api.letsencrypt.org/directory --preferred-challenges dns -d "*.nwapp.ca" -d nwapp.ca
  ```

3. Follow the instructions and choose the most appropriate options.

4. Restart ``nginx`` and your ``supervisord``.

  ```
  systemctl restart nginx
  systemctl restart supervisord
  ```

5. Using your favourite browser, load up (https://nwapp.ca/en/)[https://nwapp.ca/en/] and it should work. If it does then congradulations!

#### DigitalOcean
The problem with the above instructions is that **you are responsible for manually renewing within 90 days**. This manual renewing is tedious, can we automate? Turns out we can. The above instructions setup ``letsencrypt`` with our ``nginx`` so we have it working. In this section we will integrate our code with ``DigitalOcean`` and make auto-renewing taken care of by a script.

1. Install our ``DigitalOcean`` plugin.

  ```
  sudo yum install -y certbot-dns-digitalocean
  ```

2. Log into (DigitalOcean)[https://digitalocean.com] and create an ``API key``.
https://certbot-dns-digitalocean.readthedocs.io/en/latest/#

3. Create our credentials file:

  ```
  mkdir /etc/letsencrypt/digitalocean
  cat > /etc/letsencrypt/digitalocean/credentials.ini
  ```

4. Populate the file with your key. Here is example:

  ```
  # DigitalOcean API credentials used by Certbot
  dns_digitalocean_token = 0000111122223333444455556666777788889999aaaabbbbccccddddeeeeffff
  ```

5. Finally run the code which will automatically generate our certificate.

  ```
  certbot certonly --dns-digitalocean --dns-digitalocean-credentials /etc/letsencrypt/digitalocean/credentials.ini --dns-digitalocean-propagation-seconds 60 -d "*.nwapp.ca" -d nwapp.ca
  ```

6. (Optional) https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7

  ```bash
  sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
  ```

7. Restart the server.

    ```
    sudo systemctl restart nginx
    ```

8. Would you like to know more?

* https://certbot.eff.org/lets-encrypt/centosrhel7-nginx
* https://certbot-dns-digitalocean.readthedocs.io/en/latest/


HOW DO WE AUTO RENEW?
https://certbot.eff.org/lets-encrypt/centosrhel7-nginx.html

sudo crontab -e

# Add this to the crontab and save it:
0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && /usr/bin/certbot renew && systemctl restart nginx


## NWApp.ws

```
certbot certonly --dns-digitalocean --dns-digitalocean-credentials /etc/letsencrypt/digitalocean/credentials.ini --dns-digitalocean-propagation-seconds 60 -d "*.nwapp.ws" -d nwapp.ws
```

```
vi /etc/nginx/sites-available/nwapp.ca.conf
```

```
server {
   if ($host ~ ^[^.]+\.nwapp\.ca$) {
       return 301 https://$host$request_uri;
   } # managed by Certbot


   if ($host = nwapp.ca) {
       return 301 https://$host$request_uri;
   } # managed by Certbot


    listen 80;
    server_name nwapp.ca;
   return 404; # managed by Certbot
}

server {
    listen  80;
    server_name nwapp.ca;
    charset     utf-8;
    access_log off;
    gzip on;
    client_max_body_size 0;  # Unlimited Upload File Size

    location / {
        root  /var/www/nwapp.ca/build;
        index  index.html index.htm;
        try_files $uri /index.html =404;
    }

    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root  /usr/share/nginx/html;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/nwapp.ca/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/nwapp.ca/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```


```
vi /etc/nginx/sites-available/nwapp.ws.conf
```

```
server {
   if ($host ~ ^[^.]+\.nwapp\.ws$) {
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
        listen 80;

        server_name *.nwapp.ws;

        charset     utf-8;
        access_log off;
        gzip on;
        client_max_body_size 0;  # Unlimited Upload File Size

        # Optional if you want logging in Nging.
        access_log /var/log/nginx/default-access.log;
        error_log /var/log/nginx/default-error.log;

        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
            root /opt/django/nwapp-back/nwapp;
        }
        location /staticfiles/ {
            root /opt/django/nwapp-back/nwapp;
        }
        location /media/ {
            root /opt/django/nwapp-back/nwapp;
        }

        location / {
            proxy_set_header         Host $host;  # (1)
            proxy_pass               http://127.0.0.1:8001;
            proxy_set_header         X-Forwarded-Host $server_name;
            proxy_set_header         X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header         X-Real-IP $remote_addr;
            add_header               P3P 'CP="ALL DSP COR PSAa PSDa OUR NOR ONL UNI COM NAV"';

            # Extend our timeout to handle longer processes.
            proxy_connect_timeout 75s;
            proxy_read_timeout 300s;
        }

        listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/nwapp.ws/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/nwapp.ws/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    }
```


Redirect all http traffic.
```
vi /etc/nginx/nginx.conf
```

```
server {
    listen 80 default_server;

    server_name _;

    return 301 https://$host$request_uri;
}
```
