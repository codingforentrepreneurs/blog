---
title: Django on Docker
slug: django-on-docker

publish_timestamp: Jan. 22, 2022
url: https://www.codingforentrepreneurs.com/blog/django-on-docker/

---

As you may know, Django is a web application framework written in Python aka it's a bunch of Python code. What version of Python? Ideally 3.7 and up. Before we jump into the Docker of it all, let's start with a simple Django project. 

Are you new to Django? Be sure to get some experience first like watching [this series](https://www.youtube.com/watch?v=SlHBNXW1rTk&list=PLEsfXFp6DpzRMby_cSoWTFw8zaMdTEXgL).

If you find this guide helpful, you might also find my [Docker & Docker compose series](https://www.codingforentrepreneurs.com/projects/docker-and-docker-compose) very helpful too.


## Step 1: Create a Simple Django Project
I assume you already have Python 3.7+ installed on your local machine. If you don't you should do that now. If you're on linux, you may need to install additional dependencies to get his working (check the `Dockerfile` below for that).

Navigate to your project directory
```
cd path/to/dev/folder/
```

Create a python virtual environment
```
python3.8 -m venv venv
```

Activate your virtual environment
```
source venv/bin/activate
```

Upgrade pip
```
pip install pip --upgrade
```

Make `src` folder
```
mkdir -p src
```

Change into `src` folder
```
cd src
```

Install Django
```
pip install "Django>=4.0,<4.1"
```

Start Django Project within `src` folder
```
django-admin startproject myproj .
```

Change back to project root
```
cd ..
```

Add/Update requirements.txt

```
echo "Django>=4.0,<4.1" >> requirements.txt
echo "gunicorn" >> requirements.txt
```

Verify `requirements.txt` installed correctly
```
pip install -r requirements.txt
```

Create an empty `Dockerfile` and `entrypoint.sh`
```
echo "" > Dockerfile
echo "" > entrypoint.sh
```

Update `dockerignore`
```
echo "venv" >> .dockerignore
echo "*.py[cod]" >> .dockerignore
echo "__pycache__" >> .dockerignore
```
> If on windows, you'll have to run simply `python -m venv venv` and `.\venv\Scripts\activate` instead. All other commands should be the same. If you have multiple Python versions installed, you may even have to run `C:\Python38\python.exe -m venv venv`.

Here's the current structure of your project (omitting `venv` related files):

```
.
├── .dockerignore
├── Dockerfile
├── entrypoint.sh
├── requirements.txt
├── src
|   ├── manage.py
|   └── myproj
|       ├── __init__.py
|       ├── asgi.py
|       ├── settings.py
|       ├── urls.py
|       └── wsgi.py
└── venv
```

We're going to work on this structure for the remainder of this post. I suggest you consider [staging Django for production](https://www.codingforentrepreneurs.com/blog/staging-django-production-development/) so you can configure your `settings.py` correctly. 


## Step 2: The System _Instructions List_
What are the exact steps to get our Django project running on nearly *any* system? Let's talk in general terms to help illuminate why we need Docker.


First requirement is Python installed and we'll pick version 3.8. Let's add that to our _Instructions List_:
- Python 3.8

Now that we have Python installed, we need to keep in mind that using Virtual Environments (aka `virtualenv`, `venv`, `pipenv`, etc) is highly recommend to help isolate Python apps from other Python apps. I prefer to think of them as _Python virtual environments_ because that's pretty much all the isolation they provide. If you're from the JavaScript/Node world, you can think of this like how `node_modules` and `package.json` appear in nearly every project.

With that in mind, here's our new _Instructions List_:
- Python 3.8
- Virtual Environment

In __Step 1__ we create `requirements.txt` with `Django` and `gunicorn`. The `requirements.txt` file is just a simple way to keep track of what this particular Python application needs to be installed to run in addition to just simply Python. `requirements.txt` is not the only way this is done but it is one of the most used ways to do so since it's simple and well supported.

This means we need our Virtual Environment to install these requirements. So let's update our _Instructions List_:

- Python 3.8
- Virtual Environment
- path/to/my/venv/bin/pip install -r requirements.txt


The last step is how do we *run* our Python web application. If you're familiar with Django, you'll know you can run the following:

```
python manage.py runserver
```

As it clearly states after you run this command, it's made for **development** and is definitely not suitable for **production**. For the purposes of this guide, we want to use a **production** server instead of a development one. Luckily, we already have it listed in `requirements.txt` and it's `gunicorn`. "G" unicorn is a Web Server Gateway Interface (`wsgi`) HTTP server application that provides a reliable way to run and scale Django in production.


To run gunicorn with our project from __Step 1__ we'll:
```
cd src
gunicorn myproj.wsgi:application
```
Let me break down this command a little:
- `guincorn` this of course calls the gunicorn executable. When in doubt, use the absolute path to this gunicorn (such as `path/to/dev/folder/venv/bin/gunicorn` from __Step 1__)
- `myproj` is the name of my Django project (`django-admin startproject myproj .` in __Step 1__)
- `wsgi` is a reference to the `wsgi.py` file
- `application` is a variable in `wsgi.py`. 
- `myproject.wsgi:application` is merely a pythonic way to write a python to a variable inside a python module

Gunicorn does have additional configuration as we'll see later. 

For now, let's update our _Instructions List_:
- Python 3.8
- Virtual Environment
- path/to/my/venv/bin/pip install -r requirements.txt
- cd path/to/my/project/src
- path/to/my/venv/bin/gunicorn myproj.wsgi:application --bind "0.0.0.0:8080"


At this point, the _Instructions List_ just recaps what we did with our code. That's it.

That's how I think of Docker. Provide Docker with a list of instructions (aka a `Dockerfile`) that you need to run your code. These instructions can also include the _operating system_ you need, the programming language you need, or even the tool (like `postgresql` or `redis`). 

Now let's take a look at the actual `Dockerfile`


## Step 3: Docker's `Dockerfile` (For Python Web Apps & Django)


Converting our _Instructions List_ to a format Docker understand is this simple:

`Dockerfile`
```dockerfile
FROM python:3.8.3-slim
COPY . /app
WORKDIR /app
RUN python3 -m venv /opt/venv
RUN /opt/venv/bin/pip install -r requirements.txt
WORKDIR /app/src
CMD /opt/venv/bin/gunicorn myproj.wsgi:application --bind "0.0.0.0:8111"
```

Let's break down this `Dockerfile`

- `FROM python:3.8.3-slim` : this is the official Python 3.8.3 docker container image. We can inherit from this container so we can have the confidence that Python 3.8.3 will work nearly anywhere Docker is installed (assuming the underlying installation of Docker is not outdated).
- `COPY . /app` This is copying all files in `path/to/dev/folder/` from __Step 1__ _except_ the files/patterns that are listed in `.dockerignore`. `.dockerignore` follows the same behavior as [.gitignore](https://git-scm.com/docs/gitignore). `.dockerignore` is a simple way to make sure our Docker containers remain concise and remove items we don't need. 
- `WORKDIR /app` this just changes the directory we want our next commands to run. Use this instead of something like `RUN cd app`
- `RUN python3 -m venv /opt/venv` This will create a virtual environment for us in the `/opt` folder with the name `venv`. the `/opt` 


Assuming you know about running Django projects, the _Instructions List_ is simple and easy read. It's also simple and easy to modify to fit your needs. There's a problem though.  The _Instructions List_ *might* be the same on every machine and that's a big *might*. Naturally this *might* can be a big hurdle to overcome especially as we release code into production.

So what to do?

- Can we package our app as a binary and just run that? Yes and no. There's a number of problematic issues that arise here: security, size of the binary, redundant/inefficient code, compiled for the wrong system, and so on.
- Could we just develop on the system we want to run on production apps on? Yes and no. This *could* solve the issues that arise from this particular _Instructions List_ but what if we need to deploy another application with Python 3.10? Also, it's not always practical or cost effective for *everyone* to write code on the same system as what might be running in production
- Could we write a bunch of scripts to account for different operating systems / platforms? Yes and no. Once you do enough of these, they will start to become too cumbersome to manage especially as you scale. What's more, if the engineers/coders who write the scripts leave the project, your entire production is extremely vulnerable.


This is where Docker comes in. 


Docker makes sure it is the same.

Here's a simple `Dockerfile`:
```Dockerfile
FROM python:3.8.3-slim

# copy your local files to your
# docker container
COPY . /app

# update your environment to work
# within the folder you copied your 
# files above into
WORKDIR /app

# /opt: reserved for the installation of add-on application software packages.
# We'll use this to create & store our virtual environment

# Create a virtual environment in /opt
RUN python3 -m venv /opt/venv

# Install requirments to new virtual environment
# requirements.txt must have gunicorn & django
RUN /opt/venv/bin/pip install -r requirements.txt

RUN /opt/venv/bin/pip install pip --upgrade && \
    /opt/venv/bin/pip install -r requirements.txt && \
    chmod +x entrypoint.sh

# entrypoint.sh will be discussed later.
CMD [ "/app/entrypoint.sh" ]
```

That is a dead-simple `Dockerfile` that does almost exactly what our _Instructions List_ did before. It's now a bit more specific but the underlying concepts are still there. This `Dockerfile` is incomplete so we'll fix that in a bit.

First, let's create `entrypoint.sh` to run our `gunicorn` process with;

```bash
#!/bin/bash
APP_PORT=${PORT:-8000}
cd /app/src/
/opt/venv/bin/gunicorn --worker-tmp-dir /dev/shm myproj.wsgi:application --bind "0.0.0.0:${APP_PORT}"
```

Assuming our code and dependancies are installed, `CMD [ "entrypoint.sh"  ]` will but what we use to _run_ our Docker container image.

Let's break down each line:

- `APP_PORT=${PORT:-8000}` this allows us to set a different `PORT` value through environment variables (i'll show you below)
- `cd /app/src/` this is to change to our Django root (`/app/src`) where `manage.py` exists. `/app` is also our `WORKDIR` where the `src` folder is implied because of our Django code.
- `/opt/venv/bin/gunicorn` this is the gunicorn executable path from when we ran `RUN /opt/venv/bin/pip install -r requirements.txt` 
- `--worker-tmp-dir /dev/shm` this is an optional location for our workers' temporary directory. This is highly recommended if you deploy this container to DigitalOcean App Platform
- `myproj.wsgi:application` this is the path to our `application` declaration in Django. In this case, the django configuration folder is `myproj` where `wsgi.py` exists that includes the variable `application` by default.
- `--bind "0.0.0.0:${APP_PORT}"` this will bind the gunicorn process to run through our desired `PORT`. `0.0.0.0` is a designation to run on *any* IP address that may be declared. In other words, don't use `127.0.0.1` which is often used for development.




## Step 4: Docker Build

Assumes you have [docker installed](https://www.docker.com/products/docker-desktop), building a container with the Docker CLI is as simple as:

```
docker build -t djk8s -f Dockerfile .
```
Let's break down this command:

- `docker` is the CLI command for executing docker
- `build` is how we tell docker we want to build a container image
- `-t djk8s` In this case, we're _tagging_ this docker image. It's similar to naming a container image but allows for unique tag combinations for the same _image name_. For example `-t djk8s:v1` and `-t djk8s:c3183123` are both tagged versions of the image `djk8s`. The format is `-t IMAGE_NAME:TAG`. Keep in mind that if you run build multiple times with the same tag, it will override the previous version with the same matching tag(s). You can also provide multiple tags when you build such as `docker build -t djk8s:latest -t djk8s:commit1234` and so on.
- `-f Dockerfile` this is the default path to the `Dockerfile` you want to use to build your container image. Some projects require multiple `Dockerfile`s for different stages such as, `-f Dockerfile.dev` or `-f Dockerfile.prod-v2`.
- The trailing `.` means the context (aka folder) to run this command in. In this case, we're going to run the `-f Dockerfile` within the current folder. This has implications for the `COPY . /app` command within the `Dockerfile` as the `.` in the Dockerfile and the `docker build` command refer to the same exact local directory.

After you run `docker build ...`, it will build a container image in multiple stages. Those stages correspond to the parent image `FROM python` as well as the various steps in your `Dockerfile`.

This step can/should be automated on Github Actions (blog post coming soon) but it's important to be able to run this command on your local machine. It's also important to note that the image built from `docker build` will not automatically update itself when your code changes. When in doubt, build a new container image when code changes.



## Step 5: Docker Run
To run our container image, we'll do the following command:

```
docker run -p 8000:8000 djk8s
```
Let's break down this command
- `docker run` the docker command to run nearly *any* tagged docker image; especially public ones. In our case, we are using the tagged image we built in __Step 4__
- `-p 8000:8000` this exposes our `PORT`s. The first number `8000` is the port that Docker is exposing. This means we can go to `localhost:8000` and see our container image. The second number `8000` is mapped to what our `gunicorn` server is running on. Right now the numbers are the same but below we'll change them.
- `djk8s` is a container image that exists on our local machine. If it does not exist, Docker will look on [Docker Hub](https://hub.docker.com) for a public image. For example, running `docker run nginx` will download the [nginx](https://hub.docker.com/_/nginx) image to your local machine (if it's not already there) and run it.  Naturally you can use tags too such as `djk8s:c3183123` from __Step 4__'s explanation.

Docker's `docker run` command will only work on pre-built images; it does not build the images for you. If you're using your own custom container image, you must run `docker build` prior to `docker run`. If you ever change *any* code related to the code in the container image, you *must* rebuild and then run. 



## Step 6: Environment Variables & Docker

__Environment Variables In `Dockerfile`__
In our `Dockerfile` we can add environment variables like:

```Dockerfile
ENV my_key=my_value
```
Once you do that, your running image will have access to the value for `my_key` or whatever you put there. You can also consider using `build-args` in conjunction with `ENV` items.

`build-args` are as follows:

```Dockerfile

ARG MY_BUILD_ARG=default_value
ENV ENV_BUILD_ARG=$MY_BUILD_ARG
```
Using build args requires you to do something like `docker build -t djk8s -f Dockerfile . --build-arg MY_BUILD_ARG="abc123"`

You can verify `ARG` and `ENV` simply with:

```docker
RUN echo ${MY_BUILD_ARG}
RUN echo ${ENV_BUILD_ARG}
```

Personally, I would use environment variables sparingly in the `docker build` process especially if they should remain secrets.

Instead, use `docker run` to handle Environment variables.

### Environment Variables in `docker run` command

To use an environment variable, you can just use the `-e` flag like:

```
docker run -e PORT=8111 -e SECRET_KEY=abc123 -e ALLOWED_HOST=.djangopod.com ...
```
Each `-e` flag is a new environment variable.

You can also use a `.env` file (called dot env) with:

`.env`
```
PORT=8111
SECRET_KEY=abc123
ALLOWED_HOST=.djangopod.com
```

Then

```
docker run --env-file .env ...
```

Environment variables are the preferred method for secret keys as well as modifying the running environment without touching the code.

Let's take the example from __Step 5__:

```
docker run -p 8000:8000 djk8s
```
This command implies no environment variables are going to be added at runtime. Let's revisit what `-p 8000:8000` means by adding an environment variable that actually changes the port that gunicorn runs on (remember we set this up in __Step 3__ with the `entrypoint.sh` file based on `APP_PORT=${PORT:-8000}`):


```
docker run -e PORT=6545 -p 8000:6545 djk8s
```
What this command does for our project is:

- run our `djk8s` image
- set the environment `PORT` to `6545`. (you can see this with the shell commands `python -c "import os;print(os.environ.get('PORT'))"` or `echo $PORT`)
- Since `entrypoint.sh` runs `gunicorn` based on the environment's `PORT` variable, it will run the command:

```bash
/opt/venv/bin/gunicorn --worker-tmp-dir /dev/shm myproj.wsgi:application --bind "0.0.0.0:6545"
```
This means that our Django project will be running through gunicorn at port `6545` **inside** of the Docker container. 
- `-p 8000:6545` forwards our local port `8000` to the internal docker container port `6545` and thus exposing our Docker-running application. It's important to note that if you do *not* expose a port (either during `docker run` or in the `Dockerfile`), your container will *not* be accessible by any application outside of Docker.


## Step 7: Entering the Docker Container Command Line or `docker run` revisited.
So how do we enter into a docker container? With a virtual or remote machine, you can use `ssh`. What's the equivalent for Docker? That's what we'll talk about here.

There's two ways to use the container image command line (`/bin/bash`):

```
docker run -it djk8s /bin/bash
```
This command will work anytime just as long the `djk8s` image has been built and is public/accessible (aka not a hidden/private container image).

The other command to enter is below. 
```
docker exec -it CONTAINER /bin/bash
```
This command will only run if you have the `ID` of the container that is _currently running_. You can find this `ID` by running the command `docker ps`. 


## Step 8: A minimal Docker Compose Example

Docker Compose answers a few questions for us:
- Can I run a Docker container based on a configuration file (instead of writing all the flags like in __Step 6__)?
- What if I need a Docker-based database for the above Django project?
- What if I need to add an nginx container?
- What if I need to add a worker container?
- How do I attach storage volumes? 

Let's take a look at a Docker compose file for the above project:

`docker-compose.yaml`
```yaml
version: '3.9'
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    image: djk8s-compose:v1
    environment:
      - PORT=8023
    env_file:
      - .env
    ports:
      - "8001:8023"
```

This file sums up essentially everything we've covered since __Step 4__ but it's more than just a file. You can now run:

```
docker compose up --build
```
This will both *build* our container image and *run* it based on the arguments you added above.  As you might imagine, our docker project can get significantly more complex but this is a good start to understanding the benefit of using Docker with *any* application but especially with Django.