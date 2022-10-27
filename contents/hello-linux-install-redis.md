---
title: Hello Linux: Install Redis
slug: hello-linux-install-redis

publish_timestamp: March 17, 2019
url: https://www.codingforentrepreneurs.com/blog/hello-linux-install-redis/

---

<div class='alert alert-success'>This is the seventh post of a many part series. <a href='https://www.codingforentrepreneurs.com/blog/hello-linux/'>This post</a> is the starter post for the whole series.</div>

Redis is a datastore for "queued messages" which just means it's like a list of items that can be consumed

Looking for [local redis installation](https://www.codingforentrepreneurs.com/blog/celery-redis-django)?

*********
**Requirements**

Be sure to complete
- [Git Push Local Code to Live Linux Server](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Virtual Environment in Working Directory](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- [Setup Gunicorn & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor)
- [Nginx & UFW Firewall](https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall)
- [Custom Domain & Https with Let's Encrypt](https://www.codingforentrepreneurs.com/blog/custom-domain-and-https-with-lets-encrypt/)

**Our Live Server**
- **Ubuntu 18.04**

*********



#### 1. SSH into your server

```
ssh root@104.248.231.241
```
Replace `root@104.248.231.241` with your user / ip


#### 2. Install Redis
```
sudo apt update -y
sudo apt install redis-server -y
```

#### 3. Test Installation
```
redis-cli ping
```

You should see:
```
$ redis-cli ping
pong
```

By default, your redis host is: `redis://localhost:6379`

#### 4. Update Redis to Systemd
If you're using Ubuntu, change `supervised no` to `supervised systemd` in `sudo nano /etc/redis/redis.conf`

So in `sudo nano /etc/redis/redis.conf`, it has this setting:
```
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised systemd
```

#### 5. Reload & Restart

```
systemctl daemon-reload
sudo systemctl restart redis.service
```


#### Wrap Up

**We now have**
- [Git Setup for Push/Pulling Code From Local to Production](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Working Directory with a Virtual Environment](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- [Setup Gunicorn & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor/) 
- [Nginx & UFW Firewall](https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall)
- [Custom Domain + HTTPs with Let's Encrypt](https://www.codingforentrepreneurs.com/blog/custom-domain-and-https-with-lets-encrypt/)
- Redis Installed (this post)


**Still coming, but optional**
- Task Queues with Celery & Redis