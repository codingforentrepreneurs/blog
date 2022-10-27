---
title: Remote Redis Servers for Development
slug: remote-redis-servers-for-development

publish_timestamp: Sept. 14, 2021
url: https://www.codingforentrepreneurs.com/blog/remote-redis-servers-for-development/

---


### Using Virtual Machines for Redis during Development

In this guide, I am going to show you how to run a remote redis server in nearly any virtual machine. The goal is to do as *little* as possible in configuring our instance while gaining all the benefits of using a near production-ready instance of Redis.


#### Step 1. Boot up a Virtual Machine on [Linode](https://www.linode.com/cfe) (such as a Nanode 1 GB at $5/mo)
A couple things to note:
- Debian 10 is recommended (if you know what you're doing you'll ignore me here anyways)
- Use a location near your location 
- Make note of your `ip_address` that is provisioned for you.

#### Step 2. SSH into your Virtual Machine 
```
ssh root@yourip
```
> Such as `ssh root@23.239.0.131`

#### Step 3. Install Docker
This is by far the easiest way to install docker on a virtual machine.
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

#### Step 4. Run Redis

```
docker run --restart always -d -p 6379:6379 redis 
```
The keys are:

- `docker` is installed
- `run` is the command to run an instance
- `--restart always` this means that this docker container will run again if your system reboots
- `-d` means your container runs in detached mode (ie the background)
- `-p 6379:6379` this does 2 things: 1, exposes port `6379` to the oustide world and 1, maps the exposed `6379` port to the internal `6379` port. Do *not* change these ports unless you know how to add additional config for redis.



#### Step 5. Verify Redis (anywhere)

```
redis-cli -h yourip ping
```

- `redis-cli` is already on my machine; this is not the only way to interact with redis
- `-h yourip` is denoting the host `ip_address` from the system you set above.
- `ping` is a simple command to ping the redis server. 

Do you see `PONG`? That means your Redis server is running and ready to go. Simple enough?
