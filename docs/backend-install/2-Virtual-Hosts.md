The following instructions where modified from [How To Set Up Nginx Server Blocks on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-on-centos-7). Before proceeding, please make sure you completed the [previous instructions](/1-DigitalOcean-Droplet.md) before proceeding.

(1) Open up our ``Nginx`` configuration file:

```bash
vi /etc/nginx/nginx.conf
```

(2) Append to the end of the ``http {}`` code block:

```
include /etc/nginx/sites-enabled/*.conf;
server_names_hash_bucket_size 64;
```

(3) **Delete** the following code block:

```
server {
   listen       80 default_server;
   listen       [::]:80 default_server;
   server_name  _;
   root         /usr/share/nginx/html;

   # Load configuration files for the default server block.
   include /etc/nginx/default.d/*.conf;

   location / {
   }

   error_page 404 /404.html;
       location = /40x.html {
   }

   error_page 500 502 503 504 /50x.html;
       location = /50x.html {
   }
}
```

(4) Create the directory structure.

```bash
sudo mkdir -p /var/www/nwapp.ws/html
```

(5) Grand permission to the regular users.

```bash
sudo chown -R $USER:$USER /var/www/nwapp.ws/html
```

(6) Read access is permitted to generally all users.

```bash
sudo chmod -R 755 /var/www
```

(7) Open sample file.

```bash
vi /var/www/nwapp.ws/html/index.html
```

(8) Append file:

```html
<html>
  <head>
    <title>Welcome to Mikaponics.com!</title>
  </head>
  <body>
    <h1>Success! The nwapp.ws server block is working!</h1>
  </body>
</html>
```

(9) Create new server blocks.

```bash
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled
```

(10) Create the first server block file.

```bash
sudo vi /etc/nginx/sites-available/nwapp.ws.conf
```

(11) Append file:

```
server {
    listen  80;

    server_name *.nwapp.ws;

    location / {
        root  /var/www/nwapp.ws/html;
        index  index.html index.htm;
        try_files $uri $uri/ =404;
    }

    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root  /usr/share/nginx/html;
    }
}
```

(12) Enable the new server block files.

```bash
sudo ln -s /etc/nginx/sites-available/nwapp.ws.conf /etc/nginx/sites-enabled/nwapp.ws.conf
```

(13) Restart the server.

```bash
sudo systemctl restart nginx
```

(14) Because we are running ``CentOS 7`` we need to run the following because of [this article](https://stackoverflow.com/a/31403848) and do not skip this:

```bash
$ sudo setsebool -P httpd_can_network_connect 1
```

(15) Because of the security policies of the SELinux, we need to manually add the httpd_t to the list of permissive domains, run this command:

```bash
$ sudo semanage permissive -a httpd_t
```

(16) Also please run this command because of [this article](https://stackoverflow.com/a/26228135) and do not skip this step:

```
$ sudo chcon -Rt httpd_sys_content_t /var/www/nwapp.ws/build
```

(17) Finally in your browser, check to see that the links work.

```
http://nwapp.ws
http://london.nwapp.ws
http://toronto.nwapp.ws
```
