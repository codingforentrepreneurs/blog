---
title: Serverless Django with PostgreSQL with Cloud SQL and Cloud Run
slug: serverless-django-postgresql-cloud-sql-cloud-run

publish_timestamp: May 2, 2020
url: https://www.codingforentrepreneurs.com/blog/serverless-django-postgresql-cloud-sql-cloud-run/

---


Below is a reference guide for the [Serverless Django](https://www.codingforentrepreneurs.com/projects/serverless-django) project.

I created it as a configuration overview for using Django on Google Cloud Run as a Serverless Application using Google Cloud SQL's postgres database.

Keep in mind that there's a lot of context missing here which is why I recommend you go through the project first and then reference this for every future project.


## Local Reference Paths

**My local username**: (type `whoami` in Terminal/PowerShell):
```
cfe
```

**Local project path** (using macOS): 
```
/Users/cfe/dev/serverless-django
``` 

**Local SQL Proxy Executable**: 
```
/Users/cfe/cloud_sql_proxy
```

**Local IAM Service Account Credential File Path**: 
```
/Users/cfe/dev/serverless-django/db-proxy-iam.json
```


##  Google Cloud Reference
The below reference is connected to my project directly. Change yours accordingly.

**Google cloud project id** `<gcp-project-id>`: 
```
serverless-cfe
````

**Container tag**

```
serverless-django
```

**Container registry hostname** ([options](https://cloud.google.com/container-registry/docs/overview#registry_name)): 
```
gcr.io
```

**Container Registery Image** (`<registry-hostname>/<gcp-project-id>/<container-tag>`):

```
gcr.io/serverless-cfe/serverless-django
```

**Cloud Run service name**: 
```
serverless-django-run
```

**Cloud SQL Instance Name**: 

```
serverless-django-sql-01
```

**Cloud Run & SQL Region** `<gcp-region>`: 
```
us-west1
```

**Cloud SQL Instance Connection Name** ( `<gcp-project-id>:<gcp-region>:<cloud-sql-instance-name>`):

```
serverless-cfe:us-west1:serverless-django-sql-01
```

> You can also find the instance connection name on the [Cloud SQL Instance](https://console.cloud.google.com/sql/instances) itself.

**Database name** (in the Cloud SQL Instance):
```
serverlessdjangodb
```


**IAM Service Account Email**:
```
django-sql-service-account@serverless-cfe.iam.gserviceaccount.com
```


## Django Project Reference
**Project name** `django-admin startproject <project-name>`
```
serverless_django
```

**Django base project tree**

```bash
├── Dockerfile
├── Pipfile
├── db-proxy-iam.json
├── db.sqlite3
├── manage.py
├── scripts
│   ├── build.sh
│   ├── db_proxy.sh
│   ├── deploy.sh
│   ├── docker_run.sh
│   ├── entrypoint.sh
│   └── gcp.sh
└── serverless_django
    ├── __init__.py
    ├── asgi.py
    ├── settings
    │   ├── __init__.py
    │   ├── local.py
    │   └── production.py
    ├── urls.py
    └── wsgi.py
```

> As you can see, we've moved `settings.py` to `settings/local.py` and created `settings/production.py`. This is to keep our environments separate. Read more on [staging django](https://www.codingforentrepreneurs.com/blog/staging-django-production-development).

**Django Database Settings**

Be sure to run `pipenv install psycopg2-binary`

```
# settings/local.py
DATABASES = {
        'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'serverless_django_local_db',
        'USER': 'serverless_django_local_user',
        'PASSWORD': '<your-password>',
        'HOST': '127.0.0.1',
        'PORT': 6543,
    }
}
```
> `PORT`: `6543` is used since we'll be using a local proxy to our Google Cloud SQL database.

```
# settings/production.py

INSTANCE_CONNECTION_NAME = os.environ("INSTANCE_CONNECTION_NAME)
DATABASES = {
        'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'serverless_django_prod_db',
        'USER': 'serverless_django_prod_user',
        'PASSWORD': '<your-password>',
        'HOST': f'/cloudsql/{INSTANCE_CONNECTION_NAME}'
    }
}
```
> `INSTANCE_CONNECTION_NAME` is set as an environment variable based on the `Cloud SQL Instance Connection Name` that is set we run `gcloud run deploy`



## Automation & Deployment Scripts
All of the below scripts live in the `scripts/` directory at the root of my Django project (ie next to `manage.py`).

> **Windows Users**: a lot of these scripts can be run within PowerShell, just change `.sh` to `.ps1` and you should be able to run it like `.\scripts\build.ps1`

##### `scripts/build.sh` 
```
gcloud builds submit --tag gcr.io/serverless-cfe/serverless-django .
```

##### `scripts/db_proxy.sh`
This is used for running a local proxy into our database. Below we use the `tcp` port `6543` to avoid any conflicts with any local version of postgresql running.
```
exec /Users/cfe/cloud_sql_proxy-instances=serverless-cfe:us-west1:serverless-django-sql-01=tcp:6543 -credential_file=/Users/cfe/dev/serverless-django/db-proxy-iam.json
```


##### `scripts/deploy.sh`

```
gcloud builds submit --tag gcr.io/serverless-cfe/serverless-django .

gcloud run deploy serverless-django-run --image gcr.io/serverless-cfe/serverless-django \
--platform managed \
--region us-west1  \
--allow-unauthenticated \
--add-cloudsql-instances serverless-django-sql-01 \
--update-env-vars INSTANCE_CONNECTION_NAME="serverless-cfe:us-west1:serverless-django-sql-01" \
--service-account django-sql-service-account@serverless-cfe.iam.gserviceaccount.com
```


##### `scripts/docker_run.sh`
Build and run project locally. This simplifies testing. Setting the `PORT` environment variable is required since Google Run sets it for us as well.

```
docker build -t serverless-django -f Dockerfile .
docker run --env PORT=8888 -it -p 8888:8888 serverless-django
```


##### `scripts/entrypoint.sh`
```
#!/bin/bash

/usr/local/bin/gunicorn serverless_django.wsgi:application --bind "0.0.0.0:$PORT" --workers 3 --env DJANGO_SETTINGS_MODULE=serverless_django.settings.production
```



## Postgres on Cloud SQL
Assuming you have [Cloud SQL Proxy](https://cloud.google.com/sql/docs/postgres/sql-proxy) setup locally and a [Cloud SQL Instance](https://console.cloud.google.com/sql/instances) we'll create our databases and database users with the following:

1. Start your database [proxy](https://cloud.google.com/sql/docs/postgres/sql-proxy) (`scripts/db_proxy.sh` created above):
    ```
    ./scripts/db_proxy.sh
    ```

2. Verify postgres is installed locally (or install via [homebrew](https://brew.sh) with `brew install postgres`):
    ```
    $ psql -V
    psql (PostgreSQL) 12.2
    ```


3. Login to Cloud SQL PostgreSQL via 
    ```
    psql -U postgres -h 127.0.0.1 -p 6543 -d postgres
    ```

4. Create users & databases for our projects

    **Production database**
    ```
    CREATE ROLE serverless_django_prod_db WITH LOGIN PASSWORD '<your-password>';
    ALTER ROLE serverless_django_prod_user CREATEDB;

    create database serverless_django_prod_db;
    grant all privileges on database serverless_django_prod_db to serverless_django_prod_user;
    ```

    **Development database**
    ```
    CREATE ROLE serverless_django_local_db WITH LOGIN PASSWORD '<your-password>';
    ALTER ROLE serverless_django_local_user CREATEDB;

    create database serverless_django_local_db;
    grant all privileges on database serverless_django_local_db to serverless_django_local_user;
    ```


## Dockerfile

```
FROM python:3.8.2-slim

ENV APP_HOME /app
WORKDIR $APP_HOME

COPY . ./

RUN pip install pip pipenv django gunicorn --upgrade
RUN pipenv install --skip-lock --system --dev

CMD ["./scripts/entrypoint.sh"]
```


Resources: 
- [Preparing your Django Application for Google Cloud Run](https://medium.com/swlh/preparing-your-django-application-for-google-cloud-run-7c8cb7b7464b). This guide was huge help even with some things changing.
- [Connecting to Cloud SQL from Cloud Run](https://cloud.google.com/sql/docs/mysql/connect-run) (GCP Docs)
- [Cloud SQL Proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy)
