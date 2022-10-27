---
title: Install Python 3.8, Virtual Environments using Pipenv, Django 3+ on macOS
slug: install-django-on-mac-or-linux

publish_timestamp: March 1, 2020
url: https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux/

---

##### Need more depth and/or context about installing Python on Mac out this [tutorial series](/projects/setup-python-and-django-mac). For a complete guide on setting up your macOS for development, check out [this course](/courses/coding-with-macos).

> Looking for [windows installation](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows/)?
> Looking for [linux installation](https://www.codingforentrepreneurs.com/blog/install-django-on-linux-ubuntu/)?

## Step-by-step Text Guide

*Django* is a popular web development framework written in Python. 

*Virtual Environments*, keep project dependencies mostly isolated from one another. I recommend that each project you create, you use a different virtual envionrment. Below, we use the virtual environment manager Pipenv.

*PIP*, or Python Package Installer, allows you to install all types of python-related software (and code) include Django, virtual environments (virtualenv, pipenv, etc), Flask, Tensorflow, Python Requests, and more.

## Related Videos
[Setup Python 3.8 and Django 3+ on Mac ](/projects/setup-python-and-django-mac)

[YouTube](https://cfe.sh/youtube)



## 1. Install Python 3.8
Installing Python is much like installing any other program: go to their website, download the software, install it.
> Below this guide, we have an archived guide using [homebrew](https://brew.sh/) and virtualenv for installation.


1. Go to [python.org/downloads/mac-osx](https://www.python.org/downloads/mac-osx/)

2. Under `Stable Releases` look for: `Python 3.8.X` and replace `X` with the largest number you can find. Under that, click the link to download the `macOS 64-bit installer` 

3. After the installer downloads, open it, and install all the defaults.

4. Verify Installation:
    - Open up `Terminal` in (`Applications/Utilities/Terminal`)
    - Verify the version from above by typing:
    ```
    python3.8 -V
    ```
    Does the result match the stable release you downloaded? Great. Continue.

## 2. Install Pipenv Globally

1. Open `Terminal` in (`Applications/Utilities/Terminal`) and upgrade pip:
    ```
    python3.8 -m pip install pip --upgrade
    ```
    Another option to upgade, is `pip3 install pip --upgrade`

2. Install `Pipenv`:
    ```
    python3.8 -m pip install pipenv
    ```

    Another option to upgade, is `pip3 install pipenv --upgrade`

3. Verify `Pipenv`:
    ```
    pipenv
    ```
    If you see `zsh: command not found: pipenv` then you did the wrong installation.

## 3. Create Virtual Environment with Pipenv

1. Open `Terminal` in (`Applications/Utilities/Terminal`)

2. Make `Dev` directory:
    ```
    mkdir Dev
    ```
    You only have to do this 1 time

3. Create an empty directory (aka folder) for your project inside `~/Dev` folder:
    ```
    mkdir ~/Dev/cfehome
    ```
4. Initialize the Virtual Environment:
    ```
    cd ~/Dev/cfehome
    pipenv install --python 3.8
    ```

5. Activate the Virtual Environment:
    ```
    pipenv shell
    ```
    You can use the command `deactivate` to end your virtual environment.




## 4. Install Django & Create Django Project in your Virtual Environment

1. Navigate to your project:
    ```
    cd ~/Dev/cfehome
    pipenv install --python 3.8 # if not already done
    pipenv shell
    ```
    Naturally, `~/Dev/cfehome`, is the path to the project we created above.

2. Install Django
    To use the latest release of Django:
    ```
    pipenv install Django
    ```
    Or a specific release:
    ```
    pipenv install Django===3.0.4
    ```
    Replace `3.0.4` with `Z.Y.X` version numbers from [django](http://www.djangoproject.com/download)

3. Create Django Project
    After Django is installed, the command is simply `django-admin startproject <your-project-name>` to create the default Django project.
    ```
    cd ~/Dev/cfehome
    pipenv shell
    django-admin startproject cfehome .
    ```
    Or

    ```
    cd ~/Dev/cfehome
    pipenv run django-admin startproject cfehome .
    ```
    Notice the `.` at the end of the command above. That means it will create your project within the `~/Dev/cfehome` directory.


## 5. Now you're ready for Try Django or any other Django tutorial

- __[Try Django projects](https://www.codingforentrepreneurs.com/t/try-django)__

- __[Django projects](https://www.codingforentrepreneurs.com/t/django)__

- __[eCommerce course](https://www.codingforentrepreneurs.com/courses/ecommerce)__ : A comprehensive Django project step-by-step



Thank you!
<br/>
<br/>
<br/>
<hr/>




<div class='py-3 bg-warning px-2 rounded text-dark'><h1>Archived Version Below</h1><p class='mb-0'>The version of the installation guide below still works and is a different method than what we did above. It's just old.</p></div>


##### Mac users, check out the [coding with macOS course](/courses/coding-with-macos) for a complete guide on getting your system setup.

> Looking for [windows installation](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows/)?
> Looking for [linux installation](https://www.codingforentrepreneurs.com/blog/install-django-on-linux-ubuntu/)?

## Step-by-step Text Guide

*Django* is a popular web development framework written in Python. 

*Virtualenv*, or Virtual Environments, is a tool to keep the dependencies required by different projects in separate places, by creating virtual Python environments for them. Basically, if your project has different software versions (and it will) virtualenv keep them nice and safe.

*PIP*, or Python Package Installer, allows you to install all types of python-related software (and code) include Django, Virtual environments (virtualenv), Python Requests, and more.

## Related Videos
[JoinCFE.com](http://joincfe.com/projects#setup)

[YouTube](https://www.youtube.com/codingentrepreneurs)



## Install Python 3.6

1. Open Terminal

2. Install [homebrew](https://brew.sh/) with the command:
    ```
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    ```
3. Install Python with Brew
    ```
    brew install python
    ```
    Permissions errors?  Do this:
    ```
    sudo chown -R "$USER":admin /usr/local
    sudo chown -R "$USER":admin /Library/Caches/Homebrew
    ```

4. Get the version:
```
python3 -V
``` 
> make sure `V` is capitalized

## Install Django/Virtualenv on a Mac OS X or Linux. 

When you see code like the following:

```
ls
```
That means you should type it inside the application we mention below. Watching the free [setup videos](http://joincfe.com/projects#setup) will make this a lot easier as well.


#### Open Terminal (Applications > Utilities > Terminal)

#### Enter the following commands:
NOTE: The command `sudo` will require an admin password. The same password you use to install other programs. Typing will be hidden

1. Install Pip. (Python Package Installer):
    ```
    sudo easy_install pip
    ```
2. Install virtualenv:
    ```
    sudo pip install virtualenv
    ```
3. Navigate to where you want to store your code. 
    You have two options:
    1. Easy (on Desktop):
     ```
     cd ~/Desktop
     ```
    2. More Specific (ignoring the `cd ~/Desktop` command):
     ```
     mkdir Development
     cd Development
      ```
    Typical location for saving your code is in a folder/directory called "Development" so other you can keep it organized and in one place. 

    > Using "Dropbox" or "Google Drive" is also an optional place to store your code. If you use services like this, your code will definitely sync but it's possible virtualenv might not work properly on other computers.

4. Create a new virtualenv:
    ```
    virtualenv yourenv -p python3.6
    ``` 
    The name "yourenv" above is arbitrary. You can name it as you like. Also, `-p python3.6` will give you Python version 3.6 for your virutalenv. For this to work, Python3.6 must be installed (see above).

5. Activate virtualenv:
    ```
    source bin/activate
    ```
    The result in Terminal should be something like:
    ```
    (yourenv) Justins-iMac-2:~ jmitch$
    ``` 

6. Install Django:
    ```
    pip install django==1.11.2
    ```
    * NOTE: django==1.11.2 is for version 1.11.2. If you need a different version, replace those numbers accordingly. Such as django==1.8.7 or django==1.10.7 *

7. Happy Coding with Django.


Subscribe on our [YouTube Channel](http://joincfe.com/youtube) and join us for more in-depth tutorials on [Django development](http://joincfe.com/enroll).


Cheers!


#### Organized by CodingForEntrepreneurs