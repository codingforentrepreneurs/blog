---
title: Activate, Reactivate, Deactivate your Virtualenv
slug: activate-reactivate-deactivate-your-virtualenv

publish_timestamp: Feb. 1, 2017
url: https://www.codingforentrepreneurs.com/blog/activate-reactivate-deactivate-your-virtualenv/

---


<div class='alert alert-warning'>
Virtualenv is still great but we now officially recommend using <a href='https://www.codingforentrepreneurs.com/blog/pipenv-virtual-environments-for-python/'>pipenv</a> instead. 
</div>


Here's a quick guide to activate your virtual environment (Virtualenv) -- especially if you closed your terminal window (aka command prompt).

## Activate
This is also useful for "reactivating" it

### mac/linux
```
$ cd /path/to/your/virtualenv/
$ source bin/activate

# example
$ cd ~/dev/myvenv/
$ ls 
lib bin include src static
$ source bin/activate
(myvenv) $ 
```

### windows
```
cd \path\to\your\virtualenv\
.\Scripts\bin\activate
```

## Deactivate
```
(myvenv) $  deactivate
```
