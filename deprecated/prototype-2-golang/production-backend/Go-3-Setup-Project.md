1. While being logged in as ``techops`` run the following:

    ```
    $ sudo -i -u postgres
    $ psql
    ```


2. Then run the following to support our web-applications database.

    ```sql
    drop database nwapp_golang_db;
    create database nwapp_golang_db;
    \c nwapp_golang_db;
    CREATE USER nwatchcanada WITH PASSWORD '123password';
    GRANT ALL PRIVILEGES ON DATABASE nwapp_golang_db to nwatchcanada;
    ALTER USER nwatchcanada CREATEDB;
    ALTER ROLE nwatchcanada SUPERUSER;
    CREATE EXTENSION postgis;
    ```


3. And re-run to support our database used for unit testing.

    ```sql
    drop database nwapp_golang_test_db;
    create database nwapp_golang_test_db;
    \c nwapp_golang_test_db;
    CREATE USER nwatchcanada WITH PASSWORD '123password';
    GRANT ALL PRIVILEGES ON DATABASE nwapp_golang_test_db to nwatchcanada;
    ALTER USER nwatchcanada CREATEDB;
    ALTER ROLE nwatchcanada SUPERUSER;
    CREATE EXTENSION postgis;
    ```


### Setup Web-App from GitHub
1. Please run the following commands as the ``nwatchcanada`` user account.

    ```
    su - nwatchcanada
    ```

1. Get the project.

    ```
    $ mkdir /opt/nwatchcanada/go/src/github.com/
    $ cd /opt/nwatchcanada/go/src/github.com/
    $ go get github.com/nwatchcanada/nwapp-back
    ```


2. Install the dependencies.

    ```
    $ cd /opt/nwatchcanada/go/src/github.com/nwatchcanada/nwapp-back;
    $ ./requirements.sh;
    ```


3. Build our project.

   ```
   $ go install github.com/nwatchcanada/nwapp-back
   ```

4. Populate our environment variables in our system.

    ```bash
    #!/bin/bash
    export NWAPP_DB_HOST="localhost"
    export NWAPP_DB_PORT="5432"
    export NWAPP_DB_USER="nwatchcanada"
    export NWAPP_DB_PASSWORD="123password"
    export NWAPP_DB_NAME="nwapp_golang_db"
    export NWAPP_APP_ADDRESS="127.0.0.1:8000"
    ```

4. The next step involves you leaving the ``nwatchcanada`` user and stay as ``techops`` user.

    ```
    exit
    su - techops
    ```

5. Enable permission and security while you are a ``techops`` user.

    ```
    $ sudo setcap 'cap_net_bind_service=+ep' /opt/nwatchcanada/go/bin/nwapp-back
    $ sudo setsebool -P httpd_can_network_connect 1
    $ sudo semanage permissive -a httpd_t
    //$ sudo chcon -Rt httpd_sys_content_t /opt/golang/workery-golang/workery/static    (DEPRECATED)
    ```


### Integrate Nginx with Golang
Please run the following commands as the ``techops`` user account.

1. Load up ``Nginx``.

   ```
   $ sudo vi /etc/nginx/nginx.conf
   ```


2. Replace with the following code.

    ```
    server {
        listen       80;
        server_name  nwapp.ws;

        charset utf-8;

        location / {
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header X-Real-IP       $remote_addr;
            proxy_set_header Host            $http_host;

            proxy_pass http://127.0.0.1:8000;
        }
    }
    ```


3. Restart ``Nginx``.

    ```
    sudo systemctl restart nginx
    ```


4. Run your go app manually.

    ```
    # sudo su - nwatchcanada
    # $GOBIN/nwapp-back
    ```


5. Now in your browser go to ``http://nwapp.ws`` and you should see the app!


6. Special thanks to:

  * https://beego.me/docs/deploy/nginx.md


### Integrate Systemd with Golang

This section explains how to integrate our project with ``systemd`` so our operating system will handle stopping, restarting or starting.

1. **(OPTIONAL)** If you cannot access the server, please stop and review the steps above. If everything is working proceed forward.


2. While you are logged in as a ``techops`` user, please write the following into the console.

    ```
    $ sudo vi /etc/systemd/system/nwapp-back.service
    ```


3. Copy and paste the following. **Please change the variables to meet your own.** Please do not change the IP``127.0.0.1:8080``. Note: ``Nginx`` will be communicating with the ``golang`` app through this IP.

    ```
    Description=Neighbourhood Watch App Backend
    Wants=network.target
    After=network.target

    [Service]
    Environment=NWAPP_DB_HOST=localhost
    Environment=NWAPP_DB_PORT=5432
    Environment=NWAPP_DB_USER=nwatchcanada
    Environment=NWAPP_DB_PASSWORD=123password
    Environment=NWAPP_DB_NAME=nwapp_golang_db
    Environment=NWAPP_APP_ADDRESS=127.0.0.1:8000
    Type=simple
    DynamicUser=yes
    WorkingDirectory=/opt/nwatchcanada/go/bin
    ExecStartPre=NWAPP_DB_HOST
    ExecStartPre=NWAPP_DB_PORT
    ExecStartPre=NWAPP_DB_USER
    ExecStartPre=NWAPP_DB_PASSWORD
    ExecStartPre=NWAPP_DB_NAME
    ExecStartPre=NWAPP_APP_ADDRESS
    ExecStart=/opt/nwatchcanada/go/bin/nwapp-back
    Restart=always
    RestartSec=3
    SyslogIdentifier=nwapp-back

    [Install]
    WantedBy=multi-user.target
    ```


4. Grant access.

   ```
   $ sudo chmod 755 /etc/systemd/system/nwapp-back.service
   ```


5. (Optional) If you've updated the above, you will need to run the following before proceeding.

    ```
    $ sudo systemctl daemon-reload
    ```


6. We can now start the Gunicorn service we created and enable it so that it starts at boot:

    ```
    $ sudo systemctl start nwapp-back
    $ sudo systemctl enable nwapp-back
    ```


7. Confirm our service is running.

    ```
    $ systemctl status nwapp-back.service
    $ journalctl -f -u nwapp-back.service
    ```


8. And verify the URL works in the browser.

    ```text
    http://nwapp.ws/version
    ```

### Finalize Project Setup

1. Make sure you login as the ``nwatchcanada`` user:

```
$ sudo su - nwatchcanada
```

2. Add our tenant.

```
$ $GOBIN/nwapp-back add_tenant "london" "Neighbourhood Watch Canada London"
```

3. Add our user. (**Please change ``xxx`` password to your own!**)

```
$ $GOBIN/nwapp-back add_user "bart@mikasoftware.com" "Bart" "Mika" "XXX" 1 "london" 1
```

4. And verify the URL works in the browser.

    ```text
    http://nwapp.ws/version
    ```
