---
title: Get List of Current Users
slug: django-tutorial-get-list-of-current-users

publish_timestamp: Jan. 10, 2016
url: https://www.codingforentrepreneurs.com/blog/django-tutorial-get-list-of-current-users/

---

We get this question a good amount so we decided to create a little guide on how to solve it. Below was designed for `Django 1.9` but will likely work in future iterations of Django. 

This is not a perfect method for getting a list of current users, but it's definitely a doable one. So, let's get started.

First off, this assumes that `settings.py` has the following:
```
SESSION_EXPIRE_AT_BROWSER_CLOSE = True
```
[Docs](https://docs.djangoproject.com/en/dev/ref/settings/#session-expire-at-browser-close) - Doing the above setting ensures that sessions actually end, which will give us a more accurate count of who is currently logged in or, more specifically, who has a currently active session.

```
from django.contrib.auth.models import User
from django.contrib.sessions.models import Session
from django.utils import timezone

def get_current_users():
    active_sessions = Session.objects.filter(expire_date__gte=timezone.now())
    user_id_list = []
    for session in active_sessions:
        data = session.get_decoded()
        user_id_list.append(data.get('_auth_user_id', None))
    # Query all logged in users based on id list
    return User.objects.filter(id__in=user_id_list)
```

Now, you can use the function `get_current_users()` anywhere to get a `QuerySet` of users. This means that you can do things like:

```
queryset = get_current_users()
print(queryset)
print(queryset.exists())
print(queryset.count()) 
```

Questions? Comments? Suggestions? 

Enjoy!