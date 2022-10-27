---
title: Django on Docker - A Simple Introduction
slug: django-on-docker-a-simple-introduction

publish_timestamp: Sept. 19, 2019
url: https://www.codingforentrepreneurs.com/blog/django-on-docker-a-simple-introduction/

---


This is a simple tutorial but it isn't exactly for beginners. It's for people with some programming experience. 

But it's still very basic.

Docker is a great way to manage system-wide dependencies for your applications. Django is a easy way to launch web applications. The two together make a great pair.

##### Are you new to...
- Django? [Start here](https://www.codingforentrepreneurs.com/projects/try-django-2-2)
- Docker? [Start here](https://www.codingforentrepreneurs.com/blog/simple-docker)
- Python? [Start here](https://www.codingforentrepreneurs.com/projects/30-days-python)


##### What we're going to cover
In this one we're only going to cover creating a Django project and use Docker containers. We cover going into deployment with Django on Docker [here](https://www.codingforentrepreneurs.com/blog/django-docker-production-heroku/).


##### Requirements
- Docker installed. Do you not? Check out the post on [simple docker](https://www.codingforentrepreneurs.com/blog/simple-docker)

System Requirements
- Windows 10 Professional [docs](https://docs.docker.com/docker-for-windows/install/) (seriously; you need Virtual Machine support and Windows 10 Pro has it)
- macOS [docs](https://docs.docker.com/docker-for-mac/install/)


##### Final Project Structure
```
simple_dj_docker
|   .env
|   db.sqlite3
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
```
#### Your Django Project.

##### 1. Create a virtual environment with pipenv

Open up terminal (Mac) or PowerShell (Windows)
```
cd path/to/your/dev/folder/
```


```
mkdir simple_dj_docker
cd simple_dj_docker
pipenv install django==2.2.4 gunicorn --python 3.6
pipenv shell
```

##### 2. Create a Django project
```
django-admin startproject cfehome .
```

##### 3. Local DB & Migrations
```
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

##### 4. Update Django `settings.py`
```
# DEBUG can be True/False or 1/0
DEBUG = int(os.environ.get('DEBUG', default=1)) 
```

##### 5. Create `.env`
```
DEBUG=1
```

##### 6. Test `Gunicorn`
```
gunicorn cfehome.wsgi:application --bind 0.0.0.0:8000
```


#### Your Docker File, Image, & Container.

##### 1. Create your `Dockerfile`

```
$ cd path/to/your/dev/folder
$ cd simple_dj_docker
$ touch Dockerfile
$ ls
Dockerfile
```

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
# grab these via Python's os.environ
# these are 100% optional here
ENV PORT=8000

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

# Install project dependencies
RUN pipenv install --skip-lock --system --dev

EXPOSE 8888
CMD gunicorn cfehome.wsgi:application --bind 0.0.0.0:$PORT
```


##### 2. Build your Docker Image
```
$ cd path/to/your/dev/folder
$ cd simple_dj_docker
$ ls
Dockerfile
```
```
$ docker build -t simple-django-on-docker -f Dockerfile .
```

##### 3. Run your Container
```
$ docker run -it -p 80:8888 simple-django-on-docker
```
Now open up `http://localhost`

Use `Ctrl` + `C` to cancel the container running.


##### 4. Clean up

List running containers
```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
5966a461fa9d        simple-django-on-docker         "/bin/sh -c 'gunicorn …"   3 seconds ago       Exited (0) 1 second ago                       thirsty_cori
```

Stop your container
```
docker stop 5966a461fa9d
docker rm 5966a461fa9d
```

##### 5. What's next? Deploy this to a production server with [this guide](https://www.codingforentrepreneurs.com/blog/django-docker-production-heroku/)
