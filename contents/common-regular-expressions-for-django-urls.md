---
title: Common Regular Expressions for Django URLs
slug: common-regular-expressions-for-django-urls

publish_timestamp: March 8, 2018
url: https://www.codingforentrepreneurs.com/blog/common-regular-expressions-for-django-urls/

---

A list of common regular expressions for use in Django url's regex. Learn how [Python Regular Expressions](https://www.codingforentrepreneurs.com/blog/how-django-urls-work-with-regular-expressions/) work as they relate to Django.


### Django 2.0 & Up
```python
#urls.py
from django.urls import include, path, re_path
from appname.views import (
              AboutView,
              article_detail, 
              ContactView,
              home_view, 
              profile_detail,
              )


urlpatterns = [
    # Examples:
    re_path(r'^$', home_view, name='home_with_regex'),
    path("/", home_view, name='home_with_path'),  # same as the path above it.
    re_path(r'^contact/$', ContactView.as_view(), name='contact'),
    re_path(r'^about/$', AboutView.as_view(), name='about'),
    re_path(r'^profile/(?P<username>[\w.@+-]+)/$', profile_detail, name='profile'),
    re_path(r'^article/(?P<slug>[\w-]+)/$', article_detail, name='article'),
    re_path(r'^blog/', include("blog.urls")),
    re_path(r'^admin/', admin.site.urls),
]

# blog.urls.py
from django.urls import include, path, re_path
from django.views.generic import RedirectView, TemplateView

from .views import BlogList, BlogDetail
app_name = 'blog'
urlpatterns = [
    re_path(r'^$', BlogList.as_view(), name='list'),
    re_path(r'^(?P<slug>[\w-]+)/$', BlogDetail.as_view, name='detail'),
]
```

Notice `blog.urls.py`, see how there's the `app_name` variable? This is set inside of `urls.py` for apps and for creating [namespace urls](https://docs.djangoproject.com/en/2.0/topics/http/urls/#url-namespaces) which allows to call [reverse](https://docs.djangoproject.com/en/2.0/ref/urlresolvers/#reverse) like `reverse("<namespace>:<url-name>")`. 

A few examples of reversing a url:
 - `reverse("blog:detail", kwargs={"slug": "ommon-regular-expressions-for-django-urls"})`
 - `reverse("blog:list")`


### Django 1.9 to Django 1.11 (required in Django 1.10+)
```python
#urls.py
from django.conf.urls import url, include
from appname.views import (
              AboutView,
              article_detail, 
              ContactView,
              home_view, 
              profile_detail,
              )


urlpatterns = [
    # Examples:
    url(r'^$', home_view, name='home'),
    url(r'^contact/$', ContactView.as_view(), name='contact'),
    url(r'^about/$', AboutView.as_view(), name='about'),
    url(r'^profile/(?P<username>[\w.@+-]+)/$', profile_detail, name='profile'),
    url(r'^article/(?P<slug>[\w-]+)/$', article_detail, name='article'),
    url(r'^blog/', include("blog.urls")),
    url(r'^admin/', admin.site.urls),
]

```

### Django 1.8 and below
```python
# urls.py
from appname.views import (
              AboutView,
              ContactView,
              )
             
urlpatterns = patterns('',
    url(r'^$', 'appname.views.home_view', name='home'),
    url(r'^contact/$', ContactView.as_view(), name='contact'),
    url(r'^about/$', AboutView.as_view(), name='about'),
    url(r'^profile/(?P<username>[\w.@+-]+)/$', 'appname.views.profile_detail', name='profile'),
    url(r'^article/(?P<slug>[\w-]+)/$', 'appname.views.article_detail', name='article'),
    url(r'^blog/', include('blog.urls')),
    url(r'^admin/', include(admin.site.urls)),
)

```


### Username (user's username)

Regex:

```
(?P<username>[\w.@+-]+)
```

##### Example:

Parameters:

```python
username = 'email@email.com' 
or 
username = 'myusername' ## this paramater can be either email or username fields
```

Query:

```python
object = UserModel.objects.get(username=username)
```

Url:

```
url(?P<username>[\w.@+-]+)$', 'appname.views.show_user'),
```

View:

```python
def show_user(request, username):
    ...
    return ...
```

Live usage:

```
yourdomain.com/email@email.com/

or

yourdomain.com/myusername/

```


### Object ID (user id, profile id, group id, etc)
Regex:

```
(?P<order>\d+)
```

##### Example

Parameters:

```python
id = 1
```

Query:

```python
object = Model.objects.get(id=id)
```

Url:

```
url(r'^(?P<id>\d+)$', 'appname.views.item_id'),
```

View:

```python
def item_id(request,id):
    ...
    return ...
```

Live usage:

```
yourdomain.com/12/
```


### Username with Object Order
Regex:
```
(?P<username>[\w.@+-]+)/(?P<order>\d+)
```

##### Example

Parameters: 
```python
username = "myusername"

order = 13
```

Query:

```python
user_object = UserModel.objects.get(username=username)
queryset = UserItem.objects.filter(order=order, user=user)
```

Url:

```
url(r'^(?P<username>[\w.@+-]+)/(?P<order>\d+)/$', 'appname.views.item_home', name='home'),
```

View:

```python
def item_home(request, username, order):
    ...
    return ...
```

Live usage:
```
yourdomain.com/useritem/myusername/13/
```



### Slugs
Regex:
```
(?P<slug>[\w-]+)
```

##### Example

Parameters: 
```python
slug = "slugged-item"
```

Query:
```python
object = Articles.objects.get(slug=slug)
```

Url:
```
url(r'^(?P<slug>[\w-]+)/$', 'appname.views.article'),
```

View:
```python
def article(request,article):
    ...
    return ...
```

Live usage:
```
yourdomain.com/your-slug/
````


### Digits and Dates (through digits)

Regex:

```
4 Digits like 2015
(?P<year>\d{4})

2 Digits like 12
(?P<month>\d{2})

Any Digits like 1231 or 123438192
(?P<article_id>\d+)
``` 

##### Example

Parameters: 
```python
year = 2015
month = 01
article_id = 412
```

Query:
```python
year_queryset = Articles.objects.filter(year=year)
months_in_year_queryset = Articles.objects.filter(year=year).filter(month=month)
article_object = Articles.objects.get(id=article_id)
```

Url:
```
(r'^articles/(?P<year>\d{4})/$', views.year_archive),
(r'^articles/(\d{4})/(?P<month>\d{2})/$', views.month_archive),
(r'^articles/(?P<year>\d{4})/(?P<month>\d{2})/(?P<article_id>\d+)/$', views.article_detail),
```

View:
```python
def year_archive(request, year):
    return ..

def month_archive(request, month):
    return ..

def article_detail(request, year, month, article_id):
    ...
    return ...
```

Live usage:

```
yourdomain.com/2015/
yourdomain.com/2015/03/
yourdomain.com/2015/03/21/
```
    



If you find other regular expressions you are unsure of their meaning, feel free to [contact us](hello@teamcfe.com) or comment below..

Thank you!

Justin
Coding for Entrepreneurs