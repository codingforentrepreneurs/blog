---
title: The Top 5 Most Used Django Commands and what they mean.
slug: the-top-5-most-used-django-commands

publish_timestamp: March 22, 2017
url: https://www.codingforentrepreneurs.com/blog/the-top-5-most-used-django-commands/

---


Django command line commands as in what you type in the `Terminal` or Command Prompt` depending on your system have two different syntax operations:

- `django-admin <command>`
- `django-admin.py <command>`
- `python manage.py <command>`

I'm going to be sticking with the `python manage.py <command>` syntax for the following...
<hr/>
## 1) python manage.py runserver
This command you'll probably run the most of all commands. It means to run a emulated server on your local computer. So, after running it, you can go to [localhost:8000](http://localhost:8000) or [127.0.0.1:8000](http://127.0.0.1:8000). 

Requirements: You must run this in the `ROOT` of your django project where `manage.py` lives.

Useful subcommand: `python manage.py runserver <yourport>` so you can change the port from above. So `python manage.py runserver 8888` would allow you to access django on  [127.0.0.1:8888](http://127.0.0.1:8888). This is very useful if you want 2 or more Django servers running locally at the same time.
<hr/>

## 2) python manage.py makemigrations
When you make changes to your models, your database needs to understand how these changes might affect the database. This command automatically makes files that document these changes. You can also consider this a `version control` (aka history) for your Django models. This is useless without .... 
<hr/>
## 3) python manage.py migrate
This command replaces `python manage.py syncdb` in many ways and the main one being that `migrate` means it will update your database to the latest version of your Django models. `python manage.py migrate` will also insert external packages (such as `django-allauth`) into your database. 
<hr/>
## 4) python manage.py collectstatic
This command collects your static files to the `STATIC_ROOT` setting. Basically, it brings your `javascript`, `cascading style sheets (css)`, and `images`, to their own hosting server. 

The [Django developers](https://docs.djangoproject.com/en/1.10/howto/static-files/) recommend you use a content delivery network (CDN) for sharing your static files. Here at CFE, we feel the same way. A CDN is a hosting server for your static files. 
<hr/>
## 5) python manage.py startapp <appname>
When you need to add new functionality to your app, this command is your friend. Using it like `python manage.py startapp videos` will automatically create standard files to help you get started on your new app. It's not used as much as #1-4 but still useful. 
<hr/>
## Bonus 6) python manage.py startproject <projectname>
This command, is the best of all of them... why? Simply because it starts your Django project so you can use any of the commands above.
<hr/>
What do you think of our list? What would you add?
