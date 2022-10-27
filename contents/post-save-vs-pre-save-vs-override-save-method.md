---
title: post_save vs pre_save vs override save method
slug: post-save-vs-pre-save-vs-override-save-method

publish_timestamp: Jan. 19, 2017
url: https://www.codingforentrepreneurs.com/blog/post-save-vs-pre-save-vs-override-save-method/

---

Which is better ... `post_save` vs`pre_save` or `override save method` when dealing with Models?

This is a great question so I wanted to distill it here. 

[Signals](https://docs.djangoproject.com/en/1.10/topics/signals/), for most cases, I believe are the best way to handle sharing data between models assuming the CRUD for each related model isn't done together (ie in views). 

So `post_save`, `pre_save`, `post_delete`, `pre_delete` and so are typically the best way to go about handling data that any given model instance relies on. This can be true about manipulating model data after a save (such as using the [slugify](https://docs.djangoproject.com/en/1.10/ref/utils/#django.utils.text.slugify) method to set a slug on a title). Signals were designed specifically for this reason. The other great thing about signals is you can connect them throughout your project and not necessarily just where the Model is defined. Just import the model and the signal you want to connect to it and bam!

```
# models.py 

from django.db.models.signals import pre_save
from django.utils.text import slugify

from yourapp.models import YourModel # assuming YourModel isn't defined on this page

def your_receiver_function(sender, instance, *args, **kwargs):
      if instance.title and not instance.slug:
               instance.slug = slugify(instance.title)

pre_save.connect(your_receiver_function, sender=YourModel)`
```

Each signal has it's own benefits as you can read about in the docs [here](https://docs.djangoproject.com/en/1.10/topics/signals/) but I wanted to share a couple things to keep in mind with the `pre_save` and `post_save` signals.

- Both are called every time `.save()` on a model is called. In other words, if you save the model instance, the signals are sent.
- running `save()` on the instance within a `post_save` can often create a never ending loop and therefore cause a `max recursion depth exceeded` error --- only if you don't use `.save()` correctly. 
- `pre_save` is great for changing just instance data because you do not have to call `save()` ever which eliminates the possibility for above. The reason you don't have to call `save()` is because a `pre_save` signal literally means right before being saved. 
- Signals can call other signals and or run delayed tasks (for Celery) which can be huge for usability. 


Overriding a `save()` method can be done but is not generally recommended as signals/receivers are designed for that specific case. Receiver functions (that handle the signal calls) can be very useful and reusable among other models as well where overriding the save method is often less reusable. 

In general, use signals over overriding Model methods (like `save()` `delete()` etc) as that's exactly what the Django developers designed them for.