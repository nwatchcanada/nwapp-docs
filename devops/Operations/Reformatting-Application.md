# Reformatting Application

## Django

1. Stop the current ``Django`` services used.

    ```bash
    sudo systemctl stop gunicorn; \
    sudo systemctl stop django_rq; \
    sudo systemctl stop rq_scheduler;
    ```

2. Go inside postgres.

    ```bash
    su - postgres
    psql
    ```

3. Drop and re-create the database.

    ```sql
    drop database mikaponics_db;
    create database mikaponics_db;
    \c mikaponics_db;
    CREATE USER django WITH PASSWORD '123password';
    GRANT ALL PRIVILEGES ON DATABASE mikaponics_db to django;
    ALTER USER django CREATEDB;
    ALTER ROLE django SUPERUSER;
    CREATE EXTENSION postgis;
    ```

4. Exit the database command.

    ```
    CTRL + D
    CTRL + D
    ```

5. Go to the ``django`` article.

    ```bash
    su - django
    ```

6. Go to the project.

    ```bash
    cd /opt/django/mikaponics-back
    ```

7. Activate our virtual environment.

    ```bash
    source env/bin/activate
    ```

8. Fetch the latest data.

    ```bash
    git pull origin master
    ```

9. Start up the project again. **Please change the password to your own.**

    ```bash
    cd mikaponics;
    python manage.py migrate;
    python manage.py init_mikaponics;
    python manage.py init_crop_data_sheets
    python manage.py create_admin_user "bart@mikasoftware.com" "123password" "Bart" "Mika";
    python manage.py setup_resource_server_authorization;
    ```

10. OPTIONAL: If you would like to have some random test data to a specific user, feel free to run:

    ```bash
    python manage.py seed_user "bart@mikasoftware.com" 1 5000
    ```

11. Exit the ``django`` service account.

    ```
    CTRL + D
    ```

12. Finally start up the ``django`` services.

    ```bash
    sudo systemctl start gunicorn; \
    sudo systemctl start django_rq; \
    sudo systemctl start rq_scheduler;
    ```

13. OPTIONAL: Check to see if any errors occured.

   ```bash
   sudo systemctl status gunicorn; \
   sudo systemctl status django_rq; \
   sudo systemctl status rq_scheduler;
   ```

## React

Run the following code.

```
cd /var/www/app.mikaponics.com; \
git pull origin master; \
npm install; \
sudo npm run build; \
sudo systemctl restart nginx;
```
