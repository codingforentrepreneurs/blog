---
title: Install Django on Linux Ubuntu
slug: install-django-on-linux-ubuntu

publish_timestamp: July 26, 2017
url: https://www.codingforentrepreneurs.com/blog/install-django-on-linux-ubuntu/

---


# Install Django & Virtualenv on Linux Ubuntu with PIP
> Looking for [windows installation](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows/)?
> Looking for [mac installation](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux)?

*Django* is a popular web development framework written in Python. 

*Virtualenv*, or Virtual Environments, is a tool to keep the dependencies required by different projects in separate places, by creating virtual Python environments for them. Basically, if your project has different software versions (and it will) virtualenv keep them nice and safe.

*PIP*, or Python Package Installer, allows you to install all types of python-related software (and code) include Django, Virtual environments (virtualenv), Python Requests, and more.


==========

Open up the **Terminal** Application and start typing the following:


### 1 - Install Python Tools
```
$ sudo apt-get update

$ sudo apt-get upgrade

$ sudo apt-get install python-pip python-virtualenv python-setuptools python-dev build-essential  python3.6
```

### 2 - Install and Create a Virtualenv

```
$ sudo pip install virtualenv 

$ mkdir ~/Dev

$ cd ~/Dev

$ mkdir venv && cd venv

$ virtualenv -p python3.6 .
```

### 3 - Activate Virtualenv

```
$ cd ~/Dev/venv/ #created above

$ source bin/activate

```
If activated correctly, you should see:

```
(venv) $
```

Check python version:

```
(venv) $ python --version #should return Python 3.6
```

To deactivate:
```
(venv) $ deactivate

```

To reactivate:
```
$ /path/to/your/virtual/env/
$ source bin/activate
```


### 4 - Install Django & Start New project

```
$ /path/to/your/virtual/env/here/
$ source bin/activate
(here) $ pip install django==1.11.3

(here) $  mkdir src && cd src

(here) $ django-admin.py startproject cfehome .

(here) $ python manage.py migrate

(here) $ python manage.py createsuperuser 
```


### Done! Nice work. Now you're ready to start creating [projects](http://joincfe.com/projects)
