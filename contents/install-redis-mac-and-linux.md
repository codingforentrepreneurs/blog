---
title: Redis on Mac &amp; Linux
slug: install-redis-mac-and-linux

publish_timestamp: Sept. 15, 2020
url: https://www.codingforentrepreneurs.com/blog/install-redis-mac-and-linux/

---

Redis is a very popular data store used across all kinds of web apps it's supported natively by Mac and Linux operating systems. Heck, you can even use Docker to run Redis.

The purpose of this guide is to setup Redis so we can use in our Python applications including Django, Flask, Fastapi, Celery, and pure python apps.

*Are you a windows user? Check out [this redis install guide](https://www.codingforentrepreneurs.com/blog/redis-on-windows/)*

*Do you need to use Redis on Docker? The [windows install guide](https://www.codingforentrepreneurs.com/blog/redis-on-windows/) shows you how.*


## On Mac

#### 1. Install [Homebrew](https://brew.sh/): 
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

```

#### 2. Install redis

```
brew install redis
```

> Every once and a while run `brew upgrade redis`

#### 3. Start / Stop Redis


```
brew services start redis
```

Starting redis this way turns redis into a background service. You can easily stop redis with:

```
brew services stop redis
```

#### 4. Verify redis is running:

```
% redis-cli ping
```

What result do you see? 
- `PONG`  -- great, redis is working and ready.
- `Could not connect to Redis at 127.0.0.1:6379: Connection refused` -- this means that: (1) you did not install redis correctly or (2) redis is not running.




## On Linux (using Ubuntu)
Below was adapted from the [production-ready guide](https://www.codingforentrepreneurs.com/blog/hello-linux-install-redis) in our series [Hello Linux](https://www.codingforentrepreneurs.com/blog/hello-linux/)

#### 1. Update System
```
sudo apt update
```

#### 2. Install `redis-server`

```
sudo apt install redis-server
```

#### 3. Update configuration

```
sudo nano /etc/redis/redis.conf
```
Update `supervised` to the following:

```
supervised systemd
``` 
Save and exit nano.

Restart running service:

```
sudo systemctl restart redis.service
```


#### 4. Verify installation:

```
$ redis-cli ping
```

What result do you see? 
- `PONG`  -- great, redis is working and ready.
- `Could not connect to Redis at 127.0.0.1:6379: Connection refused` -- this means that: (1) you did not install redis correctly, (2) redis is not running, or (3) the system service did not start (check with `sudo systemctl status redis`)


## Now what?

Now you can use anything that requires redis on your local system. For us, we have a series that uses Python [Celery](https://docs.celeryproject.org/en/stable/getting-started/introduction.html) for scheduling, delaying, and offloading all kinds of tasks. You can also use redis to cache your web applications (like in Django).

**Did you find another way to install Redis on windows? Please let us know in the comments.**