---
title: Changing Default Python 3 in Terminal for Mac OS
slug: changing-default-python-3-in-terminal-for-mac-os

publish_timestamp: July 8, 2018
url: https://www.codingforentrepreneurs.com/blog/changing-default-python-3-in-terminal-for-mac-os/

---


Sometimes Python gets upgraded before you're ready. For example, I just used homebrew to upgrade Python. It installed version 3.7 and *not* 3.6.6 as I intended. This meant that typing `$ python3 -V` returned `3.7.0`. In the long run, this is great. But when it comes to creating a virtual environment -- slightly frustrating.

Let's fix that so we can change `$ python3` to use the version we want at any time.

** Using [homebrew](https://brew.sh) **
I have the following versions of python installed:

- Python 3.6.2
- Python 3.6.5
- Python 3.7.0

You can find this by

```
$ brew install python
$ ls /usr/local/Cellar/python
```

After you pick the version you want to represent `python3` in the following:

```
$ python3 -V
```


You can then change your Python Version by opening up your `bash_profile`
```
$ nano ~/.bash_profile
```

Add the following line replacing `3.7.0` with any other version you have installed
```
export PYTHONPATH="/usr/local/Cellar/python/3.7.0/bin/python3:$PYTHONPATH"
```
Save and close.

*Something I couldn't figure out is how to install specific versions of Python using homebrew. I have the ones installed above because I installed it awhile ago. If you find out how, please let me know in the comments otherwise we can just install directly from Python.org like...*

**[Python.org](https://python.org) install**

Well, assuming you install the `pkg` version of Python (like [Python 3.6.6](https://www.python.org/downloads/release/python-366/)) your install would be located in `/Library/Frameworks/Python.framework/Versions/3.6`


```
$ nano ~/.bash_profile
```

Add the following line replacing `3.6` with any version you have installed
```
export PYTHONPATH="/Library/Frameworks/Python.framework/Versions/3.6:$PYTHONPATH"
```
Save and close.


**Test Command**
After saving your `~/.bash_profile` you have to open a new Terminal window for the changes to take place.

```
$ python3 -V
```

You should now see the version you need. Also, depending on what you have installed, you can try

```
$ python3.7 -V
$ python3.6 -V
$ python3.5 -V
```

To find where those versions are installed you can run
```
$ which python3.5
```
Use that location to replace the `PYTHONPATH` whenever you may need.
