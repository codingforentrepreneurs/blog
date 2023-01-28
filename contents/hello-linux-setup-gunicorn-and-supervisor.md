---
title: Hello Linux: Gunicorn &amp; Supervisor
slug: hello-linux-setup-gunicorn-and-supervisor

publish_timestamp: March 14, 2019
url: https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor/

---


_This is the fourth post of a many part series. [This post](https://www.codingforentrepreneurs.com/blog/hello-linux/) is the starter post for the whole series._

Guincorn is our production WSGI server for running Django or Python web apps in a production environment.

Supervisor is a process manager to ensure Gunicorn (among other things) starts, restarts, and runs when we need it to.

*********
**Requirements**

Be sure to complete
- [Git Push Local Code to Live Linux Server](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Virtual Environment in Working Directory](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)

**Our Live Server**
- **Ubuntu 18.04**

*********

#### 1. SSH into your server

```
ssh root@104.248.231.241
```
Replace `root@104.248.231.241` with your user / ip



#### 2. Install & Start Supervisor

```
sudo apt-get update -y

sudo apt-get install supervisor -y 

sudo service supervisor start
```


#### 3. Install Gunicorn in our Working Directory's Virtualenv

Find the Virtualenv bin that you created in [this post](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)

Mine is `/var/www/myproject/bin/`


```
/var/www/myproject/bin/python -m pip install gunicorn
```

#### 4. Verify Gunicorn works

```
/var/www/myproject/bin/gunicorn --workers 3
```

**You should see**
```
usage: gunicorn [OPTIONS] [APP_MODULE]
gunicorn: error: No application module specified.
```

**You should not see**:
```
-bash: /var/www/myproject/bin/gunicorn: No such file or directory
```

So how many workers should we use? A rule of thumb is to do `(2 * num_cores) + 1`

How do you get the number of cores your Linux system has? 
```
grep -c ^processor /proc/cpuinfo
```
Let's say that returns `2`, you'd have 5 workers `(2 * 2) = 4 + 1 = 5`


#### 5. Create a Supervisor Process
With supervisor, we can run step 4 automatically, restart it if it fails, create logs for it, and start/stop it easily. 

Basically, this ensures that our web server will continue to run if you push new code, server reboots/restarts/goes down and back up, etc. 

Of course, if a catastrophic error occurs (or if bad code is in your Django project) then this process might fail as well.


All supervisor processes go in:
```
/etc/supervisor/conf.d/
```
So, if you ever need to add a new process, you'll just add it there.

Let's create our project's gunicorn configuration file for supervisor.

```
touch /etc/supervisor/conf.d/myproject-gunicorn.conf
```

Now you should see:
```
$ ls -al /etc/supervisor/conf.d/
myproject-gunicorn.conf
```

Now, let's add the base settings:

```
[program:myproject_gunicorn]
user=root
directory=/var/www/myproject/src/
command=/var/www/myproject/bin/gunicorn --workers 3 --bind unix:myproject.sock cfehome.wsgi:application
 
autostart=true
autorestart=true
stdout_logfile=/var/log/myproject/gunicorn.log
stderr_logfile=/var/log/myproject/gunicorn.err.log
```

Let's break it down line by line.


`[program:myproject_gunicorn]`: the `myproject_gunicorn` is the name of the supervisor process. Naturally, `myproject_gunicorn` is an arbitrary name. Once we complete *Step 5*, you'll be able to run the following commands:
    - `sudo supervisorctl status myproject_gunicorn`
    - `sudo supervisorctl start myproject_gunicorn`
    - `sudo supervisorctl stop myproject_gunicorn`
    - `sudo supervisorctl restart myproject_gunicorn`


`user=root`: Set the user you want to allow access to this process. This must be set.

`directory=/var/www/myproject/src/` This is the working directory for your project (notice mine has `src` at the end)

`command=...` This is the command you want your process to run. Our command is:
    - `/var/www/myproject/bin/gunicorn` This is the recently installed gunicorn bash command
    - `--workers 3`  This is a gunicorn argument for the number of workers
    - `--bind unix:myproject.sock` This creates  a socket for your gunicorn to bind to within your `directory` you set above.
    - `cfehome.wsgi:application` Where is your actual application? Django projects typically have a `wsgi.py` file that declares an `application`. This is the path to that exact declaration. In my case it's `src directory -> cfehome module -> wsgi module -> application`

`autostart` Do you want this process to auto start? We do.

`autorestart` Do you want this process to auto restart if things fail? We do.

`stdout_logfile` and `stderr_logfile` are locations for log files. You might need to run `mkdir /var/log/myproject/` to ensure these files are actually added. These log files are great for diagnosing errors.


#### 6. Update Supervisor

```
supervisorctl reread
supervisorctl update
```

#### 7. Check Our Supervisor Program/Process Status
As we mentioned when we created the `myproject_gunicorn` supervisor program, we can now do:
```
sudo supervisorctl status myproject_gunicorn
```

A few other useful commands (again):
- `sudo supervisorctl start myproject_gunicorn`
- `sudo supervisorctl stop myproject_gunicorn`
- `sudo supervisorctl restart myproject_gunicorn`




#### Wrap Up

**We now have**
- [Git Setup for Push/Pulling Code From Local to Production](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Working Directory with a Virtual Environment](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- Setup Gunicorn & Supervisor (this post)

**Now we need to**
- [Setup Nginx and Firewall for Running Live Django Project](https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall)
