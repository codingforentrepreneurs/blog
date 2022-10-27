---
title: How to Implement Django&#x27;s Built In Password Management
slug: howto-implement-django-builtin-password-management

publish_timestamp: Oct. 20, 2017
url: https://www.codingforentrepreneurs.com/blog/howto-implement-django-builtin-password-management/

---

Let's implement Django's built-in password management so our users can change their passwords as needed.

### Requirements
- You have your [email setup](https://www.codingforentrepreneurs.com/blog/configure-email-in-django/)
- User's email has been confirmed as we use their email to reset their password.


### 1. Update your Project's Main URLs
```
# yourproject.urls.py
from django.conf.urls import url, include


urlpatterns = [
        ...
        url(r'^accounts/', include('accounts.password.urls')),
        ...

]
```


### 2. Create a passwords module
```
$ cd /path/to/your/project/root # where manage.py is
$ python manage.py startapp accounts  # assuming we want to store our passwords module here.
$ mkdir accounts/passwords
$ touch accounts/passwords/__init__.py
$ touch accounts/passwords/urls.py 
```

### 3. Add a `urls.py` to your passwords module
```
# accounts.passwords.urls.py 
from django.conf.urls import url
from django.contrib.auth import views as auth_views

urlpatterns  = [
url(r'^password/change/$', 
        auth_views.PasswordChangeView.as_view(), 
        name='password_change'),
url(r'^password/change/done/$',
        auth_views.PasswordChangeDoneView.as_view(), 
        name='password_change_done'),
url(r'^password/reset/$', 
        auth_views.PasswordResetView.as_view(), 
        name='password_reset'),
url(r'^password/reset/done/$', 
        auth_views.PasswordResetDoneView.as_view(), 
        name='password_reset_done'),
url(r'^password/reset/\
        (?P<uidb64>[0-9A-Za-z_\-]+)/\
        (?P<token>[0-9A-Za-z]{1,13}-[0-9A-Za-z]{1,20})/$', 
        auth_views.PasswordResetConfirmView.as_view(), 
        name='password_reset_confirm'),

url(r'^password/reset/complete/$', 
        auth_views.PasswordResetCompleteView.as_view(), 
        name='password_reset_complete'),
]
```

### 4. Add Required Templates in `<your-templates-dir>/registration/`
** Ensure it's not within an app dir **

- Download the templates on [github](https://github.com/codingforentrepreneurs/Django-Password-Management-Templates). 



### 5. Ready to roll!