---
title: Django x Docker to Production on Heroku
slug: django-docker-production-heroku

publish_timestamp: Aug. 17, 2019
url: https://www.codingforentrepreneurs.com/blog/django-docker-production-heroku/

---


#### What is Docker and Why use it? 

If you cloned any of my [github](https://cfe.sh/github) projects and tried to run them, there's a **really** good chance that you'll have errors doing so unless you know *exactly* how I setup my system and my virtual environment. The majority of the repos cover the virtual environment part but not how my operating system is setup. That's challenge numero uno.

Manually deploying a [project to a linux server](https://www.codingforentrepreneurs.com/projects/hello-linux/) can be pretty straightforward but it can also be very cumbersome and tedious. Doing it hundreds of times is downright a waste of time and very, very, error prone.

Enter Docker. Docker streamlined this process by enabling you to configure the environment in advance. That configuration can be easily shared and easily automated. Further, Docker creates an isolated environment called a container (much like a Virtual Machine) which is great when you're dealing with all kinds of dependencies.

Deploying to Heroku outside of using Docker is often very easy because you just use git to push some code; Heroku handles the rest: installs dependencies, isolates environments, updates server-side software, etc.

##### So why does Docker & Heroku make sense?
Heroku has a hard code limit (aka "slug" limit) of 500MB. This is not ideal if your microservice or web application has large binaries built in (say like OpenCV or a machine learning library). Frankly, I think 500MB is just too small for a limit (my iPhone has apps that are 2+GB) but it make sense for how they traditionally run the deployment builds.

To get around Heroku's 500MB limit, I would have to manually deploy a project to a linux VM on Amazon EC2, DigitalOcean, Linode, etc, where I controlled 100% of the environment. This works really well but is very time consuming and often leads to frustrations when trying to upgrade these systems. I often have to re-learn how exactly I had these systems setup. 

[This project](http://cfe.sh/projects/hello-linux) was a result of that frustration, one single way to setup my Django/Python-based projects. I also developed complex bash scripts (the `.sh` shell scripts) to build environments that I needed.

> "Perfection is achieved not when there is nothing more to add, but when there is nothing left to remove." -- Antoine de Saint-Exupery

Docker streamlines the entire environment, Heroku makes it dead simple to deploy apps and provision needed add-ons (ie databases, email services, etc), and Django makes building powerful web applications super fast.

And thus, in this post we're going to cover:

1. A bare bones Django project
2. Docker, Containers, & Docker cli commands like `$ docker` & `$ docker-compose`
3. Deploying a Container to Heroku


### Tech Stack
- Ubuntu 18.04 in a Docker container
- Python 3.6
- Django 2.2
- Nginx
- PostgreSQL


### Final Project Structure

```
docking_django
|   .env
|   .env-db
|   db.sqlite3
|   docker-compose.yml
|   Dockerfile
│   manage.py
│   Pipfile  
│   Pipfile.lock
│
└───cfehome
│   │   __init__.py
│   │   settings.py
│   │   urls.py
│   │   wsgi.py
│   
└───nginx
    │   Dockerfile
    │   docking_django.conf
```


## Your Django Project.
The project we're working on here is *super* bare bones. That's on purpose. The goal is to learn how to use Django within a Docker container. Of course, if you're new to Django, this might be a post you come back to. 

#### 1. Create a virtual environment with pipenv
Open up terminal (Mac) or PowerShell (Windows). We're going to be using [pipenv](https://github.com/pypa/pipenv) to manage our virtual environment.

```
cd path/to/your/dev/folder/
mkdir docking_django
cd docking_django
pipenv install django==2.2.4 gunicorn dj-database-url --python 3.6
pipenv shell
```

#### 2. Create a Django project
```
django-admin startproject cfehome .
```

#### 3. Update `cfehome/settings.py`
Let's add some commented-out code for our future deployment settings. 
```
"""
Heroku database settings. Uncomment when you want to use it.
Put this below the DATABASE={ ... } configuration.
"""
# import dj_database_url
# db_from_env = dj_database_url.config()
# DATABASES['default'].update(db_from_env)
# DATABASES['default']['CONN_MAX_AGE'] = 500

```


#### 4. Migrations & Migrate
```
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

#### 5. Django Versions & Other Projects
Django is Django. Some would definitely argue this but I'd say Django 1.4 is 95% the same as Django 2.2. The 5% does cause a lot of confusion when working through various guides as well as the Python version you end up choosing. Such as Django 2.0+ doesn't support Python 2.7 and Django 1.11 does not support Python 3.7. And again, especially for beginners, Python 2.7 and 3.7 barely look different. 

So yes, Django versions **dont** matter for learning purposes. It only matters when you start to go into production. In that case use the latest version your comfortable with that's listed [here](https://www.djangoproject.com/download/).

We have a bunch of other Django projects in the following locations:

- [Our Website](https://cfe.sh/projects)
- [YouTube](https://cfe.sh/youtube)
- [GitHub](https://cfe.sh/github)

## Install & Use Docker Containers
Containers create an isolated software environment that can be easily moved from one system. 

To get Docker on your local system, you have to use Docker Desktop. It's an easy to use application for managing all things Docker. It also installs what we need to containerize our Django project.

- [Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac)
- [Docker Desktop for Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows)

### 3. Docker Configuration with `Dockerfile`

[[Docs](https://docs.docker.com/engine/reference/builder/)] A Dockerfile is the root configuration for our Docker application. 

**Create `Dockerfile`**
```
$ cd path/to/your/dev/folder/
$ cd docking_django
$ ls 
Pipfile		Pipfile.lock	cfehome		db.sqlite3		manage.py
$ touch Dockerfile
```

Within `Dockerfile`:
```
# Base Image
FROM ubuntu:18.04 as base

# set working directory
WORKDIR /usr/src/

# set default environment variables
ENV PYTHONUNBUFFERED 1
ENV LANG C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive 


# set project environment variables
# grab these via Python's os.environ
# these are 100% optional here
ENV AWS_ACCESS_KEY_ID abc123!
ENV AWS_SECRET_ACCESS_KEY 123abc!

# Install Ubuntu dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        tzdata \
        libopencv-dev \ 
        build-essential \
        libssl-dev \
        libpq-dev \
        libcurl4-gnutls-dev \
        libexpat1-dev \
        gettext \
        unzip \
        python3-setuptools \
        python3-pip \
        python3-dev \
        python3-venv \
        git \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*


# install environment dependencies
RUN pip3 install --upgrade pip 
RUN pip3 install psycopg2 pipenv

# Install project dependencies
COPY ./Pipfile /usr/src/Pipfile
RUN pipenv install --skip-lock --system --dev

# copy project to working dir
COPY . /usr/src/

CMD gunicorn cfehome.wsgi:application --bind 0.0.0.0:$PORT
```


### 2. Docker Compose `docker-compose.yml`
[[Docs](https://docs.docker.com/compose/)] Docker compose allows us to define what types of software services our application needs. We need a web application, a database volume, nginx, and possibly redis. `docker-compose.yml` is where we define what exactly our container needs which may include other Docker images. 

The `docker-compose.yml` file is also great for if you need to provide different configuration for your environment such as: `docker-compose.development.yml`, `docker-compose.production.yml`, `docker-compose.distribute.yml`, etc. 


**Create `docker-compose.yml`**
```
$ cd path/to/your/dev/folder/
$ cd docking_django
$ ls 
Pipfile		Pipfile.lock	cfehome		db.sqlite3		manage.py		DockerFile
$ touch docker-compose.yml
```

**Within `docker-compose.yml`**
```
version: '3.6'

services:
  web:
    build: .
    # command: python manage.py runserver 0.0.0.0:8000
    command: gunicorn cfehome.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - .:/usr/src/
    expose:
      - 8000
    env_file: .env
    depends_on:
      - db
  db:
    image: postgres:11.5-alpine
    restart: always
    volumes:
      - ./postgres_data:/var/lib/postgresql/data/
    env_file: .env-db
    ports:
      - 5432:5432
  redis:
    image: redis:alpine
    ports:
      - 6379:6379
  nginx:
    build: ./nginx
    ports:
      - 8000:80
    depends_on:
      - web

volumes:
  postgres_data:

```

### 3. Docker Compose Environment Variables
In the `docker-compose.yml` example above, you'll notice the `env_file`. We named them both `.env` and `.env-db`. These names are arbitary but generally follow convention. 

Since you can have multiple `docker-compose.yml` files for multiple environments, you can also have multiple `.env` files as you see fit. You can write your environment variables within the `docker-compose.yml` but I don't recommend it. (ie you often share `docker-compose.yml` but `.env` should listed in `.gitignore` files to prevent accidently sharing sensitive keys in github or bitbucket).

**Create `.env` and `.env-db`** (include the `.` before filename)

```
$ cd path/to/your/dev/folder/
$ cd docking_django
$ ls 
Dockerfile		cfehome			manage.py
Pipfile			db.sqlite3
Pipfile.lock		docker-compose.yml
$ touch .env
$ touch .env-db
```

In **.env**
_(Quoted strings are not needed)_

```
DEBUG=1
SECRET_KEY=adslfjlkjadsf**#@@#!@djksdf

AWS_ACCESS_KEY_ID=my_id
AWS_SECRET_ACCESS_KEY=my_key

DB_ENGINE=django.db.backends.postgresql
DB_TYPE=postgres
DB_DATABASE_NAME=docking_django  
DB_USERNAME=docking_user
DB_PASSWORD=docking_pw  
DB_HOST=db
DB_PORT=5432
```
`DB_HOST` is coming from the `docker-compose.html` service `db:` declaration. 


In **.env-db**
```
POSTGRES_DB=docking_django
POSTGRES_USER=docking_user
POSTGRES_PASSWORD=docking_pw#&@!
```


**Update `settings.py`**

```
SECRET_KEY = os.environ.get('SECRET_KEY', 'not-so-secret-key')

# DEBUG can be True/False or 1/0
DEBUG = int(os.environ.get('DEBUG', default=1)) 
```

```
"""
Local development database settings. 
"""
DATABASES = {
    'default': {
        'ENGINE': os.environ.get('DB_ENGINE', 'django.db.backends.sqlite3'),
        'NAME': os.environ.get('DB_DATABASE_NAME', os.path.join(BASE_DIR, 'db.sqlite3')),
        'USER': os.environ.get('DB_USERNAME', 'user'),
        'PASSWORD': os.environ.get('DB_PASSWORD', 'password'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}
```

### 4. `Nginx` Configuration

Nginx will use a pre-compiled docker image. This means we don't have to include it within our primary `Dockerfile` at all, instead it will have it's own `Dockerfile`.

Create a directory called `nginx` and within it a `Dockerfile`
```
mkdir nginx
touch nginx/Dockerfile
touch nginx/docking_django.conf
```

In `nginx/Dockerfile` add:
```
FROM nginx:1.17.2-alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY docking_django.conf /etc/nginx/conf.d
```


In `nginx/docking_django.conf` add:
```
upstream docking_django {
    server web:8000;
}

server {

    listen 80;

    location / {
        proxy_pass http://docking_django;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

}
```

That's it. Nginx is now ready for this project. Pretty simple right?

### 5. Docker and `docker-compose` Commands
Docker containers are ephemeral which means that the databases you create within them will *not persist* using the following commands. Since we're using Heroku, our database *will not* be handled by docker, it will be handled by heroku. That's because Docker is ephemeral and we want our database to persist. 

So, assuming you have Docker Desktop ([Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac) | [Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows)) installed and running, you can do the following commands in the root of your project:


**Build & Run**
This will run both `Dockerfile` and `docker-compose.yml` as needed. It will also run the command `gunicorn cfehome.wsgi:application --bind 0.0.0.0:$PORT` in the background on the port you designated for `nginx` in our case `8000`. 


**up**
```
docker-compose up -d --build
```
This might take a while to run especially if it's your first time doing so.

*Did you see `ERROR: Couldn't connect to Docker daemon....?` Naturally, this means you need to make sure Docker Desktop installed and is running.*

**Restart**
```
docker-compose restart
```

**Shutdown**
```
docker-compose down -v
```
*If you want to keep your postgres volume active, you'll want to remove `-v` from the docker-compose down`

##### Docker
**List Docker Containers**
```
docker ps
```

**List Docker Images**
```
docker images
```

> To delete all containers and images you can run `docker rm $(docker ps -a -q)` and 
`docker rmi $(docker images -q)` there is no way to restore these items after you delete them.

### 6. Coding with Docker
After your environment is setup, you can run `docker-compose up -d --build` and `docker-compose down` as much as you want. Typically when I'm done with a project for the day, I'll be sure to run `docker-compose down` since I don't need it running any longer and it should free up some ports such as `localhost:8000`.

While you code, it's useful to just use `docker-compose restart` instead of up then down since it just needs to update the code within the environment. In other words, the code won't auto-update (as far as I know) within your docker container's environment. 

Running into errors? 
1. Is your container environment complete and up to date? `Dockerfile` and `docker-compose.yml`
2. Did you start the container with `docker-compose up -d --build`?
3. Did you shut down your other containers running `docker-compose up`?
4. Did you save everything?
5. Is Docker Desktop running? Is it up to date?


## Deploy Django with Docker & Heroku

#### 1. Sign up for [Heroku](https://www.heroku.com)

#### 2. Download the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)

Open up Terminal (Mac/Linux) or PowerShell (Windows) and login to Heroku. 

```
heroku login
```

#### 3. Create a Heroku App

Navigate to your project:
```
cd path/to/your/dev/folder/
cd docking_django
```

Use the `heroku create` command to make your app.
```
heroku create docking-django
```
Take note, my heroku app name is `docking-django`

#### 4. Login to the Heroku Container Registry

```
heroku container:login
```

#### 5. Provision your Add-ons


**PostgreSQL & Databases**
You should **never** dockertize your database in production unless you know exactly how to back them up. Heroku also requires you to use their database add-ons. Remember, Docker containers are ephemeral so if you push a new image, it would, in theory, erase your old database.

```
heroku addons:create heroku-postgresql:hobby-dev
```
> If you get ` ›   Error: Missing required flag:
 ›     -a, --app APP  app to run command against
 ›   See more help with --help` then just add your app name to your command so it's like `heroku addons:create heroku-postgresql:hobby-dev -a docking-django`
 
 
**Redis**
Our Docking Django project had Redis as one of the services used even if it wasn't actually implemented in Django. Let's add the free redis addon anyways:

```
heroku addons:create heroku-redis:hobby-dev
```

That's it for addons now. We don't need an `nginx` service because Heroku has the needed web server for running `gunicorn` as nginx would otherwise.

#### 6. Update Django Database Settings

In `settings.py` uncomment the following lines:
```
"""
Heroku database settings. Uncomment when you want to use it.
"""

import dj_database_url
db_from_env = dj_database_url.config()
DATABASES['default'].update(db_from_env)
DATABASES['default']['CONN_MAX_AGE'] = 500
```


#### 7. Push & Release

Now that we have our Postgres DB and Redis provisioned, we only have 1 service left to send to heroku. That's the `web` service from the `docker-compose.yml` file. 

**Push**
```
heroku container:push web
```
or
```
heroku container:push web -a docking-django
```

**Release**
If all went well with the push, you can release it:

```
heroku container:release web
```
or
```
heroku container:release web -a docking-django
```


#### 8. Run `migrate` & `createsuperuser`
With heroku you can just simply run:

**Migrate**
```
heroku run python3 manage.py migrate
```
or 
```
heroku run python3 manage.py migrate -a docking-django
```

**Create super user**
```
heroku run python3 manage.py createsuperuser
```
or
```
heroku run python3 manage.py createsuperuser -a docking-django
```

**Bash**
```
heroku run bash
```
or
```
heroku run bash -a docking-django
```

## Next Steps

Now that you have a Django docker project deployed on Heroku, it's time to experiment with building more robust Django projects as well as implementing larger binaries like OpenCV, Tensorflow, or PyTorch. 

Good luck!

---------
Did I miss anything? Please me know in the comments.
