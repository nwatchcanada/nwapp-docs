The following instructions will set up the [nwapp-back](https://github.com/nwapp/nwapp-back) code. Please note, you must complete the [previous article](/2-Virtual-Hosts.md) before proceeding further.

# Setup the Django User

(1) Create a services user for the application:

```bash
$ sudo groupadd --system django;
$ sudo useradd --system --gid django --shell /bin/bash --home /opt/django django;
```

(2) Create the Django project home inside ``/opt``:

```bash
$ sudo mkdir /opt/django
```

(3) Give the permissions to the ``django`` user:

```bash
$ sudo chown django:django /opt/django
```

(4) Go into our new user. Note: ``sudo`` is to elevate privilege and ``su`` to switch users.

```
$ sudo su - django
```

# OPTIONAL: Support SSH Key

(1) (Optional) **On your local developers machine**, generate the ``ssh`` key pair values. When prompted about passphase, skip it. After generating the key, print the public keys to the console.

```bash
local$ ssh-keygen
local$ cat ~/.ssh/id_rsa.pub
```

(2) Now on your server, copy and paste your ssh key pair into the new user.

```bash
# su - django
$ mkdir .ssh
$ chmod 700 .ssh
$ vi .ssh/authorized_keys
```

(3) Restrict the permissions of the *authorized_keys* file with this command. Do not skip this command as you will be unable to ``ssh`` into this server without setting the permissions here:

```bash
$ chmod 600 .ssh/authorized_keys
```

(4) On your local developers machine, attempt to log into the server with the ``techops`` user account to confirm it is working. If you cannot log in then please review steps 1 to 7 or search online for answers. Here is an example:

```bash
local$ ssh -l django 165.22.234.35
```


# Setup the project
## Setup from GitHub

(1) Clone the project.

```bash
$ cd ~/
$ git clone https://github.com/nwatchcanada/nwapp-back.git
$ cd ~/nwapp-back
```

(2) Setup our virtual environment

```bash
$ virtualenv -p python3.7 env
```

(3) Now lets activate virtual environment

```bash
$ source env/bin/activate
```

(4) We will need to build ``psycopg2`` library before proceeding to install our application's `requirements.txt` file. Special thanks to [this article](https://community.webfaction.com/questions/7714/installing-psycopg2-pg_config-missing/7721).

```bash
(env)$ export PATH=/usr/pgsql-12/bin/:$PATH
(env)$ pip install psycopg2
```


(5) We will need to build ``gdal`` library before proceeding to install our application.

```bash
(env)$ export PATH=/usr/gdal30/bin/:$PATH
(env)$ pip install gdal
```


(6) Finally - Let us install the libraries this project depends on without dealing
with `psycopg2` and `gdal` giving us problems:

```bash
(env)$ pip install -r requirements.txt
```

## GeoIP2 Setup

1. You *must* download the GZIP binary files ``GeoLite2 City`` and ``GeoLite2 Country`` from [this link](https://dev.maxmind.com/geoip/geoip2/geolite2/) on your local computer. Please unzip those files in the ``./nwapp/geoip`` folder so you should have the following structure locally:

* ``nwapp-back/nwapp/geoip/GeoLite2-City.mmdb``
* ``nwapp-back/nwapp/geoip/GeoLite2-Country.mmdb``

2. Now we need to upload our files to our remote server.

```
cd /Users/bmika/python/github.com/nwatchcanada/nwapp-back/nwapp/geoip;
scp GeoLite2-City.mmdb django@165.22.234.35:/opt/django/nwapp-back/nwapp/geoip;
scp GeoLite2-Country.mmdb django@165.22.234.35:/opt/django/nwapp-back/nwapp/geoip;
```

## Database Setup

(1) Log into the ``techops`` user account.

(2) Go to your ``postgres`` account.

```bash
$ sudo -i -u postgres
$ psql
```

(3) Enter the following:

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

(4) If there are no errors then congradulations!


## Environment Variables Setup

(1) Populate the environment variables for our project.

```bash
(env)$ cd ~/nwapp-back
(env)$ ./setup_credentials.sh
```

(2) Go inside the environment variables.

```bash
(env)$ vi /opt/django/nwapp-back/nwapp/nwapp/.env
```

(3) Edit the file to suite your needs.

(DO NOT SKIP: Temporary BUGFIX for messagepack library)
cd /opt/django/nwapp-back/env/lib/python3.7/site-packages/rest_framework_msgpack
vi parsers.py

Change `from django.utils.six import text_type` to ``.

## Setup the Application

(1) Run the following commands to populate the database.

```
cd ~/nwapp-back/nwapp; \
redis-cli FLUSHDB; \
source ../env/bin/activate; \
python manage.py makemigrations; \
python manage.py migrate_schemas --executor=multiprocessing; \
python manage.py init_app; \
python manage.py setup_oauth2; \
python manage.py create_shared_user "bart@mikasoftware.com" "123password" "Bart" "Mika";
```

### Confirm Redis Operational Status

(1) Confirm that the ``redis`` service has no errors by running:

```bash
$ sudo systemctl status redis
```

(2) Confirm we can connect to the ``redis`` server with the command-line client:

```
$ redis-cli
```

(3) Confirm connectivity by typing into the console:

```
ping
```

(4) You should see the following output:

```
PONG
```

### Setup Nginx with Project

(4) Create the first server block file.

```bash
sudo vi /etc/nginx/sites-available/nwapp.ws.conf
```

(5) Replace the context of the file with the following:

```
server {
    listen 80;

    server_name *.nwapp.ws;

    charset     utf-8;
    access_log off;
    gzip on;
    client_max_body_size 0;  # Unlimited Upload File Size

    # Optional if you want logging in Nging.
    access_log /var/log/nginx/nwapp-default-access.log;
    error_log /var/log/nginx/nwapp-default-error.log;

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
        proxy_connect_timeout       900;
        proxy_send_timeout          900;
        proxy_read_timeout          900;
        send_timeout                900;
    }
}
```

(7) (Optional) The following article should be read if you are setting up ``Django REST Framework`` with your project:

```url
https://stackoverflow.com/a/34233258
```

(8) Test your ``nginx`` configuration for syntax errors by typing:

```bash
$ sudo nginx -t
```

(9) If any errors occur please investigate and fix before proceeding further. Else if no errors are reported, go ahead and restart ``nginx`` by typing:

```bash
$ sudo systemctl restart nginx
```

(10) Because we are running ``CentOS 7`` we need to run the following because of [this article](https://stackoverflow.com/a/31403848) and do not skip this:

```bash
$ sudo setsebool -P httpd_can_network_connect 1
```

(11) Because of the security policies of the SELinux, we need to manually add the httpd_t to the list of permissive domains, run this command:

```bash
$ sudo semanage permissive -a httpd_t
```

(12) Also please run this command because of [this article](https://stackoverflow.com/a/26228135) and do not skip this step:

```
$ sudo chcon -Rt httpd_sys_content_t /opt/django/nwapp-back/nwapp/static
```

### OPTION 1 of 2: Systemd Method
If you would like to get the project up and running simply by using ``systemd`` the follow the next series or steps.

#### Setup Gunicorn with Systemd

This section explains how to integrate our project with ``systemd`` so our operating system will handle stopping, restarting or starting.

(1) (OPTIONAL) If you cannot access the server, please stop and review the steps above. If everything is working proceed forward.

(2) While you are logged in as a ``techops`` user, please write the following into the console.

```bash
$ sudo vi /etc/systemd/system/nwapp_gunicorn.service
```

(3) Implement

```
[Unit]
Description=nwapp gunicorn daemon
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

(4) We can now start the Gunicorn service we created and enable it so that it starts at boot:

```bash
$ sudo systemctl start nwapp_gunicorn
$ sudo systemctl enable nwapp_gunicorn
```

(5) Confirm our service is running.

```bash
$ systemctl status nwapp_gunicorn.service
```

6. And verify the URL works in the browser.

```text
http://nwapp.ws/en/
```

7. Would you like to know more?

* [Create a Gunicorn Systemd Service File](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-centos-7#create-a-gunicorn-systemd-service-file).

* [How to make Python script run as service?](https://stackoverflow.com/a/16420472)


#### Setup Redis django_rq with Systemd

(1) While you are logged in as a ``techops`` user, please write the following into the console.

```bash
$ sudo vi /etc/systemd/system/nwapp_django_rq.service
```

(2) Implement

```
[Unit]
Description=Redis rqworker for NWApp Django App
After=network.target

[Service]
Type=simple
User=django
Group=nginx
Restart=always
RestartSec=3
WorkingDirectory=/opt/django/nwapp-back
ExecStart=/opt/django/nwapp-back/scripts/django_rq.sh

[Install]
WantedBy=multi-user.target
```

(4) We can now start the Gunicorn service we created and enable it so that it starts at boot:

```bash
$ sudo systemctl start nwapp_django_rq
$ sudo systemctl enable nwapp_django_rq
```

(5) Confirm our service is running.

```bash
$ systemctl status nwapp_django_rq.service
```


#### Setup Redis rq_scheduler with Systemd

(1) While you are logged in as a ``techops`` user, please write the following into the console.

```bash
$ sudo vi /etc/systemd/system/nwapp_rq_scheduler.service
```

(2) Implement

```
[Unit]
Description=Redis rq_scheduler for NWApp Django App
After=network.target

[Service]
Type=simple
User=django
Group=nginx
Restart=always
RestartSec=3
WorkingDirectory=/opt/django/nwapp-back
ExecStart=/opt/django/nwapp-back/scripts/rq_scheduler.sh

[Install]
WantedBy=multi-user.target
```

(3) We can now start the Gunicorn service we created and enable it so that it starts at boot:

```bash
$ sudo systemctl start nwapp_rq_scheduler
$ sudo systemctl enable nwapp_rq_scheduler
```

(4) Confirm our service is running.

```bash
$ sudo systemctl status nwapp_rq_scheduler.service
```

(5) If you did **OPTION 1** then you will need to run the following:

```bash
sudo systemctl disable nwapp_gunicorn; \
sudo systemctl disable nwapp_django_rq; \
sudo systemctl disable nwapp_rq_scheduler; \
sudo systemctl stop nwapp_gunicorn; \
sudo systemctl stop nwapp_django_rq; \
sudo systemctl stop nwapp_rq_scheduler;
```

(6) If you want to start up again, run:

```bash
sudo systemctl enable nwapp_gunicorn; \
sudo systemctl enable nwapp_django_rq; \
sudo systemctl enable nwapp_rq_scheduler; \
sudo systemctl start nwapp_gunicorn; \
sudo systemctl start nwapp_django_rq; \
sudo systemctl start nwapp_rq_scheduler;
```

(7) If you want to start up again, run:

```bash
sudo systemctl restart nwapp_gunicorn; \
sudo systemctl restart nwapp_django_rq; \
sudo systemctl restart nwapp_rq_scheduler; \
sudo systemctl restart nginx;
```

(8) See the status

```
sudo systemctl status nwapp_gunicorn; \
sudo systemctl status nwapp_django_rq; \
sudo systemctl status nwapp_rq_scheduler;
```


### OPTION 2 of 2: Supervisor Method
If you would like to get the project up and running using ``supervisor`` then follow the next series of steps.

Unfortunately because of [issue 1060](https://github.com/Supervisor/supervisor/issues/1060) in ``supervisor``. We will skip this section until the issue has been fixed.
