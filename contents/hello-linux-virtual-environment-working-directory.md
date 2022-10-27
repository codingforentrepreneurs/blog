---
title: Hello Linux: Virtual Environment in Working Directory
slug: hello-linux-virtual-environment-working-directory

publish_timestamp: March 14, 2019
url: https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory/

---


> This is the second part of a many part series. [This post](https://www.codingforentrepreneurs.com/blog/hello-linux/) is the starter post for the whole series.

We use a virtual environment to ensure our project's dependencies will remain consistent from local to production and back. 


*********
**Basic Requirements**

**Completion of [Git Setup](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)**
- In the [git setup post](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server), we create a working directory on our production server. So yeah, definitely do that.


**Live Server Details**
- **Ubuntu 18.04** is what we use here but not required.
- **Other Linux Distro?** Check how to install git, most/all other things are the same.

*********


#### 1. SSH into your server

```
ssh root@104.248.231.241
```
Replace `root@104.248.231.241` with your user / ip


#### 2. Install `virtualenv` Globally

Why `virtualenv` and not `pipenv`?

When you create a virtual environment with `pipenv` it's destination is unpredictable (certainly a flaw of `pipenv`) where using `virtualenv` the path is designated by us.

Now, when we do things manually you can use either. But with a setup script. Much like the one you see on [this repo](https://github.com/codingforentrepreneurs/Hello-Linux), `virtualenv` is much better suited for our needs.

```
sudo apt-get update -y

sudo apt-get install python3-pip python3-dev -y

sudo python3 -m pip install virtualenv
```

#### 3. Navigate to your Working Directory

From the [git setup post](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server), our working directory is in `/var/www/myproject/`

```
cd /var/www/myproject/
```


#### 4. Initialize a Virtual Environment
```
virtualenv -p python3.6 .
```
So, as you might be able to tell, we can know exactly where this virtual environment path is. Make note of it because we'll **100%** be using it later.

Here's what you should see:
```
$ cd /var/www/myproject/
$ ls
bin  include  lib
```

You'll want to keep note of `/var/www/myproject/bin/` because you can do `/var/www/myproject/bin/python -m pip install django` which will install django to that directory's virtual environment. 


#### 5. Update your `git` `post-receive` hook.
Assuming you completed [this post](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server), you should do this:

```
cd /var/repo/myproject.git/hooks/
nano post-receive
```

In `post-receive`:
```
git --work-tree=/var/www/myproject/ --git-dir=/var/repo/myproject.git/ checkout -f


/var/www/myproject/bin/python -m pip install -r /var/www/myproject/src/requirements.txt
```

So `/var/www/myproject/src/requirements.txt` assumes that your project will have a `src` folder with a valid python `requirements.txt` file.

This will do a `auto-install` of all requirements after you run a push to this server.


#### 6. Now we can test a real project to our server. 

First, exit your server:
```
exit
```

Now, let's navigate to a place for our local project

Open terminal (or PowerShell for Windows)

```
cd ~/Dev/myproject
```

Remove previous local git repo (if there)
```
rm -rf .git
```

Clone working blank project
```
git clone https://github.com/codingforentrepreneurs/CFE-Blank-Project .
```

Remove that project's git repo (it's better for us to have a blank one)
```
rm -rf .git
```

Initial Git and add our Live Server's repo.
```
git init
git remote add live ssh://root@104.248.231.241/var/repo/myproject.git
```
Run a pull, if needed
```
git pull live master
```

Create and activate your local virtual environment
```
virtualenv -p python3.6 .
source bin/activate
```
Run local installs
```
(myproject) $ cd src
(myproject) $ pip install -r requirements.txt
(myproject) $ python manage.py runserver
```
Do you have errors? If so, please add a comment below on this post.

No errors? Do a commit and push...
```
git add --all
git commit -m "Added Django Project"
git push live master
```

Now you can go to your live server's working directory to ensure we actually have our Django code there.

SSH into your server
```
$ ssh root@104.248.231.241
```
Navigate to your working project directory and verify it has a `src` directory now (it might have other files too)
```
$ cd /var/www/myproject/
$ ls
bin lib include src
```



#### Wrap Up

**We now have**
- [Git Setup for Push/Pulling Code From Local to Production](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- Working Directory setup with a Virtual Environment (this post)

**Now we need to**
- [Setup our production PostgreSQL Database](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
