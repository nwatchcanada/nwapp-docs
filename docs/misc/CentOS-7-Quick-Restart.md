# Quickly Restarting the Project
## Description
The goal of this article is to provide instructions on how to quickly format the project and restart it, if you have previously installed it successfully. **Please pick different passwords then here**.

## Backend Instructions
### Stop Running Services
Please run the following:

  ```bash
  sudo systemctl stop gunicorn; \
  sudo systemctl stop django_rq; \
  sudo systemctl stop rq_scheduler;
  ```


### Database
While being logged in as ``root`` or ``techops`` please write the following in your console.

  ```bash
  sudo -i -u postgres;
  dropdb nwapp_db;
  createdb nwapp_db;
  psql nwapp_db;
  ```

  ```sql
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
  su - django
  cd nwapp-back
  source env/bin/activate
  cd nwapp
  ```

Just copy and paste this into your command console.

```bash
redis-cli FLUSHDB;
python manage.py makemigrations; \
python manage.py migrate_schemas --executor=multiprocessing; \
python manage.py init_app; \
python manage.py setup_oauth2;
```

Optional, but **please change the password** when you are using in **production** environment:

```bash
python manage.py create_shared_user "bart@mikasoftware.com" "123password" "Bart" "Mika";
```

### Resume Services
Please run the following:

  ```bash
  sudo systemctl start gunicorn; \
  sudo systemctl start django_rq; \
  sudo systemctl start rq_scheduler;
  ```

## Frontend Instructions
While being logged in as ``root`` or ``techops`` please write the following in your console.

```bash
cd /var/www/nwapp.ca; \
git pull origin master; \
npm install; \
sudo npm run build; \
sudo systemctl restart nginx;
```
