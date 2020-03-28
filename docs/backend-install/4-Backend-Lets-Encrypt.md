#### Nginx

The following instructions will enable wildcard SSL certificates for CentOS 8
using the DigitalOcean Plugin.

Special thanks:
https://certbot.eff.org/lets-encrypt/centosrhel8-nginx
https://certbot.eff.org/docs/using.html
https://certbot-dns-digitalocean.readthedocs.io/en/stable/
https://stackoverflow.com/a/53830571


1. Download and configure

```bash
$ cd /usr/src
$ wget https://dl.eff.org/certbot-auto
$ chmod a+x certbot-auto
$ sudo ./certbot-auto --debug --install-only

$ cd /opt/eff.org/certbot/venv
$ source bin/activate
$ sudo pip install certbot-dns-digitalocean
$ deactivate
```


DEPRECATED BELOW - SKIP
```
$ sudo mv certbot-auto /usr/local/bin/certbot-auto
$ sudo chown root /usr/local/bin/certbot-auto
$ sudo chmod 0755 /usr/local/bin/certbot-auto
```

2. Log into (DigitalOcean)[https://digitalocean.com] and create an ``API key``.
https://certbot-dns-digitalocean.readthedocs.io/en/latest/#

3. Create our credentials file:

  ```
  $ sudo mkdir -p /etc/letsencrypt/digitalocean
  $ sudo cat > /etc/letsencrypt/digitalocean/credentials.ini
  ```

4. Populate the file with your key. Here is example:

  ```
  # DigitalOcean API credentials used by Certbot
  dns_digitalocean_token = 0000111122223333444455556666777788889999aaaabbbbccccddddeeeeffff
  ```

5. Better permissions

  ```
  sudo chmod 700 /etc/letsencrypt/digitalocean/credentials.ini
  ```


6. Open up our virtual host file.

```
sudo vi /etc/nginx/sites-available/nwapp.ws.conf
```


Make sure you have the domain information specified. Once you are ready, continue...

(3) Install

    $ cd /usr/bin/
    $ certbot certonly \
    --dns-digitalocean \
    --dns-digitalocean-credentials /etc/letsencrypt/digitalocean/credentials.ini \
    --dns-digitalocean-propagation-seconds 60 \
    --nginx \
    -d "*.nwapp.ws" -d nwapp.ws


## HOW DO WE AUTO RENEW?
(1) Run the following command:

```
sudo crontab -e
```

(2) Append the following contents.

```
# Add this to the crontab and save it:
0 0,12 * * * /usr/bin/certbot renew && systemctl restart nginx
```

(3) Would you like to know more? Please read more [here](https://certbot.eff.org/lets-encrypt/centosrhel7-nginx.html).

# Update Nginx Config

(1) Open up our virtual host file.

```
vi /etc/nginx/sites-available/nwapp.ws.conf
```

(2) Completely replace the contents of the file with the following by copy and pasting:

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

    server_name nwapp.ws *.nwapp.ws;

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

(7) (OPTIONAL) Useful tutorial: https://www.digitalocean.com/community/tutorials/how-to-redirect-www-to-non-www-with-nginx-on-centos-7
