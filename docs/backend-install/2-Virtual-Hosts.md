
# DigtalOcean Nameserver

1. Go and log into [DigitalOcean](https://digitalocean.com).

2. Go to **Networking** section.

3. Enter the domain and add it. Note: Please note that the URL used here is different.

4. Add ``A record``  where the hostname points to our droplet.

5. Add ``CNAME`` where the hostname is ``www`` and the alias is ``@``.

6. Add ``CNAME`` where the hostname is ``*`` and the alias is ``@``.


## Setup Sub-Hosts

1. Open up our ``Nginx`` configuration file:

        $ sudo vi /etc/nginx/nginx.conf

2. Append to the end of the ``http {}`` code block:

        include /etc/nginx/sites-enabled/*.conf;
        server_names_hash_bucket_size 64;

3. Create new server blocks.

        sudo mkdir /etc/nginx/sites-available
        sudo mkdir /etc/nginx/sites-enabled

### nwapp.ca
1. Create the directory structure.

        sudo mkdir -p /var/www/nwapp.ca/html

2. Grand permission to the regular users.

        sudo chown -R $USER:$USER /var/www/nwapp.ca/html

3. Read access is permitted to generally all users.

        sudo chmod -R 755 /var/www

4. Open sample file.

        vi /var/www/nwapp.ca/html/index.html

5. Append file:

        <html>
            <head>
                <title>Welcome to nwapp.ca!</title>
             </head>
             <body>
                <h1>Success! The nwapp.ca server block is working!</h1>
            </body>
        </html>

6. Create the first server block file.

        sudo vi /etc/nginx/sites-available/nwapp.ca.conf

7. Append file:

        server {
            listen  80;

            server_name 165.22.234.35 nwapp.ca www.nwapp.ca;

            location / {
                root  /var/www/nwapp.ca/html;
                index  index.html index.htm;
                try_files $uri $uri/ =404;
            }

            error_page  500 502 503 504  /50x.html;
            location = /50x.html {
                root  /usr/share/nginx/html;
            }
        }

8. Enable the new server block files.

        sudo ln -s /etc/nginx/sites-available/nwapp.ca.conf /etc/nginx/sites-enabled/nwapp.ca.conf;

9. Confirm our server can restart.

        sudo nginx -t;

10. Restart the server.

        sudo systemctl restart nginx

12. Update your nginx config

        sudo vi /etc/nginx/nginx.conf

13. The following can be deleted.

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
                ## enable php support ##
                location ~ \.php$ {
                root /usr/share/nginx/html;
                fastcgi_pass   127.0.0.1:9000;
                fastcgi_index  index.php;
                include        fastcgi_params;
                fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
                }
        }

14. Confirm our server can restart.

        sudo nginx -t;

15. Restart the server.

        sudo systemctl restart nginx

16. Because we are running ``CentOS 7`` we need to run the following because of [this article](https://stackoverflow.com/a/31403848) and do not skip this:

        sudo setsebool -P httpd_can_network_connect 1

17. Because of the security policies of the SELinux, we need to manually add the httpd_t to the list of permissive domains, run this command:

        sudo semanage permissive -a httpd_t


(16) Also please run this command because of [this article](https://stackoverflow.com/a/26228135) and do not skip this step:

        sudo chcon -Rt httpd_sys_content_t /var/www/nwapp.ca/build


(17) Finally in your browser, check to see that the links work.

```
http://nwapp.ca
http://london.nwapp.ca
http://toronto.nwapp.ca
```

### nwapp.ws
1. Create the directory structure.

        sudo mkdir -p /var/www/nwapp.ws/html

2. Grand permission to the regular users.

        sudo chown -R $USER:$USER /var/www/nwapp.ws/html

3. Read access is permitted to generally all users.

        sudo chmod -R 755 /var/www

4. Open sample file.

        vi /var/www/nwapp.ws/html/index.html

5. Append file:

        <html>
            <head>
                <title>Welcome to nwapp.ws!</title>
             </head>
             <body>
                <h1>Success! The nwapp.ws server block is working!</h1>
            </body>
        </html>

6. Create the first server block file.

        sudo vi /etc/nginx/sites-available/nwapp.ws.conf

7. Append file:

        server {
            listen  80;

            server_name 165.22.234.35 nwapp.ws www.nwapp.ws;

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

8. Enable the new server block files.

        sudo ln -s /etc/nginx/sites-available/nwapp.ws.conf /etc/nginx/sites-enabled/nwapp.ws.conf;

9. Confirm our server can restart.

        sudo nginx -t;

10. Restart the server.

        sudo systemctl restart nginx

(17) Finally in your browser, check to see that the links work.

```
http://nwapp.ws
http://london.nwapp.ws
http://toronto.nwapp.ws
```
