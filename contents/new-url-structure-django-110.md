---
title: New URL Structure for Django
slug: new-url-structure-django-110

publish_timestamp: Jan. 3, 2017
url: https://www.codingforentrepreneurs.com/blog/new-url-structure-django-110/

---


Django 1.10 has changed the requirements for urls.py and how you call views.

If you've ever seen the error: "view must be a callable or a list/tuple in the case of include() ..." or something of that sort then you definitely need this short post. Read below for what your `urls.py` should be depending on your Django version.


```python
from django.conf.urls import url, include
# Django 1.9 and Up (required in Django 1.10+)
# urls.py

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
    url(r'^profile/(?P<username>[\w.@+-]+)/$', profile_detail, name='about'),
    url(r'^article/(?P<slug>[\w-]+)/$', article_detail, name='about'),
    url(r'^blog/', include("blog.urls")),
    url(r'^admin/', admin.site.urls),
]


# Django 1.8 and below
# urls.py
from appname.views import (
              AboutView,
              ContactView,
              )
              
              
urlpatterns = patterns('',
    url(r'^$', 'appname.views.home_view', name='home'),
    url(r'^contact/$', ContactView.as_view(), name='contact'),
    url(r'^about/$', AboutView.as_view(), name='about'),
    url(r'^profile/(?P<username>[\w.@+-]+)/$', 'appname.views.profile_detail', name='about'),
    url(r'^article/(?P<slug>[\w-]+)/$', 'appname.views.article_detail', name='about'),
    url(r'^blog/', include('blog.urls')),
    url(r'^admin/', include(admin.site.urls)),
)

```
