---
title: Deploying Django on Docker to Heroku with OpenCV
slug: deploy-django-on-docker-to-heroku-opencv

publish_timestamp: Sept. 20, 2019
url: https://www.codingforentrepreneurs.com/blog/deploy-django-on-docker-to-heroku-opencv/

---


Building off [this post](https://www.codingforentrepreneurs.com/blog/django-on-docker-a-simple-introduction) we're going to be deploying a Django on Docker project to [Heroku](heroku.com).


##### Are you new to...
- Django? [Start here](https://www.codingforentrepreneurs.com/projects/try-django-2-2)
- Docker? [Start here](https://www.codingforentrepreneurs.com/blog/simple-docker)
- Python? [Start here](https://www.codingforentrepreneurs.com/projects/30-days-python)


##### Requirements
- Docker/Docker Desktop installed. Do you not? Check out the post on [simple docker](https://www.codingforentrepreneurs.com/blog/simple-docker)

- Heroku CLI Installed from [here](https://devcenter.heroku.com/articles/heroku-cli)


System Requirements
- Windows 10 Professional [docs](https://docs.docker.com/docker-for-windows/install/) (seriously; you need Virtual Machine support and Windows 10 Pro has it)
- macOS [docs](https://docs.docker.com/docker-for-mac/install/) (you might need to install [XCode](https://developer.apple.com/xcode/) as well.)




##### Final Project & Structure
```
simple_dj_docker
|   .env
|   db.sqlite3
|   deploy.ps1
|   deploy.sh
|   Dockerfile
|   Dockerfile-local
│   manage.py
│   Pipfile  
│   Pipfile.lock
│
└───cfehome
│   │   __init__.py
│   │   settings.py
│   │   urls.py
│   │   wsgi.py
```

### Get Django Working Locally


##### 1. Clone/Copy the project from [this post](https://www.codingforentrepreneurs.com/blog/django-on-docker-a-simple-introduction).

Open up terminal (Mac) or PowerShell (Windows)
```
cd path/to/your/dev/folder/
mkdir dj_docker_to_heroku
cd dj_docker_to_heroku
```

**Clone or Copy [the repo](https://github.com/codingforentrepreneurs/Django-on-Docker)**

```
git clone https://github.com/codingforentrepreneurs/Django-on-Docker .
```
> Don't know how to use git? Watch the video to see how we download it. It's easy. The [starter code](https://github.com/codingforentrepreneurs/Django-on-Docker) is here.


##### 2. Run `pipenv` install
```
$ cd path/to/your/dev/folder/
$ cd dj_docker_to_heroku
$ ls
Dockerfile  Pipfile     README.md   manage.py
LICENSE     Pipfile.lock    cfehome 
```

```
pipenv install
```

##### 3. Verify Django is working

```
pipenv run gunicorn cfehome.wsgi:application --bind 0.0.0.0:8888
```
Open [http://localhost:8888](http://localhost:8888) in your web browser. Do you see the Django starter page? Sweet. You're good to go.

> Above is our final container command, you can also run `pipenv run python manage.py runserver`


Hit `control` + `c` to cancel the server out.


### Get Django on Docker working Locally

Don't have docker installed? Go through [this post](https://www.codingforentrepreneurs.com/blog/simple-docker).

##### 1. Verify Docker Install
Open up terminal (Mac) or PowerShell (Windows)
```
$ docker --version
Docker version 19.03.2, build 6a30dfc
```

##### 2. Build your project's image.

**Navigate to your project**
```
$ cd path/to/your/dev/folder/
$ cd dj_docker_to_heroku
$ ls
Dockerfile  Pipfile     README.md   manage.py
LICENSE     Pipfile.lock    cfehome 
```

**Run the `docker build` command**
```
docker build -t dj-docker-to-heroku -f Dockerfile .
```
Of course the line `dj-docker-to-heroku` is custom. Change yours if you want.


##### 3. Run a container

```
docker run -it -p 8888:8888 dj-docker-to-heroku
```
Open [http://localhost:8888](http://localhost:8888) 


Is your project still running? Sweet. Let's do the fun part.




## Deploy Django with Docker & Heroku

#### Pre-Steps
- Sign up for [Heroku](https://www.heroku.com)
- Download and install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)


#### 1. Login to heroku
Open up Terminal (Mac/Linux) or PowerShell (Windows) and login to Heroku. 

**Verify**
```
$ heroku --version
heroku/7.29.0 darwin-x64 node-v11.14.0
```

**Login**
```
heroku login
```


#### 3. Create a Heroku App

**Navigate to your project**
```
$ cd path/to/your/dev/folder/
$ cd dj_docker_to_heroku
```

Use the `heroku create` command to make your app.
```
heroku create dj-docker
```
Take note, my heroku app name is `dj-docker`

#### 4. Login to the Heroku Container Registry

```
heroku container:login
```

> I haven't convered what a 'container registry' is yet but think of it like a version-control for your deployed containers. Whenver you push your project via Docker to heroku, it's going to use an instance of your docker image. That instance of course is called a container. A container registry (there are many) holds a copy of this instance for machines to use. If that's confusing, please ask questions below.


#### 5. Provision your Add-ons


**PostgreSQL Database for Production**
I'll make this simple. Don't use databases in a container. A container is like an ice cube. Once it melts, it's gone. You can easily make another one by using an ice tray but the old ice cube's contents are gone forever.

Databases need to live forever. Let's use heroku's managed database service for free:

```
heroku addons:create heroku-postgresql:hobby-dev -a dj-docker
```

Replace `dj-docker` with your Heroku project's name
 

#### 6. Update Django Database Settings

In `settings.py` uncomment the following lines:
```
"""
Heroku database settings. 
"""

import dj_database_url
db_from_env = dj_database_url.config()
DATABASES['default'].update(db_from_env)
DATABASES['default']['CONN_MAX_AGE'] = 500
```


#### 7. Update our `pipenv` environment with new depedancies

```
$ cd path/to/your/dev/folder/
$ cd dj_docker_to_heroku
$ pipenv install dj-database-url psycopg2
```
> Mac users: I had to open XCode and accept permissions for `psycopg2` to install. 

#### 8. Change our Dockerfile to be Heroku Ready
Docker images on heroku need to be slightly different than what we can do locally. Heroku has specific parameters for containers as you can read about [here](https://devcenter.heroku.com/articles/container-registry-and-runtime).

We're going to create a Dockerfile specifically for Heroku deployments.
```
$ cd path/to/your/dev/folder/
$ cd dj_docker_to_heroku
$ mv Dockerfile Dockerfile-local
$ touch Dockerfile
```
> Windows users: use `echo $null >> Dockerfile-Heroku` instead of `touch`

`Dockerfile-local` is here in case we want to do additional docker-related testing locally. Heroku looks the specific filename `Dockerfile` when it's building a container. When we test locally now, we can just use `docker build -t dj-docker-to-heroku-local -f Dockerfile-local .`

In your new `Dockerfile`

```
# Base Image
FROM python:3.6

# create and set working directory
RUN mkdir /app
WORKDIR /app

# Add current directory code to working directory
ADD . /app/

# set default environment variables
ENV PYTHONUNBUFFERED 1
ENV LANG C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive 

# set project environment variables
# grab these via the Python os.environ
# these are 100% optional here
# $PORT is set by Heroku
ENV PORT=8888

# Install system dependencies with OpenCV
RUN apt-get update && apt-get install -y --no-install-recommends \
        tzdata \
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
RUN pip3 install pipenv

# Install project dependencies
RUN pipenv install --skip-lock --system --dev

# Expose is NOT supported by Heroku
# EXPOSE 8888
CMD gunicorn cfehome.wsgi:application --bind 0.0.0.0:$PORT

```


#### 9. Build Docker Image
Whenever you make changes to your code or virtual environment, it's important to build another image. 

```
docker build -t dj-docker-to-heroku -f Dockerfile .
```

#### 10. Push & Release

Now we need to push an instance (container) of our image to heroku. 

**Push**
```
heroku container:push web -a dj-docker
```
Replace `dj-docker` with your Heroku project's name

**Release**
If all went well with the push, you can release it:


```
heroku container:release web -a dj-docker
```


#### 11. Run `migrate` & `createsuperuser`
These are very important when you do a push with Django. Be sure to replace `dj-docker` with your Heroku's project name.

With heroku you can just simply run:

**Migrate**

```
heroku run python3 manage.py migrate -a dj-docker
```
Replace `dj-docker` with your project name


**Create super user**
```
heroku run python3 manage.py createsuperuser -a dj-docker
```

**Bash**
```
heroku run bash -a dj-docker
```


#### 12. Create a *deploy* script. 
The purpose of this is so we don't have to remember all the steps to deploy our code. I *love* making these for my production projects.

##### Mac/Linux users
**Navigate to your project**
```
$ cd path/to/your/dev/folder/
$ cd dj_docker_to_heroku
$ nano deploy.sh
$ sudo chmod +x deploy.sh
```

**In `deploy.sh`**
```
# build our heroku-ready local Docker image
docker build -t dj-docker-to-heroku -f Dockerfile .


# push your directory container for the web process to heroku
heroku container:push web -a dj-docker


# promote the web process with your container 
heroku container:release web -a dj-docker


# run migrations
heroku run python3 manage.py migrate -a dj-docker
```
Naturally my tagged Docker image is named `dj-docker-to-heroku` and my Heroku app is `dj-docker`. Your names will be different.

**Test**
```
$ cd path/to/your/dev/folder/
$ cd dj_docker_to_heroku
$ ./deploy.sh
```


##### Windows users
**Navigate to your project**
```
> cd path/to/your/dev/folder/
> cd dj_docker_to_heroku
> echo $null >> deploy.ps1
```

**In `deploy.ps1`**
```
# build our local Docker image
docker build -t dj-docker-to-heroku -f Dockerfile .


# push a container to heroku
heroku container:push web -a dj-docker


# promote the container 
heroku container:release web-a dj-docker


# run migrations
heroku run python3 manage.py migrate -a dj-docker
```

**Test**

```
> cd path/to/your/dev/folder/
> cd dj_docker_to_heroku
> ./deploy.sh
```


### BONUS: OpenCV in Production, on Heroku, with Django on Docker
Keep in mind that the final code will be on [our repo for this post](https://github.com/codingforentrepreneurs/Django-on-Docker-to-Heroku).

In your `Dockerfile` within your project `path/to/your/dev/folder/dj_docker_to_heroku/`, we need to update a few required installations.

**From**
```
# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
        tzdata \
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
RUN pip3 install pipenv
```

Run your deploy scripts from above.


**To**
```
# Install system dependencies with OpenCV
RUN apt-get update && apt-get install -y --no-install-recommends \
        tzdata \
        libopencv-dev \ 
        build-essential \
        libssl-dev \
        libpq-dev \
        libcurl4-gnutls-dev \
        libexpat1-dev \
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
RUN pip3 install pipenv
RUN pip3 install opencv-contrib-python

```
> If you're having issues remember that OpenCV is a large program; it might take up a good amount of space on your system.  Also, you might have to reference [our repo for this post](https://github.com/codingforentrepreneurs/Django-on-Docker-to-Heroku) to ensure your code/Dockerfile matches ours.


##### Test OpenCV in Production
```
heroku run python3 -c "import cv2; print(cv2.__version__)" -a dj-docker
```

##### Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/1pZbuvbvYY8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
