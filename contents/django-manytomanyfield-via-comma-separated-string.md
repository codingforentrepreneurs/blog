---
title: Django Tutorial - ManyToManyField via a Comma Separated String in Admin &amp; Forms
slug: django-manytomanyfield-via-comma-separated-string

publish_timestamp: Aug. 21, 2019
url: https://www.codingforentrepreneurs.com/blog/django-manytomanyfield-via-comma-separated-string/

---

The `ManyToManyField` is super useful for all kinds of data association but it's not exactly user-friendly when it comes to associating this data in the Django admin or in a Django form.

Let's change that. 


```python
from django.conf import settings
from django.db import models

# topics.models.py
class TopicManager(models.Manager):
    def create_or_new(self, title):
        title = title.strip()
        qs = self.get_queryset().filter(title__iexact=title)
        if qs.exists():
            return qs.first(), False
        return Topic.objects.create(title=title), True
    
    def comma_to_qs(self, topics_str):
        final_ids = []
        for topic in topics_str.split(','):
            obj, created = self.create_or_new(topic)
            final_ids.append(obj.id)
        qs = self.get_queryset().filter(id__in=final_ids).distinct()
        return qs
        
    
class Topic(models.Model):
    title = models.CharField(max_length=120)
    slug = models.SlugField(blank=True, null=True)
    
    objects = TopicManager()

# posts.models.py
class Post(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=120)
    slug = models.SlugField()
    body = models.TextField()
    publish_date = models.DateTimeField(auto_now=False, auto_now_add=False, null=True, blank=True)
    topics = models.ManyToManyField(Topic, blank=True)
```


```python
# posts.forms.py
from django import forms
from django.utils import timezone

class PostForm(forms.ModelForm):
    topic_str = forms.CharField(label='Topics', widget=forms.Textarea, required=False)
    class Meta:
        model = Post
        fields = [
            "user",
            "title",
            "media",
            "slug",
            "body",
            "topic_str",
            "publish_date"
        ]
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        instance = kwargs.get("instance")
        if instance:
            self.fields['topic_str'].initial = ", ".join(x.title for x in instance.topics.all())
    
    def save(self, commit=True, *args, **kwargs):
        instance = super().save(commit=False, *args, **kwargs)
        topics_str = self.cleaned_data.get('topics_str')
        if commit:
            topic_qs = Topic.objects.comma_to_qs(topics_str)
            if not instance.id:
                '''
                This is a new instance.
                '''
                instance.save()
            instance.topics.clear()
            instance.topics.add(*topic_qs)
            instance.save()
        return instance
```


```python
# posts.admin.py
from django.contrib import admin
from django.db.models import Count
from django.utils.safestring import mark_safe

from topics.models import Topic

from .forms import PostForm

class PostAdmin(admin.ModelAdmin):
    raw_id_fields = ['user']
    list_display = ["title", "publish_date", "topic_list"]
    form = PostForm
    
    def topic_list(self, obj):
        topic_list = ", ".join([f"<a href='/blog/post/?topics__title={x.title}'>{x.title}</a>" for x in obj.topics.all()])
        return mark_safe(topic_list)
    
    def save_model(self, request, obj, form, change):
        topic_str = form.cleaned_data.get('topic_str')
        topic_qs = Topic.objects.comma_to_qs(topic_str)
        if not obj.id:
            obj.save()
        obj.topics.clear()
        obj.topics.add(*topic_qs)
        obj.save()

admin.site.register(Post, PostAdmin)
```

Of course this isn't the only way to accomplish this but it's definitely an effective way. It's exactly what we use for our blog posts and associating topics to them. 

Do you have a challenge that needs to be solved? Please submit your ideas on [/suggest](/suggest).