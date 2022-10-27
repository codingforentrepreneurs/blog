---
title: SSL/TLS Settings for Django
slug: ssltls-settings-for-django

publish_timestamp: April 25, 2017
url: https://www.codingforentrepreneurs.com/blog/ssltls-settings-for-django/

---


Let's make your Django project's settings exactly what we do [here](https://www.codingforentrepreneurs.com/blog/create-a-blank-django-project/). Once you do that, you'll have a `production.py` file. That's where we'll be working. 

- Make sure your host has the ability to secure sites like these do: [Heroku](https://www.codingforentrepreneurs.com/projects/heroku/), [Elastic Beanstalk](https://www.codingforentrepreneurs.com/projects/elastic-beanstalk/), [Linode](https://www.codingforentrepreneurs.com/projects/linode/), [Webfaction](https://www.codingforentrepreneurs.com/projects/webfaction/), and [Digital Ocean](https://www.codingforentrepreneurs.com/projects/digital-ocean/). 
- You have your SSL/TLS Security Certificate installed (on one of those services)


Update `production.py` with: 
```
CORS_REPLACE_HTTPS_REFERER      = True
HOST_SCHEME                     = "https://"
SECURE_PROXY_SSL_HEADER         = ('HTTP_X_FORWARDED_PROTO', 'https')
SECURE_SSL_REDIRECT             = True
SESSION_COOKIE_SECURE           = True
CSRF_COOKIE_SECURE              = True
SECURE_HSTS_INCLUDE_SUBDOMAINS  = True
SECURE_HSTS_SECONDS             = 1000000
SECURE_FRAME_DENY               = True

```
> If you're using [nginx](https://www.codingforentrepreneurs.com/topics/nginx/) to handle your HTTPS redirects, you can probably set `SECURE_SSL_REDIRECT = False` instead. 

Update `local.py` & `base.py` to:

```
CORS_REPLACE_HTTPS_REFERER      = False
HOST_SCHEME                     = "http://"
SECURE_PROXY_SSL_HEADER         = None
SECURE_SSL_REDIRECT             = False
SESSION_COOKIE_SECURE           = False
CSRF_COOKIE_SECURE              = False
SECURE_HSTS_SECONDS             = None
SECURE_HSTS_INCLUDE_SUBDOMAINS  = False
SECURE_FRAME_DENY               = False
```
