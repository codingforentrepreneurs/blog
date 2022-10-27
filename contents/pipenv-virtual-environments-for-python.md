---
title: Pipenv Virtual Environments for Python
slug: pipenv-virtual-environments-for-python

publish_timestamp: July 9, 2018
url: https://www.codingforentrepreneurs.com/blog/pipenv-virtual-environments-for-python/

---


`Pipenv` is an **amazing** replacement for `virtualenv` and it's highly recommend that your future projects use it. Below is a guide on how to activate your first `pipenv`. I intend to use it on all future Python projects and adapt some older ones (if I can) to using `pipenv`. 

Watch video [below](#watch) or [on youtube](https://youtu.be/K2fNEoZfuy8).



<hr/>
**Basic Setup Requirements**

Below you'll want to do for sure on your system. 

1. Install Python [Windows](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows/), [Mac](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux/), or [Linux](https://www.codingforentrepreneurs.com/blog/install-django-on-linux-ubuntu/)

2. Install Pipenv system wide

    **Mac/Linux**

    ```
    $ sudo pip install pipenv
    ``` 

    **Windows**

    ```
    > pip install pipenv
    ``` 

3. Create your Project's Primary Folder
    ```
    $ mkdir Dev
    $ cd Dev
    $ mkdir venv 
    $ cd venv 
    ```
4. Review the [pipenv docs](https://docs.pipenv.org/)

<hr/>

# Initialize Pipenv

If this seems easy, that's because it is. Starting a virtual enviroment has literally never been easier: 

```
$ cd ~/Dev/venv 
$ pipenv --python 3.7
```
or
```
$ cd ~/Dev/venv 
$ python3 -m pipenv --python 3.7
```
**Note:** that `python3 --V` is a command that works on my path. If you have errors, just use `pipenv --python python` or start installing with `pipenv install <a-pypi-package>`

*Did you know? We have additional options for initializing a new `pipenv` below?*

This creates a `Pipfile` for your project. Something that contains: 

```
[[source]]
url = 'https://pypi.python.org/simple'
verify_ssl = true
name = 'pypi'

[requires]
python_version = '3.7'

[packages]

[dev-packages]

```

<hr/>
# Install Packages

Get the latest:
```
$ pipenv install requests
```

With a **version**:
```
$ pipenv install django==2.0.7
```

With a **requirements.txt** file:
```
$ pipenv install -r requirements.txt
```

<hr/>
# Advanced Initialize Pipenv 
### Using System's Default Python Install (2 or 3)
```
$ cd ~/Dev/venv 
$ pipenv
```

#### Using a Python version that's easily accessible

**Python 3** 

Below will use your system's default Python 3 version.
```
$ python3 -V
Python 3.7.0
```
Any errors? If not, keep going.

```
$ pipenv --python python3 
```
or
```
$ pipenv --three
```


**Python 2** 

Using Python 2 is no longer recommended but sometimes you need to. 

```
$ python -V
Python 2.7.15
```
Any errors? If not, keep going.

```
$ pipenv --python python 
```
or
```
$ pipenv --two
```


### Using a Specific Python version installed locally.

Essentially, locate the path to the executabile Python file. Such that:
```
/path/to/python --V
```
If no errors, then you can do:
```
pipenv --python /path/to/python
```


#### Downloaded & Installed from [python.org](https://www.python.org/downloads/release/python-354/)

**Mac**
```
pipenv --python /Library/Frameworks/Python.framework/Versions/3.6/bin/python3
```
*To find what versions you have installed just go to `ls -al /Library/Frameworks/Python.framework/Versions/` and see which folders are there, mine has `3.5` and `3.6`*

**Windows**
```
pipenv --python C:\Python36
```
*This location will be denoted when you install Python or whatever version.*


#### Installed via Using Homebrew (Mac Only)

```
pipenv --python /usr/local/Cellar/python/3.6/bin/python3
```

*To find what versions you have installed just go to `ls -al ls /usr/local/Cellar/python/` and see which folders are there, mine has `3.5` and `3.6`*


### Bonus Items
**Make Virtual Environment Deterministic**

Add the environment variable to your system for `PIPENV_VENV_IN_PROJECT=1`. This will ensure your virtual environment is created in the same place you initialize it under the `.venv` directory. It also means you won't have an artibitary hash created for it as the above method is. 

**Mac** Users add 
```
export PIPENV_VENV_IN_PROJECT=1
```
to your `.bash_profile` by doing
```
nano ~/.bash_profile
```

**Windows** add the environment variable in your control panel [like in the setup process here](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows/#download-python-3-6-x).

*Thanks to pj on youtube for this recommendation*

<hr/>
# Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/K2fNEoZfuy8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
