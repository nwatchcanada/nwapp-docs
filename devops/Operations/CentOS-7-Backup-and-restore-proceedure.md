# CentOS 7 Backup and Restore
The goal of this article is to help setup the various backup / restore solutions for this project.

## Manual Backup / Restore Procedure
### CentOS 7 QA / Production
While you are on the CentOS 7 server, please login as the administrator and then run the following in your console:

1. Go inside postgres.

  ```
  su - postgres
  ```

2. Export out database.

  ```
  pg_dump -U postgres -d workery_db > workery_db.backup
  ```

3. Exit postgres.

  ```
  exit
  ```

4. Move our export to our ``django`` user.

  ```
  mv /var/lib/pgsql/workery_db.backup /home/techops/workery_db.backup
  ```

5. Delegate authority for the administrator to manage the exported database.

  ```
  chown techops /home/techops/workery_db.backup;
  chgrp techops /home/techops/workery_db.backup;
  ```

6. Log in as the administrator and confirm the database export was successful.

  ```
  su - techops
  cd ~/
  ls -alh
  ```

7. Congradulations you've exported a copy of the database.

### Developer Localhost
#### Download Database
The following instructions will work ``Linux`` or ``MacOS``.

1. Download the database (Mac/Linux)

  ```
  scp -i ~/.ssh/id_rsa techops@workery.ca:/home/techops/workery_db.backup /var/tmp
  ```

2. Congradulations you've saved a copy of the database onto your local machine.

3. Confirm your copy.

  ```
  ls -alh /var/tmp
  ```

#### Restore Database (MacOS)


1. Load up the ``postgres`` app (we are assuming you are running [Postgres.app](https://postgresapp.com)) and run the following. Please ensure you do not run the ``drop`` command in your **production environment**!

  ```
  drop database workery_db;
  create database workery_db;
  ```

2. Open ``Terminal`` and run the following:

  ```
  psql -U django workery_db < /var/tmp/workery_db.backup;
  ```

3. Congradulations you've restored the production environment data onto your local machine. Now before you run your application, please run the following backfill command.

  ```
  (env)$ python manage.py backfill
  ```
