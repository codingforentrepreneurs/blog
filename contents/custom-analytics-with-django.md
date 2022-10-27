---
title: Custom Analytics with Django
slug: custom-analytics-with-django

publish_timestamp: Oct. 10, 2017
url: https://www.codingforentrepreneurs.com/blog/custom-analytics-with-django/

---

Analytics can make our service better for our users. **Custom analytics** give us control over the data while providing many of the same key insights. 

I love custom analytics because it can be so simple as you'll see below. 

This guide is designed for **Django** but specifically for the [eCommerce course](https://www.codingforentrepreneurs.com/courses/ecommerce/). You can watch it [here](https://www.codingforentrepreneurs.com/courses/ecommerce/custom-analytics/), [below](#watch), or [on youtube](http://joincfe.com/youtube)

### 1 - Create the `analytics app`
```
python manage.py startapp analytics
```

### 2 - Craft the Model
```
# analytics.models.py

from django.db import models
from django.contrib.contenttypes.fields import GenericForeignKey
from django.contrib.contenttypes.models import ContentType



class ObjectViewed(models.Model):
    user            = models.ForeignKey(User, blank=True, null=True)
    content_type    = models.ForeignKey(ContentType, on_delete=models.SET_NULL, null=True)
    object_id       = models.PositiveIntegerField()
    ip_address      = models.CharField(max_length=120, blank=True, null=True)
    content_object  = GenericForeignKey('content_type', 'object_id')
    timestamp       = models.DateTimeField(auto_now_add=True)

    def __str__(self, ):
        return "%s viewed: %s" %(self.content_object, self.timestamp)

    class Meta:
        ordering = ['-timestamp']
        verbose_name = 'Object Viewed'
        verbose_name_plural = 'Objects Viewed'

```

### 3 - Create the `get_client_ip` utility method
```
# analytics.utils.py

def get_client_ip(request):
    x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
    if x_forwarded_for:
        ip = x_forwarded_for.split(',')[0]
    else:
        ip = request.META.get('REMOTE_ADDR')
    return ip
```


### 4 - Create a Django Signal
```
# analytics.signals.py 

from django.dispatch import Signal

object_viewed_signal = Signal(providing_args=['instance', 'request'])

```


### 5 - Create a Custom View Mixin (for Class Based Views)

```
# analytics.mixins.py

from .signals import object_viewed_signal

class ObjectViewMixin(object):
    def dispatch(self, request, *args,**kwargs):
        try:
            instance = self.get_object()
        except DoesNotExist:
            instance = None
        if instance is not None:
            object_viewed_signal.send(instance.__class__, instance=instance, request=request)
        return super(ObjectViewMixin, self).dispatch(request, *args, **kwargs)

```

You can now use this on any given Class Based View view that has a `get_object` method. Such as:

```
# products.views.py

from django.views.generic import DetailView


from analytics.mixins import ObjectViewMixin
from .models import Product


class ProductDetailView(ObjectViewMixin, DetailView):
    model = Product

```

### 6. Handle the Object Viewed Signal
```
# analytics.models.py 

from .signals import  object_viewed_signal

class ObjectViewed(models.Model):
    ...
    # code from above
    ...


def object_viewed_receiver(sender, instance, request, *args, **kwargs):
    c_type = ContentType.objects.get_for_model(sender)
    ip_address = None
    try:
        ip_address = get_client_ip(request)
    except:
        pass
    new_view_instance = ObjectViewed.objects.create(
                user=request.user, 
                content_type=c_type,
                object_id=instance.id,
                ip_address=ip_address
                )

object_viewed_signal.connect(object_viewed_receiver)

```


### 7. Challenge. Your Turn
I want to challenge you to make a way to monitor a User's Session. That is, save their session data after they login. It's not much different than above. What's cool about it is you can limit a user's number of sessions. Either way, we'll show you how to implement that in the [section here](https://www.codingforentrepreneurs.com/courses/ecommerce/custom-analytics/) and in the video [below](#watch).



#### Watch
*Coming Soon*