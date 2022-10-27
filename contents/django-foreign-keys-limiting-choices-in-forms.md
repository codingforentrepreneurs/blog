---
title: Django Foreign Keys: Limiting Choices in Forms
slug: django-foreign-keys-limiting-choices-in-forms

publish_timestamp: Aug. 19, 2019
url: https://www.codingforentrepreneurs.com/blog/django-foreign-keys-limiting-choices-in-forms/

---

I'm going to assume you're not a beginner for this one. If you are, check out [this series]() and come back to this post.

The problem we're going to solve is a simple one: limit foreign key choices based on any arbitrary lookup like: 
    - `request.user` 
    - or `company_id=1` 
    - or `publish_date__gte=timezone.now()`
    - or a related file object is not used.
    
Let's say you have a blog for your entire team. Each person What I'm saying is how do we take a model like:


```python
from django.conf import settings
from django.db import models

# files.models.py
class FileItem(models.Model):
    owner = models.ForeignKey(settings.AUTH_USER_MODEL,  on_delete=models.CASCADE)
    label = models.CharField(max_length=120, blank=true, null=True)
    file = models.FileField(upload_to='files/')

# posts.models.py
class Post(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=120)
    media = models.ForeignKey(FileItem, blank=True, null=True, on_delete=models.SET_NULL)
    slug = models.SlugField()
    body = models.TextField()
    publish_date = models.DateTimeField(auto_now=False, auto_now_add=False, null=True, blank=True)
```

The above models imply that each post has 1 file item (via the `media` field). So what we want to do is find all the `FileItem` instances that are unused. We can do that with the following:


```python
unused_files = FileItem.objects.filter(post__isnull=True)
```

In forms, it's pretty simple to limit the choice. All you need to do is override the `__init__` method:


```python
# post.forms.py
from django import forms
from django.utils import timezone

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = [
            "user",
            "title",
            "media",
            "slug",
            "body",
            "publish_date"
        ]
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        unused_files = FileItem.objects.filter(post__isnull=True)
        instance = kwargs.get("instance")
        if instance:
            if instance.media:
                # if we're using this form to edit a post instance, we'll do this
                current_file = File.objects.filter(pk=instance.media.pk) 
                unused_files = ( unused_files | current_file ) # combine querysets
        self.fields['media'].queryset  = unused_files
        # pre-fill the timezone for good measure
        self.fields['publish_date'].initial = timezone.now()
```

Of course, if you want this to work within the Django admin, you'll also want to do this:


```python
# post.admin.py
from django.contrib import admin

from .forms import PostForm

class PostModelAdmin(admin.ModelAdmin):
    raw_id_fields = ['user']
    list_display = ["title", "publish_date"]
    form = PostForm
```

Do you have a challenge that needs to be solved? Please submit your ideas on [/suggest](/suggest).