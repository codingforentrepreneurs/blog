---
title: A Multiple Model Django Search Engine
slug: a-multiple-model-django-search-engine

publish_timestamp: March 22, 2018
url: https://www.codingforentrepreneurs.com/blog/a-multiple-model-django-search-engine/

---

The Django [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) makes it simple to search. In this one, we'll create an internal search that can be rather useful.

Getting lost on this guide? Perhaps watching any of the [Try Django](https://www.codingforentrepreneurs.com/tags/try-django/) series would help.

Watch the [video](#watch) or on [youtube](https://youtu.be/1wi0AHxjcn8)
#### Model Examples


```python
# blog.models
class Post(models.Model):
    user            = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title           = models.CharField(max_length=120)
    description     = models.TextField(null=True, blank=True)
    slug            = models.SlugField(blank=True, unique=True)
    publish_date    = models.DateTimeField(auto_now_add=False, auto_now=False, null=True, blank=True)
    timestamp       = models.DateTimeField(auto_now_add=True)

    
# courses.models
class Lesson(models.Model):
    title           = models.CharField(max_length=120)
    description     = models.TextField(null=True, blank=True)
    slug            = models.SlugField(blank=True, unique=True)
    featured        = models.BooleanField(default=False)
    publish_date    = models.DateTimeField(auto_now_add=False, auto_now=False, null=True, blank=True)

    
# profiles.models
class Profile(models.Model):
    user            = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title           = models.CharField(max_length=120)
    description     = models.TextField(null=True, blank=True)
    slug            = models.SlugField(blank=True, unique=True)
    timestamp       = models.DateTimeField(auto_now_add=True)

```

#### Simple Search (aka Lookup)

```

>>> from django.utils import timezone
>>> from blog.models import Post
>>> qs = Post.objects.filter(publish_date__lte=timezone.now(), title__icontains="Django")

>>> from django.utils import timezone
>>> from courses.models import Lesson
>>> qs = Lesson.objects.filter(publish_date__lte=timezone.now(), featured=True)

>>> from profiles.models import Profile
>>> obj = Profile.objects.get(user__id=1) 
```


#### Advancing the Query Lookup

Here's a few more queries that are a little more robut because of `Q Lookups`. You might want to learn more about lookups in the [Products Component](https://www.codingforentrepreneurs.com/courses/ecommerce/products-component/) or specifically in [Understanding Lookups](https://www.codingforentrepreneurs.com/courses/ecommerce/products-component/understanding-lookups/).


```
>>> from django.db.models import Q
>>> from django.utils import timezone
>>> from blog.models import Post
>>> query = "Django"


>>> or_lookup = (Q(title__icontains=query) | Q(description__icontains=query))
>>> print(or_lookup)
(OR: ('title__icontains', 'Django'), ('description__icontains', 'Django'))

>>> and_lookup = (Q(title__icontains=query) & Q(description__icontains=query))
>>> print(and_lookup)
(<Q: (AND: ('title__icontains', 'Django'))>, <Q: (AND: ('description__icontains', 'Django'))>)


>>> qs_and = Post.objects.filter(publish_date__lte=timezone.now()).filter(and_lookup)
>>> print(qs_and)
>>> qs_or = Post.objects.filter(publish_date__lte=timezone.now()).filter(or_lookup)
>>> print(qs_or)
```

Now that we see how easy `Q Lookups` are, we should implement a method to our model manager to handle any given query. Below is an example on the Post model, I'll leave it to you to implement your own.

#### Update Model Manager to include a Search Method


```python
# blog.models
from django.db.models import Q

class PostManager(models.Manager):
    def search(self, query=None):
        qs = self.get_queryset()
        if query is not None:
            or_lookup = (Q(title__icontains=query) | 
                         Q(description__icontains=query)|
                         Q(slug__icontains=query)
                        )
            qs = qs.filter(or_lookup).distinct() # distinct() is often necessary with Q lookups
        return qs

    
class Post(models.Model):
    user            = models.ForeignKey(settings.AUTH_USER_MODEL)
    title           = models.CharField(max_length=120)
    description     = models.TextField(null=True, blank=True)
    slug            = models.SlugField(blank=True, unique=True)
    publish_date    = models.DateTimeField(auto_now_add=False, auto_now=False, null=True, blank=True)
    timestamp       = models.DateTimeField(auto_now_add=True)
    
    objects         = PostManager()
```

Now, let's implement this concept into a view.

#### Create the search view


```python
# search.views.py
from itertools import chain
from django.views.generic import ListView

from blog.models import Post
from courses.models import Lesson
from profiles.models import Profile

class SearchView(ListView):
    template_name = 'search/view.html'
    paginate_by = 20
    count = 0
    
    def get_context_data(self, *args, **kwargs):
        context = super().get_context_data(*args, **kwargs)
        context['count'] = self.count or 0
        context['query'] = self.request.GET.get('q')
        return context

    def get_queryset(self):
        request = self.request
        query = request.GET.get('q', None)
        
        if query is not None:
            blog_results        = Post.objects.search(query)
            lesson_results      = Lesson.objects.search(query)
            profile_results     = Profile.objects.search(query)
            
            # combine querysets 
            queryset_chain = chain(
                    blog_results,
                    lesson_results,
                    profile_results
            )        
            qs = sorted(queryset_chain, 
                        key=lambda instance: instance.pk, 
                        reverse=True)
            self.count = len(qs) # since qs is actually a list
            return qs
        return Post.objects.none() # just an empty queryset as default
```

#### Create a new template tag to get the class name


```python
# search.templatetags.class_name.py
from django import template

register = template.Library()

@register.filter()
def class_name(value):
    return value.__class__.__name__
```

#### Create the view template

`search/view.html`
```
{% extends "base.html" %}
{% load class_name %}
{% block content %}

<div class='row title-row my-5'>
    <div class='col-12 py-0'>
        <h3 class='my-0 py-0'>{{ count }} results for <b>{{ query }}</b></h3>
    </div>
</div>
        
        
{% for object in object_list %}
    {% with object|class_name as klass %}
      {% if klass == 'Post' %}
           <div class='row'>
             <div class='col-12'>
                Blog post: <a href='{{ object.get_absolute_url }}'>{{ object.title }}</a>
            </div>
          </div>

      {% elif klass == 'Lesson' %}
           <div class='row'>
             <div class='col-12'>
                Lesson Item: <a href='{{ object.get_absolute_url }}'>{{ object.title }}</a>
              </div>
            </div>
        
      {% elif klass == 'Profile' %}
           <div class='row'>
                <div class='col-12'>
                   Lesson Item: <a href='{{ object.get_absolute_url }}'>{{ object.title }}</a>
                </div>
            </div>
      {% else %}
           <div class='row'>
             <div class='col-12 col-lg-8 offset-lg-4'>
                <a href='{{ object.get_absolute_url }}'>{{ object }} | {{ object|class_name }}</a>
            </div>
           </div>
        {% endif %}
        
    {% endwith %}
    
{% empty %}
<div class='row'>
    <div class='col-12 col-md-6 mx-auto my-5 py-5'>
    <form method='GET' class='' action='.'>
    
        <div class="input-group form-group-no-border mx-auto" style="margin-bottom: 0px; font-size: 32px;">
            <span class="input-group-addon cfe-nav" style='color:#000'>
                <i class="fa fa-search" aria-hidden="true"></i>
            </span>
            <input type="text" name="q" data-toggle="popover" data-placement="bottom" data-content="Press enter to search" class="form-control cfe-nav mt-0 py-3" placeholder="Search..." value="" style="" data-original-title="" title="" autofocus="autofocus">
        </div>

    </form>

    </div>
</div>
{% endfor %}
{% endblock content %}
```

#### Even more advanced searching...

See from here, we can advance our search even more. This part `qs = sorted(queryset_chain, key=lambda instance: instance.pk, reverse=True)` shows us we can reorder our responses based on an aribratry field, or even an instance method. 

That means we can create a method to calculate "rank" for any given model and any given model instance. This rank, can be based off **analytics** like link clicks, page views, link social shares, or any other rank parameter we might like. Thus creating a better experience for your searches.

Do you want to see a "page rank" search implemented? Let us know in the comments.


# Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/1wi0AHxjcn8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>