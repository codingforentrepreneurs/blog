---
title: Install Jupyter Notebooks in a Virtual Environment
slug: install-jupyter-notebooks-virtualenv

publish_timestamp: June 8, 2018
url: https://www.codingforentrepreneurs.com/blog/install-jupyter-notebooks-virtualenv/

---

Let's install Jupyter for a different type of experience coding with Python. 

Jupyter is an __interactive notebook__ which means you can run blocks of code much like you were working in the `Python shell` except on a whole new level. A Jupyter Notebook (aka iPython Notebook `.ipnb`) can be easily shared with others and has become a popular staple of Data Scientists everywhere.

As a bonus, we can use Jupyter in lieu of code editors like Sublime Text or PyCharm. I love my code editor but this is just a nice feature as you'll see very soon.

Let's get it installed.

If use [Anaconda](https://www.anaconda.com/download), you already have Jupyter installed so feel free to skip this! 

Personally, I prefer virtualenv over Anaconda for many reasons amd one of which is when using Anaconda and Django I seem to constantly run into headaches. So, the remainder of this post is showing you how to install it like I do: using virutalenv.

### Requirements:
1. Do you have Python 3 installed? 
2. Do you have virtualenv even installed?

If no to either, go through a below guide for your system setup:
- [Mac users](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux/)
- [Linux users](https://www.codingforentrepreneurs.com/blog/install-django-on-linux-ubuntu/)
- [Windows Users](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows/)


### 1. Create a new virtualenv
```
$ cd path/to/your/dev/folder/
$ mkdir plutopy
$ cd plutopy
$ virtualenv -p python3 .
```
> Note you can omit `-p python3` if your system's default is python 3. You can check by running `python -V`

### 2. Activate your virtualenv
Continuing from above we'll do the following

##### Mac / Linux Terminal
```
$ source bin/activate
```

##### Windows PowerShell
```
> .\Scripts Activate
```

### 3. Install Jupyter
```
(plutopy) $ python -V
Python 3.6.5
(plutopy) $ pip install jupyter notebook
```

### 4. Start Jupyter
```
(plutopy) $ jupyter notebook
```

This should open a web broswer with your Jupyter Notebook running.