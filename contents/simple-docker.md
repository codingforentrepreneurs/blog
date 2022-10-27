---
title: Simple Docker Container Tutorial
slug: simple-docker

publish_timestamp: Sept. 18, 2019
url: https://www.codingforentrepreneurs.com/blog/simple-docker/

---


Docker is all the rage. I cover it more in depth on [this post](https://www.codingforentrepreneurs.com/blog/django-docker-production-heroku/) but I wanted to show you how to use the absolute basics of Docker. 

Let's get started.

#### Requirements
- Windows 10 Professional [docs](https://docs.docker.com/docker-for-windows/install/) (seriously; you need Virtual Machine support and Windows 10 Pro has it)
- macOS [docs](https://docs.docker.com/docker-for-mac/install/)

#### 1. Create a Docker Hub account [here](https://kirr.co/ud0sxn)

#### 2. Install Docker Desktop [here](https://kirr.co/36n3i1)

#### 3. After install, be sure to login to Docker Desktop

#### 4. Verify Installation

Open Terminal (Mac / Linux) / PowerShell
```
docker --version
```
You should see something like:
```
Docker version 19.03.2, build abc123
```


#### 5. Your first Docker project

Find a place for your project:
```
cd path/to/your/dev/folder
```

Now that we're here, let's create our first `Dockerfile`
```
touch Dockerfile
```
> `Dockerfile` is the convention and best practice name. No extension.



#### 6. Dockerfile 101
A docker file is a standardized way for Docker to build your _Docker Image_. A _Docker Image_ is essentially an _operating system_ and typically a _linux_ one. The _Docker Images_ can be moved around easily and repurposed for a variety of use cases.

Let's edit our `Dockerfile` now:

```
# In path/to/your/dev/folder/Dockerfile
# Base Image
FROM python:3.6

RUN apt-get update && apt-get install -y --no-install-recommends \
        python3-setuptools \
        python3-pip \
        python3-dev \
        python3-venv \
        git \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 8000

CMD python -c "print('hello world')"
```
What's going on here?

Remember, capitalization is very important.

##### `FROM` 
`FROM` is going to build your image from another pre-existing image. This can get pretty advanced so for now, we just use a good `python 3.6` image. 

##### `RUN`
- `RUN` is a command that allows you to do any bash shell command you'd do normally. In our case, we just do some basic system updates and basic installs.

##### `EXPOSE`
`EXPOSE` allows your docker image to have a `port` or posts exposed to outside the image. This is important for web applications and software you want to receive requests.

##### `CMD`
`CMD` this is the final command your docker image will run. It's typically reserved for something like running a web application.

##### `COPY`
`COPY` copy is another command we haven't yet added to our `Dockerfile`. This will allow you to copy local files to your Docker image.

#### 6. Commands for Docker

##### `$ docker build -t <your-tag> -f Dockerfile .`
```
$ cd path/to/your/dev/folder
$ ls
Dockerfile
```
Now run the build:
```
$ docker build -t hello-world -f Dockerfile .
```

This will build your actual image. It might take awhile (5-15 minutes) depending a number of factors. After you build it 1 time, the future builds *might* not take as long :).

- `-t` portion means "tag" and you can add your _own tag name_ I used `hello-world` since this might be your first time using Docker. When in doubt, include a tag.
- `-f` is the path to the `Dockerfile` you're going to use to build your image.


##### `$ docker images`
Running this command anywhere on your system, assuming Docker Desktop is running, will show you all of the images your system has. It will also have the `python 3.6` image we grabbed in our Dockerfile. Pretty cool right?

> Pro tip, to remove all of these images just run `docker rm $(docker ps -a -q)` then `docker rmi $(docker images -q)`

##### `$ docker ps -a`
This will show you all of the containers that are currently running. After we build our image, we need to run it as a container. 


##### `$ docker run -it <your-tag> `
This creates a "container" from your tagged image. In other words, a **container is an instance of an image**.


Following the example above, you'll run:

```
$ cd path/to/your/dev/folder
$ ls
Dockerfile
```
Now run the build:
```
$ docker run -it hello-world
hello world
```
If you copied our `Dockerfile` exactly, you should see the response from `docker run -it hello-world` as simply `hello world`. This means your image is working!


> Tip: You can actually run `docker run -it <your-tag>` anywhere on your system and it should run your project.

##### `$ docker run -d -it <your-tag> `
If you add `-d` it will run in the background.
This has it's own section because running your container in the background can cause conflicts with other containers using the same `port`. 

To stop a background running container, you can first list all running containers:

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
5966a461fa9d        hello-world         "/bin/sh -c 'python …"   3 seconds ago       Exited (0) 1 second ago                       thirsty_cori
```

##### `$ docker stop <your-container-id> `
Above we ran `$ docker ps -a`. This showed us our docker container had an id of `5966a461fa9d`. We can now stop and remove that container

```
docker stop 5966a461fa9d
docker rm 5966a461fa9d
```


#### 6. Bash Shell in a Docker Image
```
docker run -it <your-tag> /bin/bash
```

That's it. You now have access to your docker container's bash shell.


#### 7. Now let's add some code.


<br/><br/>
<a name='video' id='video'></a>
<iframe width="560" height="315" src="https://www.youtube.com/embed/7S73WERRqO4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
