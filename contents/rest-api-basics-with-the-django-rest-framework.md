---
title: REST API Basics with the Django REST Framework
slug: rest-api-basics-with-the-django-rest-framework

publish_timestamp: Dec. 19, 2017
url: https://www.codingforentrepreneurs.com/blog/rest-api-basics-with-the-django-rest-framework/

---


This post is the same as the project on [github](https://github.com/codingforentrepreneurs/REST-API-Basics). 

This is a basic guide on how to build a REST API with Django & Python. For much deeper depth and understanding, check out our new course on [REST API](https://kirr.co/rfqyre).

### Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/tG6O8YF91HE" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>
### Software
- Django 1.11.8 (https://kirr.co/hjogvt)
- Python 3.6.3 (https://kirr.co/ftq97z)
- Django Rest Framework 3.7.3 (https://kirr.co/svez0s)
- Django Rest Framework JWT 1.11.0 (https://kirr.co/vpibmo)


### Related Source Code: https://kirr.co/9gqpkg


### Initial Setup
```
virtualenv -p python3 restapi-basics

# activate it 
# mac: `source bin/activate`
# windows: `.\Scripts\activate`

pip install django==1.11.8 djangorestframework==3.7.3 djangorestframework-jwt==1.11.0

mkdir src && cd src

django-admin startproject cfehome .
django-admin startapp postings
```



### Update your settings to reflect the following:

```
INSTALLED_APPS = [
    ...
    'rest_framework',
    'postings'
]



REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ),
}
```
