---
title: Django version 2.0 // A Few Key Features
slug: django-version-20-a-few-key-features

publish_timestamp: Dec. 8, 2017
url: https://www.codingforentrepreneurs.com/blog/django-version-20-a-few-key-features/

---

I'll be updating this post from time to time as I uncover new things about Django version 2.0. This post will not be exhaustive since the [release notes](https://docs.djangoproject.com/en/dev/releases/) are.

Something else that's important to note: this post is mainly for our content. If you're randomly finding this, you might be wondering why we left out a few things. It's simple: the things below I know will have an immediate impact for our students looking to move to Django 2.0 (but remember always follow the version in the videos!) Enjoy.

### New Starter page!
The image above, that's the new default starter page. Ahhh much better.

### `is_authenticated()` is now `is_authenticated`

Example:
```
# < 2.0 
request.user.is_authenticated()

# 2.0

request.user.is_authenticated

```


### `reverse` moved

```
# < 2.0 
from django.core.urlresolvers import reverse

# 2.0+
from django.urls import reverse
```

`django.core.urlresolvers` is now simply `django.urls`

### `on_delete` is now required on [Foreign Keys](https://docs.djangoproject.com/en/2.0/ref/models/fields/#foreignkey)
```
#models.py

user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
updated_by = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL, null=True, Blank=True, related_name='update_user')
```



### `MIDDLEWARE_CLASSES` in settings is just `MIDDLEWARE` now 


### Simplified URLs without REGEX

Both of the below accomplish the same result (and keyword argument):
```
from django.urls import re_path, path

from profiles.views import ProfileDetailView

urlpatterns = [
    ...
    re_path(r'^user/(?P<pk>\d+)/$', ProfileDetailView.as_view()),
    path('user/<int:pk>/', ProfileDetailView.as_view()),
    ...
]
```


Both of the below **almost** accomplish the same result (and keyword argument) as stated in the [release notes](https://docs.djangoproject.com/en/2.0/releases/2.0/):

```
from django.urls import re_path, path

from posts.views import YearArchive
urlpatterns = [
    ...
    re_path(r'^user/(?P<year>[0-9]{4})/$', YearArchive.as_view()),
    path('user/<int:year>/', YearArchive.as_view()),
]
```

The biggest difference between the `year` keyword arguments are the regex limits the allowed digits to 4, the other, will match any **int**


### Namespacing

The `reverse("<namespace>:<url-name>", kwargs={"<kwarg>": "<val>"})` method is great for building our urls. 

In `Django <= 1.11`, we'd simply do:

``` 
# cfehome/urls.py 
urlpatterns = [
    url(r'^posts/', include('posts.urls', namespace='posts')) 
]
``` 

Notice the `namespace` keyword argument. 

In `Django 2.0+`, we can just do:

```
# cfehome/urls.py
urlpatterns = [
    re_url(r'^posts/', include(('posts.urls', 'posts'))) 
]
```

Same thing is accomplished. We can even elminate that namespace variable all together by adding a new variable to `urls.py` in any given app. Such as:

```
# cfehome/urls.py
urlpatterns = [
    re_path(r'^posts/', include('posts.urls')) 
]
```

```
# posts/urls.py
from django.urls import re_path, path
from posts.views import YearArchive, ArchiveListView

app_name = 'posts'

urlpatterns = [
    ...
    re_path(r'^$', ArchiveListView.as_view(), name='archive-list'),
    re_path(r'^user/(?P<year>[0-9]{4})/$', YearArchive.as_view(), name='year-archive'),
    path('user/<int:year>/', YearArchive.as_view(), name='year-archive-path'),
]

```

Now, try to implement this using [reverse](https://docs.djangoproject.com/en/2.0/ref/urlresolvers/#django.urls.reverse) and the [url template tag](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#url)

```
reverse("posts:archive-list")


reverse("posts:archive-list", kwargs={"year": 2017})



{% url "posts:archive-list" %}

{% url "posts:year-archive" year=2017 %}
```


Thanks to [Senpos](https://www.codingforentrepreneurs.com/u/Senpos/) and [amir.s](https://www.codingforentrepreneurs.com/u/amir.s/) for the contributions!

=====


Those are the major impact areas for now. Let me know if you find more in the comments. Thanks!