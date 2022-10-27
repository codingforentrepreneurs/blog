---
title: Hello Linux: Celery &amp; Supervisor
slug: hello-linux-celery-supervisor

publish_timestamp: March 17, 2019
url: https://www.codingforentrepreneurs.com/blog/hello-linux-celery-supervisor/

---


_This is the eight post of a many part series. [This post](https://www.codingforentrepreneurs.com/blog/hello-linux/) is the starter post for the whole series._

Celery is a Python package that integrates with Redis (or RabbitMQ) to help offload various functions from your core python app. This is very useful for things like generating repots, processing data, and even scheduling periodic tasks.

Supervisor is a process manager (as you likely know) that will ensure that Celery is running along side of your app.

*********
**Requirements**

Be sure to complete
- [Git Push Local Code to Live Linux Server](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Virtual Environment in Working Directory](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- [Setup Gunicorn & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor)
- [Nginx & UFW Firewall](https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall)
- [Custom Domain & Https with Let's Encrypt](https://www.codingforentrepreneurs.com/blog/custom-domain-and-https-with-lets-encrypt/)
- [Install Redis](https://www.codingforentrepreneurs.com/blog/hello-linux-install-redis)

**Our Live Server**
- **Ubuntu 18.04**

*********


#### 1. Celery + Redis + Django
We've covered this in detail before in this guide in [this Guide](https://www.codingforentrepreneurs.com/blog/celery-redis-django). Naturally ignore the **heroku** portion if you intend to do the rest of this guide.


#### 2. SSH into your server

```
ssh root@104.248.231.241
```
Replace `root@104.248.231.241` with your user / ip


#### 3. Install & Start Supervisor
Assuming you did [this post](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor) recently, you shouldn't need to do this.
```
sudo apt-get update -y

sudo apt-get install supervisor -y 

sudo service supervisor start
```


#### 4. Install Celery in our Working Directory's Virtualenv


Find the Virtualenv bin that you created in [this post](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)

Mine is `/var/www/myproject/bin/`


```
/var/www/myproject/bin/python -m pip install "celery[redis]" redis 
```
> Fun note; if you already installed redis/celery locally, updated requirements.txt, did a push, there's a good chance your celery is _already_ installed.

#### 5. Verify Celery works

```
/var/www/hello_linux/bin/celery -A
```

**You should see**
```
usage: celery [-h] [-A APP] [-b BROKER] [--result-backend RESULT_BACKEND]
              [--loader LOADER] [--config CONFIG] [--workdir WORKDIR]
              [--no-color] [--quiet]
celery: error: argument -A/--app: expected one argument
```

**You should not see**:
```
-bash: /var/www/myproject/bin/celery: No such file or directory
```


#### 6. Create a Supervisor Process
Almost the same exact instructions from [here](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor)
With supervisor, we can run step 4 automatically, restart it if it fails, create logs for it, and start/stop it easily. 

Basically, this ensures that our web server will continue to run if you push new code, server reboots/restarts/goes down and back up, etc. 

Of course, if a catastrophic error occurs (or if bad code is in your Django project) then this proceess might fail as well.


All supervisor proccesses go in:
```
/etc/supervisor/conf.d/
```
So, if you ever need to add a new process, you'll just add it there.

Let's create our project's celery configuration file for supervisor.

```
touch /etc/supervisor/conf.d/myproject-celery.conf
```

Now you should see:
```
$ ls -al /etc/supervisor/conf.d/
myproject-celery.conf
```

Now, let's add the base settings:

```
[program:myproject_celery]
user=root
directory=/var/www/myproject/src/
command=/var/www/myproject/bin/celery -A myproject worker -l info
 
autostart=true
autorestart=true
stdout_logfile=/var/log/myproject/celery.log
stderr_logfile=/var/log/myproject/celery.err.log"
```

Let's break it down line by line.


`[program:myproject_celery]`: the `myproject_celery` is the name of the supervisor process. Naturally, `myproject_celery` is an arbitrary name. Once we complete *Step 5*, you'll be able to run the following commands:

- `sudo supervisorctl status myproject_celery`
- `sudo supervisorctl start myproject_celery`
- `sudo supervisorctl stop myproject_celery`
- `sudo supervisorctl restart myproject_celery`


`user=root`: Set the user you want to allow access to this process. This must be set.

`directory=/var/www/myproject/src/` This is the working directory for your project (notice mine has `src` at the end)

`command=...` This is the command you want your process to run. Our command is:
    - `/var/www/myproject/bin/celery` This is the recently installed celery bash command
    - `-A myproject`  This is a celery argument for where your project's celery app is located. Mine is in `celery.py` in my main Django conf directory. (We'll add this in the next step). In my case it's `src directory -> cfehome module -> celery module -> celery application`
    - `-l info` This just means "log level is info" so we just get general info about this app. Check the celery [docs](http://docs.celeryproject.org/en/latest/reference/celery.bin.worker.html#cmdoption-celery-worker-l) for more log levels. 
    

`autostart` Do you want this process to auto start? We do.

`autorestart` Do you want this process to auto restart if things fail? We do.

`stdout_logfile` and `stderr_logfile` are locations for log files. You might need to run `mkdir /var/log/myproject/` to ensure these files are actually added. These log files are great for diagnosing errors.




#### 7. Update Supervisor

```
supervisorctl reread
supervisorctl update
```

#### 8. Check Our Supervisor Program/Process Status
As we mentioned when we crated the `myproject_guincorn` supervisor program, we can now do:
```
sudo supervisorctl status myproject_celery
```

A few other useful commands (again):
- `sudo supervisorctl start myproject_celery`
- `sudo supervisorctl stop myproject_celery`
- `sudo supervisorctl restart myproject_celery`




#### Wrap Up

**We now have**
- [Git Setup for Push/Pulling Code From Local to Production](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Working Directory with a Virtual Environment](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- [Setup Gunicorn & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor/) 
- [Nginx & UFW Firewall](https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall)
- [Custom Domain + HTTPs with Let's Encrypt](https://www.codingforentrepreneurs.com/blog/custom-domain-and-https-with-lets-encrypt/)
- Celery, Redis, & Supervisor (this post)
