On your server, please make sure the `csv` folder has been created under the `django` user account.

```
mkdir /opt/django/nwapp-back/nwapp/tenant_etl/csv
```


Go to the directory with all the CSV files in your local development machine:

```
cd ~/python/github.com/nwatchcanada/nwapp-back/nwapp/tenant_etl/csv
```

Upload to the ``django`` user account.

```
scp prod_districts_plus_uuid.csv django@nwapp.ws:/opt/django/nwapp-back/nwapp/tenant_etl/csv/;
scp prod_watches_plus_uuid.csv django@nwapp.ws:/opt/django/nwapp-back/nwapp/tenant_etl/csv/;
scp prod_members_plus_uuid.csv django@nwapp.ws:/opt/django/nwapp-back/nwapp/tenant_etl/csv/;
```
