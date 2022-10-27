---
title: A Unique Slug Generator for Django
slug: a-unique-slug-generator-for-django

publish_timestamp: Feb. 3, 2017
url: https://www.codingforentrepreneurs.com/blog/a-unique-slug-generator-for-django/

---

Using the [Random String Generator](https://www.codingforentrepreneurs.com/blog/random-string-generator-in-python/), we create unique slugs for any given model. 

```
from django.utils.text import slugify
'''
random_string_generator is located here:
http://joincfe.com/blog/random-string-generator-in-python/
'''
from yourapp.utils import random_string_generator

def unique_slug_generator(instance, new_slug=None):
    """
    This is for a Django project and it assumes your instance 
    has a model with a slug field and a title character (char) field.
    """
    if new_slug is not None:
        slug = new_slug
    else:
        slug = slugify(instance.title)

    Klass = instance.__class__
    qs_exists = Klass.objects.filter(slug=slug).exists()
    if qs_exists:
        new_slug = "{slug}-{randstr}".format(
                    slug=slug,
                    randstr=random_string_generator(size=4)
                )
        return unique_slug_generator(instance, new_slug=new_slug)
    return slug
```

The above assumes that your model at least has the following:
```
class YourModel(models.Model):
    title   = models.CharField(max_length=120)
    slug  = models.SlugField(blank=True)
```



#### Let's break this down a little (for extra context):

*What does `Klass = instance.__class__` mean?*

What this does is set `Klass` to the actual model class that the instance comes from. That allows you to call things on it. 

Such as a `QuerySet` like `Klass.objects.all()`. 

`Klass` could be `User`, `Product`, `Profile`, or many other models you created in your project. 

So the queryset: `Klass.objects.filter(slug=slug)` is looking up all instances of that particular class that matches your lookup call `(slug=slug)` 

Finally, adding `.exists()` returns a boolean value of whether or not the queryset exists.