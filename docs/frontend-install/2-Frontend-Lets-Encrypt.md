(1) Please make sure the backend application has had the wildcard SSL implemented.

(3) Install

    $ cd /opt/eff.org/certbot/venv
    $ source bin/activate
    $ certbot certonly \
    --dns-digitalocean \
    --dns-digitalocean-credentials /etc/letsencrypt/digitalocean/credentials.ini \
    --dns-digitalocean-propagation-seconds 60 \
    -d "*.nwapp.ca" -d nwapp.ca;


# Update Nginx Config

1. Open the file.

    ```
    sudo vi /etc/nginx/sites-available/nwapp.ca.conf
    ```


2. Copy and paste the following.

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
    server_name *.nwapp.ca;
    charset     utf-8;
    access_log off;
    gzip on;
    client_max_body_size 0;  # Unlimited Upload File Size

    location / {
        add_header 'Access-Control-Allow-Origin' '*';
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

3. Confirm and restart the server.

    ```
    $ sudo nginx -t
    $ sudo systemctl restart nginx
    ```

4. Confirm that the `Location` returned is `https://nwapp.ca` once you run the following command:

    ```
    curl -I http://nwapp.ca
    ```

5. (OPTIONAL) Useful tutorial: https://www.digitalocean.com/community/tutorials/how-to-redirect-www-to-non-www-with-nginx-on-centos-7

## Auto-Renew

```
sudo crontab -e
```

add

```
0 0,12 * * * /opt/eff.org/certbot/venv/bin/certbot renew && systemctl restart nginx
```
