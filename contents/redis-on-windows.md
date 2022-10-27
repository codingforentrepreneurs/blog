---
title: Redis on Windows
slug: redis-on-windows

publish_timestamp: Sept. 15, 2020
url: https://www.codingforentrepreneurs.com/blog/redis-on-windows/

---


Redis is a very popular data store used across all kinds of web apps. Redis, unfortunately, is not natively supported by Windows. Not to worry, you can still use Redis it just takes a few extra steps. 

Here are the options we'll explore:

- Using Docker
- Using Memurai

*Are you a macOS or Linux user? [Check this guide](https://www.codingforentrepreneurs.com/blog/install-redis-mac-and-linux)*


Personally, I like using Docker since it gives us the official version of Redis baked in along with standard redis configuration (if you need it). This version or any configuration you do can definitely be used in production.

Memurai is a service I found that might be a great solution for using Redis on Windows. I haven't used it much myself but it looks promising.

Let's get it all setup.

## Using Docker

Naturally, you need to install [docker desktop]() on your windows 10. There's a huge caveat that might push you towards using Memurai:

**Docker requires you to have Windows 10 professional**

Windows 10 professional allows for the OS virtualization that is just not provided in Windows 10 home. 

- Are you new to docker? Check out [this basic intro](https://www.codingforentrepreneurs.com/blog/simple-docker)
    

### 1. Verify Docker
Open PowerShell and run:
```
> docker --version
Docker version 19.03.12, build 48a66213fe
```


### 2. Create a project directory

```
> cd Dev # or mkdir Dev
> mkdir docker-redis
> echo "" > DockerFile
```

### 3. Update (or create) a `Dockerfile`

```
FROM redis
CMD [ "redis-server"]
```

> You don't have to build the redis image yourself but I prefer to so I can make changes easily and it's already prepared for it.


### 4. Build our docker image:

```
> docker build -t "cfe-redis" .
```
> Personally, I like adding commands like `docker build -t "cfe-redis" .` to a `build.ps1` file so I can just call `./build.ps1` if I ever want to build it again in the future.


### 5. Run our docker image:

```
> docker run -it --rm -p 6379:6379 "cfe-redis"
```

A couple things to note:

`-it --rm` is so this docker container only runs when we want it to and the container will be removed after we're done running it.

`-p 6379:6379` is exposing the port `6379` on our local system AND in our docker container. 

`cfe-redis` is merely the tag name of the image we created in the build process.

> Just like with `build.ps1`, make a `run.ps1` file containing `docker run -it --rm -p 6379:6379 "cfe-redis"` as a shortcut to running this container.

### 6. Verify Docker connection

On Linux and Mac, you can just run `redis-cli ping` but, since we're using docker, we don't have the `redis-cli` command available on our system. 

We need another way to test. Since I use `redis` in my [Python](https://cfe.sh/t/python) projects, I'll create a simple python program to ensure our non-container items can connect to `redis`. 

> I already have Python 3.8 installed. If you don't, check out [this guide](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows).

##### Create virtual environment
```
> cd ~/Dev/docker-redis
> python -m venv .
```
> You can use `pipenv` as well. I'm just keeping things simple by using the built-in virtual environment manager


##### Activate virtual environment

```
C:\> .\Scripts\Activate 
(docker-redis) C:\>
```

##### Run `pip install redis`
This command does not install the redis server; it just installs a redis connector that we can use in python.

```
(docker-redis) C:\> pip install redis 
```

#### Create `ping_redis.py`
```python
#ping_redis.py

import redis
r = redis.Redis(host='localhost', port=6379, db=0)
setter = r.set('foo', 'bar')
getter = r.get('foo')

print(setter, getter)
```

Notice the `port=6379` I used? This is the default redis port and it matches to what I exposed above.

#### Run `ping_redis.py`

```
(docker-redis) C:\> python ping_redis.py
```

What result do you have?
- `True b'bar'` -- this means redis is running correctly
- `No connection could be made because the target machine actively refused it.` -- this means your version of redis is either (1) not running or (2) running incorrectly or (3) you got the port number wrong. 


### 7. Future running

Now, whenever you need to use `redis` in your projects. Just run:

```
docker run -it --rm -p 6379:6379 "cfe-redis"
```
Anywhere on your system.





## Using [Memurai.com](https://www.memurai.com/)

I haven't used memurai much myself but it seems promising. Let's see

### 1. Download Developer Edition [here](https://www.memurai.com/get-memurai)


### 2. Run downloaded installer

### 3. Agree to all defaults *except* port.

For the `port`, I'll use `6380`.

> I'm leaving `6379` open. It's the default redis port and I'm not 100% convinced I'll use `memurai` in the future. I'm positive I'll use Docker and so I'll leave `6379` open for docker versions of redis like above.

### 4. Verify in Python -- Create a virtual environment.
`memurai` should be running by default after you installed it so we need to verify.

Assuming you did not do the docker portion above, let's create a new virtual environment to test redis is running via memurai. Open up `PowerShell` and run:

```
> mkdir Dev
> cd Dev
> mkdir memurai-redis
> cd memurai-redis
```

> I already have Python 3.8 installed. If you don't, check out [this guide](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows).

```
> python -m venv .
```

##### Activate virtual environment

```
C:\> .\Scripts\Activate 
(memurai-redis) C:\>
```

##### Run `pip install redis`
This command does not install the redis server; it just installs a redis connector that we can use in python.

```
(memurai-redis) C:\> pip install redis 
```

#### Create `ping_redis.py`
```python
#ping_redis.py

import redis
r = redis.Redis(host='localhost', port=6380, db=0)
setter = r.set('foo', 'bar')
getter = r.get('foo')

print(setter, getter)
```

Notice the `port=6380` I used? This is the redis port I used during the `memurai` installation.

#### Run `ping_redis.py`

```
(memurai-redis) C:\> python ping_redis.py
```

What result do you have?
- `True b'bar'` -- this means redis is running correctly
- `No connection could be made because the target machine actively refused it.` -- this means your version of redis is either (1) not running or (2) running incorrectly or (3) you got the port number wrong. 



### 5. Future running
If you installed the defaults, memurai should start with your system startup just remember which port (`6380`) you used for the memurai redis.


## Now what?

Now you can use anything that requires redis on your local system. For us, we have a series that uses Python [Celery](https://docs.celeryproject.org/en/stable/getting-started/introduction.html) for scheduling, delaying, and offloading all kinds of tasks. You can also use redis to cache your web applications (like in Django).

**Did you find another way to install Redis on windows? Please let us know in the comments.**
