---
title: Django, Gunicorn, &amp; NGINX Running together within a Container
slug: django-gunicorn-nginx-docker

publish_timestamp: Nov. 18, 2022
url: https://www.codingforentrepreneurs.com/blog/django-gunicorn-nginx-docker/

---


If you're running [Django](https://djangoproject.com) & [gunicorn](https://docs.gunicorn.org/) within a [Docker](https://www.docker.com) container and need to limit access to a specific domain, what do you do?

The answer? Run NGINX along side gunicorn and Django. 

This post will describe how you do this. Let's not confuse this with running two containers but rather running nginx and gunicorn with the same container. 

To do this, we'll have Django running via gunicorn as a daemon process (i.e. background) and NGINX as a non-daemon process (i.e. non-background).

The contents in this post are _not production-ready_ but can be modified to be production-ready by at least using environment variables, an actual production database, turning off debug mode in Django, leveraging a static files server, and likely other items that you can learn about in my course [Django Deployment Pipeline](https://www.codingforentrepreneurs.com/courses/django-deployment-pipeline/).

__Sample project and code are all on [github](https://github.com/codingforentrepreneurs/django-gunicorn-nginx-docker-containers)__

## Create a local virtual environment

Navigate to a dev folder
```bash
cd path/to/your/dev/folder
```

Make a project folder
```bash
mkdir -p django-nginx
```
Create a Python virtual environment
```bash
python3 -m venv venv
```
I prefer `venv` but you can use whatever you'd like. If you're using windows you'll use something like `C:\Python311\python.exe -m venv venv`

Activate it:
```bash
source venv/bin/activate
```
Again on windows it's: `.\venv\Scripts\activate`


## Create `requirements.txt` and Install

```bash
cd path/to/your/dev/folder/
cd django-nginx
```

Make an `src` folder to hold our django project
```bash
mkdir -p src
```

Add `requirements.txt` to src:

```bash
echo "" > src/requirements.txt
echo "django" >> src/requirements.txt
echo "gunicorn" >> src/requirements.txt
```

Naturally, you'll need to add any and all other project requirements


Now let's install these:

```bash
$(venv) python -m pip install -r src/requirements.txt
```
You can also use `venv/bin/python -m pip install -r src/requirements.txt`



## Create Django project

```bash
cd path/to/your/dev/folder/
cd django-nginx
cd src
```

```bash
$(venv) django-admin startproject cfehome .
```
- You can also use `../venv/bin/django-admin startproject cfehome .` within the `src/` folder.
- `cfehome` is the name of our django project, call it what you'd like just be sure to update everything else to the name you pick.


## Add Nginx configuration
Below we'll create a basic nginx configuration to forward traffic along to our django/gunicorn project running at port `8000`. It's important to note that this port _will not_ change regardless of where this container is running. It's true we _could_ change it but it's really not necessary.


```bash
cd path/to/your/dev/folder/
cd django-nginx
mkdir nginx
```

Create `nginx/default.conf`:

```nginx
 upstream django_project {
    server localhost:8000;
}

error_log /var/log/nginx/error.log;

server {
    listen       80;
    server_name  codingforentrepreneurs.com *.codingforentrepreneurs.com;
    root   /www/data/;
    access_log /var/log/nginx/access.log;

    location / {
        proxy_pass http://django_project;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
Update your `server_name` directive with all the domain names you might need. 

A few things to note:
- `error_log /var/log/nginx/error.log;` and `access_log /var/log/nginx/access.log;` will be linked to `/dev/stdout` within our `Dockerfile`
- The name `django_project` for the `upstream` declaration matches the `proxy_pass` declaration (`http://django_project;`). You do _not_ need `https` within your container or this NGINX configuration.
- Nginx can load balance to other instances of nginx. This configuration should _not_ load balance to other containers. 


## Create `entrypoint.sh`

Now, our Docker configuration needs an external file that will end up execute our processes during runtime and `entrypoint.sh` will serve as this file.

With this file we'll:

- run _gunicorn_ as a daemon process (background)
- run _nginx_ as a non-daemon process

It might not surprise you but this is the _opposite_ of what these programs do by default... and that's okay! That said, the official [nginx image](https://hub.docker.com/_/nginx) runs as a non-daemon process too. 


```bash
cd path/to/your/dev/folder/
cd django-nginx/src
mkdir -p config
```

In `django-nginx/src/config/entrypoint.sh` we'll put:
```bash
#!/bin/bash
RUN_PORT="8000"

/opt/venv/bin/python manage.py migrate --no-input
/opt/venv/bin/gunicorn cfehome.wsgi:application --bind "0.0.0.0:${RUN_PORT}" --daemon

nginx -g 'daemon off;'
```
A few things to note here:
- `RUN_PORT="8000"` is the port that gunicorn will bind to __internally__ for the container. When you run your container you should _not_ map to this PORT unless you want to circumvent using nginx (thus circumventing the point of this article.) 
- `/opt/venv/bin/` will be created in our `Dockerfile` along with the command `python -m venv /opt/venv`. The directory `/opt` is a great place to store virtual environments inside containers.
- `python manage.py migrate --no-input` will run our database migrations if needed. This command should _never_ be run during a _container build time_. 
- `gunicorn ... --bind "0.0.0.0:${RUN_PORT}" ` does the actual binding to this port. I keep it as a variable since I reuse this `entrypoint.sh` configuration a lot.
- `gunicorn ... --daemon` is how we run gunicorn in the background. Using `--daemon` is _only recommended_ in a Docker container when you have a programming that is a non-daemon process (ie a program that runs continuously in the foreground).
- `nginx -g 'daemon off;'` run nginx and run it in a non-background (daemon) mode.


## Create `Dockerfile`


```bash
cd path/to/your/dev/folder/
cd django-nginx
```

In `django-nginx/Dockerfile` add:

```dockerfile
# yup, python 3.11!
FROM python:3.11-slim

# install nginx
RUN apt-get update && apt-get install nginx -y
# copy our nginx configuration to overwrite nginx defaults
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
# link nginx logs to container stdout
RUN ln -sf /dev/stdout /var/log/nginx/access.log && ln -sf /dev/stderr /var/log/nginx/error.log

# copy the django code
COPY ./src ./app

# change our working directory to the django projcet roo
WORKDIR /app

# create virtual env (notice the location?)
# update pip
# install requirements
RUN python -m venv /opt/venv && \
    /opt/venv/bin/python -m pip install pip --upgrade && \
    /opt/venv/bin/python -m pip install -r requirements.txt

# make our entrypoint.sh executable
RUN chmod +x config/entrypoint.sh

# execute our entrypoint.sh file
CMD ["./config/entrypoint.sh"]
```

## Build and run!

Build this project:
```
docker build -t django-nginx -f Dockerfile .
```

Run the container:
```
docker run -p 8000:80 --name django-nginx --rm django-nginx 
```

The confusing part of this command is likely `-p 8000;80` because we used `8000` above in both the `nginx/default.conf` file as well as the `src/config/entrypoint.sh` file. 

The machine that is running `docker run` will map `localhost:8000` to the docker conatiner of port `80`. It was by design that `nginx/default.conf` is listening at port `80`. 

To circumvent `nginx` you'd use the port mapping of `-p 8000:8000` as port `8000` within the container is being handled by `gunicorn`.
