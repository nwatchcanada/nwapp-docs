# Overview
The backend web-service is written in the ``Golang` programming language` and is responsible for providing **API endpoints** for the frontend to consume.

The project repository is [nwapp-back](https://github.com/nwatchcanada/nwapp-back) and it is open-source.

# Instructions

1. Before beginning please make sure the following requirements are met:

    * You have installed [``Golang``](https://golang.org/) on your machine.
    * You have properly setup the [installed and setup the environment](https://golangbot.com/golang-tutorial-part-1-introduction-and-installation/).
    * You are beginner in ``Golang`` and [you are willing to learn more](https://golangbot.com).
    * **Please do not continue until all the above points have been met!**

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
        CREATE USER nwatchcanada WITH PASSWORD '123password';
        GRANT ALL PRIVILEGES ON DATABASE nwapp_golang_db to nwatchcanada;
        ALTER USER nwatchcanada CREATEDB;
        ALTER ROLE nwatchcanada SUPERUSER;
        CREATE EXTENSION postgis;

6. Next we need to set the **environment variables** of our application. Our application is setup to take environment variables from the operating system. To begin, copy and paste the following and edit to suite your needs.

        export NWAPP_DB_HOST="localhost"
        export NWAPP_DB_PORT="5432"
        export NWAPP_DB_USER="nwatchcanada"
        export NWAPP_DB_PASSWORD="123password"
        export NWAPP_DB_NAME="nwapp_golang_db"
        export NWAPP_APP_ADDRESS="127.0.0.1:8000"

8. Finally confirm the web-application works by running the server.

        $ go run main.go

9. And now in a new tab in your favourite web-browser, visit [127.0.0.1:8000](http://27.0.0.1:8000).

# Notes

* When you restart your developer machine, the environment variables will be cleared, you will need to re-run exporting them again. If you want to avoid it, do the following:

    1. Run the following command:

        vi ~/.bash_profile

    2. Copy and paste your environment variables (see above).

    3. Set our profile.

        source ~/.bash_profile
