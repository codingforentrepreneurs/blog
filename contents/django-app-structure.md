---
title: Django App Structure
slug: django-app-structure

publish_timestamp: Oct. 11, 2016
url: https://www.codingforentrepreneurs.com/blog/django-app-structure/

---


To make your individual Django app more portable (that is easier to move to other projects), you can use the following structure. In this one we called the app "posts".

```
posts/
|       __init__.py
|        admin.py
|        forms.py
|        models.py
|        tests.py
|        views.py
|---- migrations/
|                  __init__.py
|---- templates/
|--------- posts/
|                       post-detail.html
|                       post-list.html
|---- static/
|--------- posts/
|---------------- js/
|                                    post.custom.js
|---------------- css/
|                                    post.custom.css
|---------------- img/
|                                     post-default-img.jpg
```

Then when you run `python manage.py collectstatic` your static files will also go into the `STATIC_ROOT` you set in settings.py. 

for using these static files you would do:

`{% static "posts/js/post.custom.js" %}`, `{% static "posts/css/post.custom.css" %}` and so on.

and the templates:

`{% include "posts/post-detail.html" %}`

or 

`return render(request, 'posts/post-detail.html', {})`
