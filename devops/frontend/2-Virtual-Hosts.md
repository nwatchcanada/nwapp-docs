# Setup Virtual Hosts for the NWApp Platform.
The following instructions where modified from [How To Set Up Nginx Server Blocks on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-on-centos-7). Before proceeding, please make sure you completed the [previous instructions](/1-DigitalOcean-Droplet.md) before proceeding.

1. Open up our ``Nginx`` configuration file:

    ```
    vi /etc/nginx/nginx.conf
    ```

2. Append to the end of the ``http {}`` code block:

    ```
    include /etc/nginx/sites-enabled/*.conf;
    server_names_hash_bucket_size 64;
    ```

3. **Delete** the following code block:

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

3. Create the directory structure.

    ```
    sudo mkdir -p /var/www/nwapp.ca/html
    ```

3. Grand permission to the regular users.

    ```
    sudo chown -R $USER:$USER /var/www/nwapp.ca/html
    ```

4. Read access is permitted to generally all users.

    ```
    sudo chmod -R 755 /var/www
    ```

5. Open sample file.

    ```
    vi /var/www/nwapp.ca/html/index.html
    ```

6. Append file:

    ```
    <html>
      <head>
        <title>Welcome to Mikaponics.com!</title>
      </head>
      <body>
        <h1>Success! The nwapp.ca server block is working!</h1>
      </body>
    </html>
    ```

13. Create new server blocks.

    ```
    sudo mkdir /etc/nginx/sites-available
    sudo mkdir /etc/nginx/sites-enabled
    ```


14. Create the first server block file.

    ```
    sudo vi /etc/nginx/sites-available/nwapp.ca.conf
    ```

15. Append file:

    ```
    server {
        listen  80;

        server_name *.nwapp.ca;

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
    ```

18. Enable the new server block files.

    ```
    sudo ln -s /etc/nginx/sites-available/nwapp.ca.conf /etc/nginx/sites-enabled/nwapp.ca.conf
    ```

19. Restart the server.

    ```
    sudo systemctl restart nginx
    ```

1. Because we are running ``CentOS 7`` we need to run the following because of [this article](https://stackoverflow.com/a/31403848) and do not skip this:

    ```
    $ sudo setsebool -P httpd_can_network_connect 1
    ```

2. Because of the security policies of the SELinux, we need to manually add the httpd_t to the list of permissive domains, run this command:

    ```
    $ sudo semanage permissive -a httpd_t
    ```

8. Also please run this command because of [this article](https://stackoverflow.com/a/26228135) and do not skip this step:

    ```
    $ sudo chcon -Rt httpd_sys_content_t /var/www/nwapp.ca/build
    ```

20. Finally in your browser, check to see that the links work.

    ```
    http://nwapp.ca
    http://london.nwapp.ca
    http://toronto.nwapp.ca
    ```
