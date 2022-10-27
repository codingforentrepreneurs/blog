---
title: How Django URLs work with Regular Expressions
slug: how-django-urls-work-with-regular-expressions

publish_timestamp: March 8, 2018
url: https://www.codingforentrepreneurs.com/blog/how-django-urls-work-with-regular-expressions/

---


Let's better understand Django's url routing. We're going to be looking into specifically Regular Expression URL Routing with Django's `re_path` ([docs](https://docs.djangoproject.com/en/2.0/ref/urls/#re-path)) (just `url` ([docs](https://docs.djangoproject.com/en/2.0/ref/urls/#url)) in Django < 2.0) module.

Watch it [here](#watch) or on [youtube](https://youtu.be/8rExil_EWtk)

#### Naturally, to use this guide, we're going to be working with Django 2.0 and up
```
pip install django==2.0.3

```
#### Requirements:
- Some Django Experience (even Django <2.0 like [Try Django v1.11](https://www.youtube.com/watch?v=yDv5FIAeyoY&t=6s))
- A Django project setup and ready to work with

## Regular Expressions for Pattern Matching in URLs

Let's look at Regular Exprssions (regex) in Python as they relate to URLs. 

It's pretty simple (just run the code below).


```python
import re


blog_path_1 = '/blog/1/'
blog_path_2 = '/blog/2/'
blog_path_3 = '/blog/34/'
blog_path_4 = '/blog/winter-is-coming/'

matching_digits_regex = r'^/blog/(?P<id>\d+)/$'
        
def get_id(path):
    p = re.compile(matching_digits_regex)
    print('{}\'s id is {}'.format(path, p.search(path).group('id')))

get_id(blog_path_1)
get_id(blog_path_2)
get_id(blog_path_3)

try:
    get_id(blog_path_4)
except:
    print("{} does not have a matching id".format(blog_path_4))

print("\n")

matching_slug_regex = r'^/blog/(?P<slug>[\w-]+)/$'
def get_slug(path):
    p = re.compile(matching_slug_regex)
    print('{}\'s slug is {}'.format(path, p.search(path).group('slug')))

get_slug(blog_path_1)
get_slug(blog_path_2)
get_slug(blog_path_3)
get_slug(blog_path_4)
```

    /blog/1/'s id is 1
    /blog/2/'s id is 2
    /blog/34/'s id is 34
    /blog/winter-is-coming/ does not have a matching id
    
    
    /blog/1/'s slug is 1
    /blog/2/'s slug is 2
    /blog/34/'s slug is 34
    /blog/winter-is-coming/'s slug is winter-is-coming


What we see with these strings, we can extract the exact parameters we need. Let's look at a couple more examples:


```python
'''
Two slugs in one path
'''
multiple_slugs_regex = r'^/blog/(?P<slug>[\w-]+)/(?P<slug2>[\w-]+)/$'
path = '/blog/winter-is-coming/the-knight-king-is-here/'
print(re.compile(multiple_slugs_regex).search(path).group('slug', 'slug2'))


'''
Mix IDs and Slugs
'''
digits_and_slug_regex = r'^/blog/(?P<id>\d+)/(?P<slug>[\w-]+)/$'
path = '/blog/1233/the-knight-king-is-here/'
print(re.compile(digits_and_slug_regex).search(path).group('id', 'slug'))


'''
Mix Argument Names
'''
username_and_slug_regex = r'^/blog/(?P<username>[\w.@+-]+)/(?P<slug>[\w-]+)/$'
path = '/blog/jon.snow/the-knight-king-is-here/'
print(re.compile(username_and_slug_regex).search(path).group('username', 'slug'))
```

    ('winter-is-coming', 'the-knight-king-is-here')
    ('1233', 'the-knight-king-is-here')
    ('jon.snow', 'the-knight-king-is-here')


### See Common Regular Expressions for Django urls: [here](https://www.codingforentrepreneurs.com/blog/common-regular-expressions-for-django-urls/)

| Regular Expression (regex)        | Keyword argument(s)      |
| ------------- |:-------------:| -----:|
| `r'^/blog/(?P<id>\d+)/(?P<slug>[\w-]+)/$'` | id, slug |
| `r'^/blog/(?P<slug>[\w-]+)/(?P<slug2>[\w-]+)/$'`      | slug1, slug2     |
| `r'^/profile/(?P<username>[\w.@+-]+)/$'`      | username    |
| `r'^/profile/(?P<username>[\w.@+-]+)/posts/(?P<slug>[\w-]+)/$'`      | username, slug    |
| `r'^/profile/$'`      | `None`    |

### What is URL Routing?
Before we apply the Regular Expressions to Django, we need to understand what URL routing is, it's actually pretty simple. It's just a matter of how Django understands what the user is requesting. 



| Full Path        | Path / URL Route        | View (handler)  |
| ------------- |:-------------:| -----:|
| http://example.com/ | / | HomePageView |
| http://example.com/about/      | /about/      |   AboutUsView |
| http://example.com/about/team/      | /about/team/      |   AboutOurTeamView |
| http://example.com/contact/ | /contact/      |    ContactUsView |




Real life example*: Any given Post Office (like `USPS`) routes mail to where it needs to go. It does *not* handle the mail. That's what **URL Routing** is -- it routes a request to where it needs to be handled (such as a Django View or an Angular Component).

**++XP: This is just about routing, not about handling.**

`/about/` goes to urls.py which sends it to `AboutUsView`

`/about/team/` goes to urls.py which sends it to `AboutOurTeamView`

`/contact/` goes to urls.py which sends it to `ContactUsView`

and so on.


#### Dynamic URL Routing
The above examples are **static routes** in that they pretty much never change. The `/about/` path will always be the `/about/` path (unless we manually change it).

What about when you release a new blog post? Say `/blog/jon-is-the-knight-king/` or `/blog/westworld-is-incredible/` how is this routed? Introducing **dynamic routes**.


| Full Path        | Path / URL Route          | View (handler)  |
| ------------- |:-------------:| -----:|
| http://example.com/blog/jon-is-the-knight-king/ | /blog/jon-is-the-knight-king/ | BlogPostDetailView |
| http://example.com/blog/westworld-is-incredible/ | /blog/westworld-is-incredible/ | BlogPostDetailView |


We see in this example, the **handler (view)** is the exact same for both paths. The only way for the View (handler) to know which post to display, it must accept some additional values.

AND since we know about Regular Expressions (aka *regex*), we can use this knowledge to help Django better understand what the route is requesting exactly at least according to the url's full path.

### Example urls.py
Django 2.0 & up uses `re_path` ([docs](https://docs.djangoproject.com/en/2.0/ref/urls/#re-path))
Django 1.11 & down uses `url` ([docs](https://docs.djangoproject.com/en/2.0/ref/urls/#url))

We're ignoring the simple `path` ([docs](https://docs.djangoproject.com/en/2.0/ref/urls/#path))

Below is a URLs.py file that does four things:
1. Sets a regex path to be routed to particular view using `re_path`
2. In the `BlogPostDetailView.as_view()` path, we see there is a non-optional keyword argument of `id`. This is a **dynamic route**.
3. If a user's requested path seems to match a **dynamic route** but is missing the `non-optional keyword argument(s)` then the path is skipped to look for another path if there is one. 
4. If no url patterns match the user's requested path, a 404 page is rendered automatically by Django.


##### yourproject.urls.py
```
from django.urls import include, path, re_path

from someapp.views import (
    BlogPostListView,
    BlogPostDetailView
)
urlpatterns = [
    re_path(r'^blog/$', BlogPostListView.as_view(), name='home'),
    re_path(r'^blog/(?P<id>\d+)/$', BlogPostDetailView.as_view(), name='detail'),
]
```

| Full Path        | Keyword Argument (if any)        | View (handler)  |
| ------------- |:-------------:| -----:|
| http://example.com/blog/ | `None` | BlogPostListView |
| http://example.com/blog/12/  | id=12      |   BlogPostDetailView |
| http://example.com/blog/a-new-post/      | `a-new-post` does not match `(?P<id>\d+)`, so `None`      |   Raise a 404 Page |


#### Summary

Any given user requests our website. There are a lot of URLs they can go to. Our `urls.py` helps route any given url path to the appropriate view. We use `regular expressions` to match these url paths to their corresponding request handler (aka View). If a path, either dynamic or static, is matched the View will then handle the request and return a response to the User. If a path is not matched, Django will automatically send a 404 Page Not Found response.

Keep in mind that URL routing is not concerned at all how the request is handled. That's not it's job. All it does is make sure that the requested url path (`http://example.com/blog/12/` or `http://example.com/about/`) is sent to the appropriate view.

# Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/8rExil_EWtk" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
