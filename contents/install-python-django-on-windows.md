---
title: Install Python 3.8, Virtual Environments using Pipenv, Django 3+ on Windows
slug: install-python-django-on-windows

publish_timestamp: March 1, 2020
url: https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows/

---


### Install Python 3.8, Virtual Environments, Django 3+ on Windows using pipenv, pip (Python Package Installer) and Windows Powershell


> Looking for [Mac OS Installation](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux/)?
Looking for [Linux installation](https://www.codingforentrepreneurs.com/blog/install-django-on-linux-ubuntu/)?

`Pipenv` allows you to install any Python library (aka Python-related software) to an isolated environment from other python packages. `pipenv` and `pip` work hand-in-hand in managing your virtual environment. You can install things like `Django`, `Requests`, `PyTorch`, `Tensorflow`, and much more.

The first part of this guide is made for Windows 10, the bottom part of the guide works on Windows 7, 8, 10 & Up.

## Step-by-step video series is [here](https://www.codingforentrepreneurs.com/projects/setup-python-and-django-windows)



### 1. Download from https://www.python.org/downloads/
1. Pick version 3.8.X (replace x with the highest number displayed)
2. Be sure to check you're downloading the right python version for your system (64bit vs 32bit) Not sure? 
	1. Do a Cortana search for `System Information`, open it
	2. look for `System Type`
	3. mine says `x64-based PC` which means mine is 64-bit and I should download the `Windows x86-64 executable installer`

### 2. Open Python Installer (likely in `Downloads`):
1. Tick/Select `Add Python 3.8 to PATH`
2. Select `Customize Installation` (this is important)
3. Tick/Select `pip` (others, leave as default)
4. Hit next
5. Tick/Select:
	- `Install for all users`
	- `Add Python to environment variables`
	- `Create shortcuts for installed applications`
	- `Precomplie standard libary`
6. Customize Install Location and use:
`C:\Python38`
7. Hit `Install`

### 3. Verify Python Installed in Powershell
1. Search/Open Windows Powershell
2. Type `python -V` and hit enter. Does the following show up?:
    ```
    Python 3.8.2
    ```
3. If typing `python -V` fails, try:
	- Restart Computer
	- Uninstall python and redo step 2 above.

4. Verify pip by entering:
    ```
    pip freeze
    ```
5. If you see `The term 'pip' is not recognized as the name...` then you do the installation correctly. Otherwise, you're good.

### 4. Update Powershell Settings:
You should only have to do this 1 time, if done correctly.

1. Search Windows Powershell (a search is important)
2. Right click, select _Run as Administrator_; cofirm security pop-up if needed
3. Enter: 
    Set-ExecutionPolicy Unrestricted

### 5. Create Dev Folder (directory):

1. Open `Windows Powershell` (not needed to run as Admin now)
2. Type:
    ```
    C:\ > cd ~/
    C:\ > mkdir Dev
    ```

### 6. Install `Pipenv`:
Going forward, whenever you see `>` or `$` before code, that means you should be working in the `Windows Powershell` (or `Command Prompt` if you don't have `Windows Powershell`)

1. To install a `Pipenv` as our virtual environment manager: 
	```
	> python -m pip install pipenv
	```
    > Using `python -m` allows us to definitely use the python we just installed. You can also try [this guide](https://pipenv.readthedocs.io/en/latest/)

2. Verify:
    ```
    > pipenv
    Usage: pipenv [OPTIONS] COMMAND [ARGS]...

    Options:
    --where             Output project home information.
    --venv              Output virtualenv information.
    ...
    ``` 

### 7. Create a Pipenv-based Virtual Environment:
1. Navigate to Dev:
	```
	> cd ~/Dev
	``` 
2. Make your project's parent directory:
	```
	> mkdir cfehome
	> cd cfehome
	```
3. Create `pipenv` virtual environment:
	```
	> pipenv install
	```
    > Using `pipenv install` will use your default `python`. 

4. Activate your environment:
	```
	> cd \path\to\your\project\
    > pipenv shell
	```

5. Nice work! Just verify:
	```
	(cfehome) > pip freeze
	```
	- the `(cfehome)` is the name of the `virtualenv parent directory` from above. When you see this, that means the virtualenv has been activated.
	- `pip freeze` should return nothing at this point

6. Verify Python Executable Location

    Remember when we set `C:\Python38` above? The execuble location should be `C:\Python38\python.exe`, let's check our pipenv's python executable:

    ```
    > python
    ```
    ```
    >>> import sys
    >>> print(sys.executable)
    ```

### 7. Reactivate and Deactivate Pipenv Virtual Environment

    > cd ~\Dev\cfehome # or your projects's path
    > pipenv shell
    (cfehome) > deactivate
    > pipenv shell
    (cfehome) >

### 8. Now install *any* Python Package:
    > cd ~\Dev\cfehome
    > pipenv shell
    (cfehome) > pipenv install Django==3.0.2

> *NOTE: Django==3.0.2 is for version 3.0.2. If you need a different version, replace those numbers accordingly. Such as django==2.2.8 or django==1.11.2. This is true for any python package.

### 9. Nice work! What did you think of our guide? Would like share it on <a href="https://www.facebook.com/sharer/sharer.php?u=https://kirr.co/6r8wr9" target="_blank">facebook</a>? 


==================================

<div class='py-3 bg-warning px-2 rounded text-dark'><h1>Archived Version Below</h1><p class='mb-0'>The version of the installation guide below still works and is a different method than what we did above. It's just old.</p></div>


### Install Python, Virtual Environments, Django on Windows using PIP (Python Package Installer) and Windows Powershell


> Looking for [Mac OS Installation](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux/)?
Looking for [Linux installation](https://www.codingforentrepreneurs.com/blog/install-django-on-linux-ubuntu/)?

The python package installer (PIP) allows you to install all types of python-related software (and code) include Django, Virtual environments (virtualenv), Python Requests, Flask, CFE CLI, and more.

The first part of this guide is made for Windows 10, the bottom part of the guide works on Windows XP, 7, 8, 10 & Up.


#### Watch installation series [on youtube](https://youtu.be/3TqO5FfhV28)  or [on CFE](https://kirr.co/dq8x6e) 



### 1. Download from https://www.python.org/downloads/
1. Pick version 3.6.X (replace x with the highest number displayed)
2. Be sure to check you're downloading the right python version for your system (64bit vs 32bit) Not sure? 
	1. Do a Cortana search for `System Information`, open it
	2. look for `System Type`
	3. mine says `x64-based PC` which means mine is 64-bit and I should download the `Windows x86-64 executable installer`

### 2. Open Python Installer (likely in `Downloads`):
1. Tick/Select `Add Python 3.6 to PATH`
2. Select `Customize Installation` (this is important)
3. Tick/Select `pip` (others, leave as default)
4. Hit next
5. Tick/Select:
	- `Install for all users`
	- `Add Python to environment variables`
	- `Create shortcuts for installed applications`
	- `Precomplie standard libary
6. Customize Install Location and use:
`C:\Python36
7. Hit `Install`

### 3. Verify Python Installed in Powershell
1. Search/Open Windows Powershell
2. Type `python -V` and hit enter. Does the following show up?:
```
Python 3.6.2
```
3. If typing `python -V` fails, try:
	- Restart Computer
	- Uninstall python and redo step 2 above.

4. Verify pip by entering:
```
pip freeze
```
5. If you see `The term 'pip' is not recognized as the name...` then you do the installation correctly. Otherwise, you're good.

### 4. Update Powershell Settings:
You should only have to do this 1 time, if done correctly.
1. Search Windows Powershell (a search is important)
2. Right click, select "Run as Administrator"; cofirm security pop-up if needed
3. Enter:
```
Set-ExecutionPolicy Unrestricted
```

### 5. Create Dev Folder (directory):

1. Open `Windows Powershell` (not needed to run as Admin now)
2. Type:
```
C:\ > cd ~/
C:\ > mkdir Dev
```

### 6. Install Virtualenv:
Going forward, whenever you see `>` or `$` before code, that means you should be working in the `Windows Powershell` (or `Command Prompt` if you don't have `Windows Powershell`)

1. To install a virtual environment: 
	```
	> pip install virtualenv
	```
2. Verify:
```
> pip freeze
virtualenv==15.1.0
``` 

### 7. Create a Virtualenv:
1. Navigate to Dev:
	```
	> cd ~/Dev
	``` 
2. Make virtualenv parent directory:
	```
	> mkdir cfehome
	> cd cfehome
	```
3. Create virtualenv
	```
	> virtualenv .
	```
	Note, if you have two versions of python installed you may have to do this:
	```
	> virtualenv -p python3 .
	```

4. Virtualenv shortcut to above steps:
	```
	> cd ~/Dev
	> virtualenv yourenvname
	> cd yourenvname
	```

5. Activate your environment:
	```
	> cd \path\to\your\virtualen\env\ 
	> cd ~\Dev\cfehome
	> .\Scripts\activate
	```
6. Nice work! Just verify:
	```
	(cfehome) > pip freeze
	```
	- the `(cfehome)` is the name of the `virtualenv parent directory` from above. When you see this, that means the virtualenv has been activated.
	- `pip freeze` should return nothing at this point

### 8. Reactivate and Deactivate Virtualenv
```
> cd ~\Dev\cfehome # or your virtualenv's path
> .\Scripts\activate
(cfehome) > deactivate
> .\Scripts\activate
(cfehome) >
``` 

### 9. Now install *any* Python Package:
```
> cd ~\Dev\cfehome
> .\Scripts\activate
(cfehome) > pip install django==1.11.5
```
> *NOTE: django==1.11.5 is for version 1.11.5. If you need a different version, replace those numbers accordingly. Such as django==1.8.7 or django==1.10.7 or django==1.11.2. This is true for any python package.

### 10. Nice work! What did you think of our guide? Would like share it on <a href="https://www.facebook.com/sharer/sharer.php?u=https://kirr.co/6r8wr9" target="_blank">facebook</a>? 


==================================


### Watch on YouTube
<iframe width="560" height="315" src="https://www.youtube.com/embed/3TqO5FfhV28" frameborder="0" allowfullscreen></iframe>


Still having trouble? Try our old installation guide:



#### Download Python 3.6.X
1. Download The [Latest Python 3 Release](https://www.python.org/downloads/release/python-360/). It's currently Python 3.6.0 and the download link is listed under *files* and choose the appropriate one for your system. 64-bit likely works for you. *Note: Python 3.6 is preferred for Coding for Entrepreneurs tutorials. (We have upgraded from version 2.7)*
2. Run Standard Installation Process
3. Add Environment Variables to setup up Command Prompt to work with Python. This is where we'll run the bulk of our commands. (Works on Windows XP, 7, 8, 10 & Up)
 	1. Open `Control Panel`
 	2. Select `System and Security`
 	3. Select `System` 
 	4. Select `Advanced System Settings`
 	5. Select `Advanced` Tab
 	6. Select `Environment Variables`
 	7. Under "User variables for <username>" select the variable `PATH` then hit `edit`
 	8. If `PATH` is not a current user variable, select `new` and set `Variable Name` as `PATH`
 	9. Add the following to the end of whatever is written in the `Variable Value` (if anything):
	```
	C:\Python36;C:\Python36\python.exe;C:\Python\36\Lib\site-packages;C:\Python36\Lib\site-packages\django\bin;C:\Python36\Scripts;
	```
	If something was already in `Variable Value` the end result will look something like:
	```
	C:\Windows\System32;C:\Python36;C:\Python36\python.exe;C:\Python\36\Lib\site-packages;C:\Python36\Lib\site-packages\django\bin\;C:\Python36\Scripts;
	```

4. Open a new `Command Prompt` window and type `python` if you see something like:
	```
	Python 3.6.0 (default, Feb 13 2017, 13:18:45)
	>>> 
	``` 

	You have `Python` successfully installed. You can now exit python:

	```
	>>> exit()
	```

### Install Pip
Pip is the Python Package Installer which allows you to install things like Django, Virtualenv, Requests, and more to your local computer. A must-have for developing with Python.

1. Save the "get-pip.py" file to your Desktop. You can find the file [here](https://bootstrap.pypa.io/get-pip.py) or at the [docs](http://pip.readthedocs.org/).
2. Open Command Prompt and do:
	```
	C:\ > cd Desktop
	C:\ > python get-pip.py
	```

3. Now "pip" should work system wide. The following Commands will now work:
	```
	C:\ > pip install virtualenv
	C:\ > pip freeze
	C:\ > pip install Django==1.11.2
	```
*NOTE: django==1.11.2 is for version 1.11.2. If you need a different version, replace those numbers accordingly. Such as django==1.8.7 or django==1.10.7*



#### Still having issues? Read on.

- If you can't get virtualenv working, just work outside of it. Just remember that different software versions do not always work well. Revisit installing virtualenv once you have a better grasp of Django.

- You may need to open a new command prompt window to work correctly.

- Having trouble installing PIP? Let's try using setuptools. 

This assumes you installed Python version 3.6 successfully. To test, open command prompt and type:

```
C: \ > python	 
>>> exit() 
```

If you can do the above, you have python installed on your computer. 


1. Save the ["ez_setup.py" file](https://bootstrap.pypa.io/ez_setup.py) to your desktop. 

2. Open Command Prompt and do:
```
> cd Desktop
> python ez_setup.py
> easy_install pip
```

3. Now "pip" should work system wide. The following Commands will now work:
```
> pip install virtualenv
> pip freeze
> pip install Django==1.11.2
```


You're now ready to start using Python Packages like Django!



Cheers!


#### Organized by CodingForEntrepreneurs
