# Quickly Restarting the Project
## Description
The goal of this article is to provide instructions on how to quickly format the project and restart it, if you have previously installed it successfully. **Please pick different passwords then here**.

## Backend Instructions
### Stop Running Services
Please run the following:

  ```bash
  sudo systemctl stop nwapp_gunicorn; \
  sudo systemctl stop nwapp_django_rq; \
  sudo systemctl stop nwapp_rq_scheduler;
  ```

Confirm status:

  ```bash
  sudo systemctl status nwapp_gunicorn; \
  sudo systemctl status nwapp_django_rq; \
  sudo systemctl status nwapp_rq_scheduler;
  ```

### Database
While being logged in as ``root`` or ``techops`` please write the following in your console.

```bash
$ sudo -i -u postgres
$ psql
```

Enter the following:

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

Once the code above was written, please exit and and be logged in as either ``root`` or ``techops``.


### Restart Script
While being logged in as ``root`` or ``techops`` please write the following in your console.

  ```
  sudo su - django
  cd nwapp-back
  source env/bin/activate
  git pull origin master
  cd nwapp
  ```

Just copy and paste this into your command console.

```bash
redis-cli FLUSHDB; \
python manage.py makemigrations; \
python manage.py migrate_schemas --executor=multiprocessing; \
python manage.py init_app; \
python manage.py setup_oauth2;
```

If you would like to seed the database with random data then please run the following:

```bash
python manage.py create_shared_organization london \
       "Neighbourhood Watch London" \
       "NWatch App" \
       "This is our main tenant organization" \
       "Canada" \
       "London" \
       "Ontario" \
       "200" \
       "Centre" \
       "23" \
       "1" \
       "" \
       "" \
       "N6J4X4" \
       "America/Toronto" \
       "https://www.coplogic.ca/dors/en/filing/selectincidenttype?dynparam=1584326750929" \
       "42.983611" \
       "-81.249722" \
       "13";

python manage.py run_historic_csv_import_for_districts "london" "prod";
python manage.py run_historic_csv_import_for_watches "london" "prod";
python manage.py run_historic_csv_import_for_members "london" "prod";
```

Optional, but **please change the password** when you are using in **production** environment:

```bash
python manage.py create_shared_user "bart@mikasoftware.com" "123password" "Bart" "Mika";
```


Randomizer

```
python manage.py create_random_district "london" 50;
python manage.py create_random_watch "london" 500;
python manage.py create_random_member "london" 1000;
python manage.py create_random_area_coordinator "london" 100;
python manage.py create_random_associate "london" 100;
python manage.py create_random_task_item "london" 250;
```

### Resume Services
Please run the following:

  ```bash
  sudo systemctl start nwapp_gunicorn; \
  sudo systemctl start nwapp_django_rq; \
  sudo systemctl start nwapp_rq_scheduler;
  ```

Confirm status:

  ```bash
  sudo systemctl status nwapp_gunicorn; \
  sudo systemctl status nwapp_django_rq; \
  sudo systemctl status nwapp_rq_scheduler;
  ```

### Restart Services
Please run the following:

  ```bash
  sudo systemctl restart nwapp_gunicorn; \
  sudo systemctl restart nwapp_django_rq; \
  sudo systemctl restart nwapp_rq_scheduler;
  ```

Confirm status:

  ```bash
  sudo systemctl status nwapp_gunicorn; \
  sudo systemctl status nwapp_django_rq; \
  sudo systemctl status nwapp_rq_scheduler;
  ```

## Frontend Instructions
While being logged in as ``root`` or ``techops`` please write the following in your console.

  ```bash
  cd /var/www/nwapp.ca; \
  sudo git pull origin master; \
  sudo npm install; \
  sudo npm run build; \
  sudo systemctl restart nginx;
  ```
