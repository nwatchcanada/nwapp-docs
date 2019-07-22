# Overview
The backend web-service is written in the ``Golang` programming language` and is responsible for providing **API endpoints** for the frontend to consume.

The project repository is [nwapp-back](https://github.com/nwatchcanada/nwapp-back) and it is open-source.

# Instructions

1. Before beginning please make sure the following requirements are met:

    * You have installed [``Golang``](https://golang.org/) on your machine.
    * You have properly setup the [installed and setup the environment](https://golangbot.com/golang-tutorial-part-1-introduction-and-installation/).
    * You are beginner in ``Golang`` and [you are willing to learn more](https://golangbot.com).

2. Clone the project to your development machine in our namespace.

        $ mkdir ~/go/src/github.com/nwatchcanada
        $ cd  ~/go/src/github.com/nwatchcanada
        $ git clone https://github.com/nwatchcanada/nwapp-back.git

3. Open up your ``command`` console and install our dependent libraries:

        $ ./requirements.sh

4. (Optional) If you get any errors pertaining to required permissions, just run the following, else skip.

        $ chmod u+x requirements.sh

5. Open up your ``Postgres`` console and run the following code:

        drop database nwapp_golang_db;
        create database nwapp_golang_db;
        \c nwapp_golang_db;
        CREATE USER django WITH PASSWORD '123password';
        GRANT ALL PRIVILEGES ON DATABASE nwapp_golang_db to django;
        ALTER USER django CREATEDB;
        ALTER ROLE django SUPERUSER;
        CREATE EXTENSION postgis;

6. Next we create a `.env` file and populate it with our applications required environment variable. Start by creating our ``.env`` file.

        $ cat > .env

7. Copy and paste the following, edit to suite your needs and if you would like to know more what each environment variable does then [click here]().

        DB_HOST=localhost
        DB_PORT=5432
        DB_USER=django
        DB_PASSWORD=123password
        DB_NAME=nwapp_golang_db
        APP_ADDRESS=127.0.0.1:8000

8. Finally confirm the web-application works by running the server.

        $ go run main.go

9. And now in a new tab in your favourite web-browser, visit [127.0.0.1:8000](http://27.0.0.1:8000).
