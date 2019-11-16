# Setup NWApp Web-Application Backend
The following instructions will set up the [nwapp-back](https://github.com/nwapp/nwapp-back) code. Please note, you must complete the [previous article](/2-Virtual-Hosts.md) before proceeding further.

## Instructions

### Setup the Django User

1. Create a services user for the application:

    ```
    sudo groupadd --system django;
    sudo useradd --system --gid django --shell /bin/bash --home /opt/django django;

2. Create the Django project home inside ``/opt``:

    ```
    sudo mkdir /opt/django
    ```

3. Give the permissions to the ``django`` user:

    ```
    sudo chown django:django /opt/django
    ```

4. Go into our new user. Note: ``sudo`` is to elevate privilege and ``su`` to switch users.

    ```
    $ sudo su - django
    ```

### Setup the project
#### Setup from GitHub

1. Clone the project.

    ```
    $ cd ~/
    $ git clone https://github.com/nwatchcanada/nwapp-back.git
    $ cd ~/nwapp-back
    ```


2. Setup our virtual environment

    ```
    $ virtualenv -p python3.6 env
    ```


3. Now lets activate virtual environment

    ```
    $ source env/bin/activate
    ```


4. Now lets install the libraries this project depends on.

    ```
    (env)$ pip install -r requirements.txt
    ```


#### Database Setup

1. Log into the ``techops`` user account.

2. Go to your ``postgres`` account.

    ```
    $ sudo -i -u postgres
    $ psql
    ```

3. Enter the following:

    ```sql
    drop database nwapp_db;
    create database nwapp_db;
    \c nwapp_db;
    CREATE USER django WITH PASSWORD '123password';
    GRANT ALL PRIVILEGES ON DATABASE nwapp_db to django;
    ALTER USER django CREATEDB;
    ALTER ROLE django SUPERUSER;
    CREATE EXTENSION postgis;
    ```


#### Environment Variables Setup

1. Populate the environment variables for our project.

    ```
    (env)$ cd ~/nwapp-back
    (env)$ ./setup_credentials.sh
    ```

2. Go inside the environment variables.

    ```
    (env)$ vi /opt/django/nwapp-back/nwapp/nwapp/.env
    ```

3. Edit the file to suite your needs.


#### Setup the Application

1. Run the following commands to populate the database.

    ```
    cd ~/nwapp-back/nwapp; \
    python manage.py makemigrations; \
    python manage.py migrate; \
    python manage.py init_app; \
    python manage.py setup_resource_server_authorization; \
    python manage.py collectstatic;
    ```


### DigtalOcean Nameserver

1. Go and log into [DigitalOcean](https://digitalocean.com).

2. Go to **Networking** section.

3. Enter the domain and add it. Note: Please note that the URL used here is different.

4. Add ``A record``  where the hostname points to our droplet.

5. Add ``CNAME`` where the hostname is ``www`` and the alias is ``@``.

6. Add ``CNAME`` where the hostname is ``*`` and the alias is ``@``.


### Confirm Redis Operational Status

1. Confirm that the ``redis`` service has no errors by running:

    ```
    $ sudo systemctl status redis
    ```

2. Confirm we can connect to the ``redis`` server with the command-line client:

    ```
    $ redis-cli
    ```

3. Confirm connectivity by typing into the console:

    ```
    ping
    ```

4. You should see the following output:

    ```
    PONG
    ```

### Setup Nginx with Project

1. (OPTIONAL) Open up our ``Nginx`` configuration file:

    ```
    vi /etc/nginx/nginx.conf
    ```

2. (OPTIONAL) Delete the following code.

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

3. (OPTIONAL) Add the following code.

    ```
    server {
        listen 80 default_server;
        deny all;
    }
    ```

4. Create the first server block file.

    ```
    sudo vi /etc/nginx/sites-available/nwapp.ws.conf
    ```

5. Replace the context of the file with the following:

    ```
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
    }
    ```

6. Enable the new server block files.

    ```
    sudo ln -s /etc/nginx/sites-available/nwapp.ws.conf /etc/nginx/sites-enabled/nwapp.ws.conf
    ```

5. (Optional) The following article should be read if you are setting up ``Django REST Framework`` with your project:

    ```url
    https://stackoverflow.com/a/34233258
    ```

6. Test your ``nginx`` configuration for syntax errors by typing:

    ```
    $ sudo nginx -t
    ```

7. If any errors occur please investigate and fix before proceeding further. Else if no errors are reported, go ahead and restart ``nginx`` by typing:

    ```
    $ sudo systemctl restart nginx
    ```

8. Because we are running ``CentOS 7`` we need to run the following because of [this article](https://stackoverflow.com/a/31403848) and do not skip this:

    ```
    $ sudo setsebool -P httpd_can_network_connect 1
    ```

9. Because of the security policies of the SELinux, we need to manually add the httpd_t to the list of permissive domains, run this command:

    ```
    $ sudo semanage permissive -a httpd_t
    ```

10. Also please run this command because of [this article](https://stackoverflow.com/a/26228135) and do not skip this step:

    ```
    $ sudo chcon -Rt httpd_sys_content_t /opt/django/nwapp-back/nwapp/static
    ```

### OPTION 1 of 2: Systemd Method
If you would like to get the project up and running simply by using ``systemd`` the follow the next series or steps.

#### Setup Gunicorn with Systemd

This section explains how to integrate our project with ``systemd`` so our operating system will handle stopping, restarting or starting.

1. (OPTIONAL) If you cannot access the server, please stop and review the steps above. If everything is working proceed forward.

2. While you are logged in as a ``techops`` user, please write the following into the console.

    ```
    $ sudo vi /etc/systemd/system/gunicorn.service
    ```

3. Implement

    ```
    [Unit]
    Description=gunicorn daemon
    After=network.target

    [Service]
    User=django
    Group=nginx
    Restart=always
    RestartSec=3
    WorkingDirectory=/opt/django/nwapp-back
    ExecStart=/opt/django/nwapp-back/env/bin/gunicorn -c gunicorn_config.py nwapp.wsgi

    [Install]
    WantedBy=multi-user.target
    ```

4. We can now start the Gunicorn service we created and enable it so that it starts at boot:

    ```
    $ sudo systemctl start gunicorn
    $ sudo systemctl enable gunicorn
    ```

5. Confirm our service is running.

    ```
    $ systemctl status gunicorn.service
    ```

6. And verify the URL works in the browser.

    ```text
    http://nwapp.ws/en/
    ```

7. Would you like to know more?

* [Create a Gunicorn Systemd Service File](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-centos-7#create-a-gunicorn-systemd-service-file).

* [How to make Python script run as service?](https://stackoverflow.com/a/16420472)


#### Setup Redis django_rq with Systemd

1. While you are logged in as a ``techops`` user, please write the following into the console.

    ```
    $ sudo vi /etc/systemd/system/django_rq.service
    ```

3. Implement

    ```
    [Unit]
    Description=Redis rqworker for Django App
    After=network.target

    [Service]
    Type=simple
    User=django
    Group=nginx
    Restart=always
    RestartSec=3
    WorkingDirectory=/opt/django/nwapp-back
    ExecStart=/opt/django/nwapp-back/django_rq.sh

    [Install]
    WantedBy=multi-user.target
    ```

4. We can now start the Gunicorn service we created and enable it so that it starts at boot:

    ```
    $ sudo systemctl start django_rq
    $ sudo systemctl enable django_rq
    ```

5. Confirm our service is running.

    ```
    $ systemctl status django_rq.service
    ```


#### Setup Redis rq_scheduler with Systemd

1. While you are logged in as a ``techops`` user, please write the following into the console.

    ```
    $ sudo vi /etc/systemd/system/rq_scheduler.service
    ```

2. Implement

    ```
    [Unit]
    Description=Redis rq_scheduler for Django App
    After=network.target

    [Service]
    Type=simple
    User=django
    Group=nginx
    Restart=always
    RestartSec=3
    WorkingDirectory=/opt/django/nwapp-back
    ExecStart=/opt/django/nwapp-back/rq_scheduler.sh

    [Install]
    WantedBy=multi-user.target
    ```

3. We can now start the Gunicorn service we created and enable it so that it starts at boot:

    ```
    $ sudo systemctl start rq_scheduler
    $ sudo systemctl enable rq_scheduler
    ```

4. Confirm our service is running.

    ```
    $ sudo systemctl status rq_scheduler.service
    ```

5. If you did **OPTION 1** then you will need to run the following:

    ```
    sudo systemctl disable gunicorn; \
    sudo systemctl disable django_rq; \
    sudo systemctl disable rq_scheduler; \
    sudo systemctl stop gunicorn; \
    sudo systemctl stop django_rq; \
    sudo systemctl stop rq_scheduler;
    ```

6. If you want to start up again, run:

    ```
    sudo systemctl enable gunicorn; \
    sudo systemctl enable django_rq; \
    sudo systemctl enable rq_scheduler; \
    sudo systemctl start gunicorn; \
    sudo systemctl start django_rq; \
    sudo systemctl start rq_scheduler;
    ```

7. If you want to start up again, run:

    ```
    sudo systemctl restart gunicorn; \
    sudo systemctl restart django_rq; \
    sudo systemctl restart rq_scheduler; \
    sudo systemctl restart nginx;
    ```

8. See the status

    ```
    sudo systemctl status gunicorn; \
    sudo systemctl status django_rq; \
    sudo systemctl status rq_scheduler;
    ```


### OPTION 2 of 2: Supervisor Method
If you would like to get the project up and running using ``supervisor`` then follow the next series of steps.

Unfortunately because of [issue 1060](https://github.com/Supervisor/supervisor/issues/1060) in ``supervisor``. We will skip this section until the issue has been fixed.
