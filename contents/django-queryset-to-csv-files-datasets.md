---
title: Django QuerySet to CSV Files &amp; Datasets
slug: django-queryset-to-csv-files-datasets

publish_timestamp: Feb. 8, 2018
url: https://www.codingforentrepreneurs.com/blog/django-queryset-to-csv-files-datasets/

---


Django QuerySets to CSVs is a useful feature when you're wanting export data in a meaningful way. This is also useful, in my opinion, because QuerySets are easier to work with than raw SQL queries. 

Here's what we're going to do:
1. Setup a basic Analytics model
2. Show basic commands to get specific data from that model
3. Parse a queryset of that model into CSV-ready rows
4. Save CSV File locally
5. Force-download CSV file in a view
6. Save CSV File to a FileField in a Model


### 1. Setup a Basic Analytics Model

But wait, how do you *implement this model* in views? Simple, go to [this post](https://www.codingforentrepreneurs.com/blog/custom-analytics-with-django/) and learn how.

```python
# analytics.models.py

from django.conf import settings
from django.db import models
from django.contrib.contenttypes.fields import GenericForeignKey
from django.contrib.contenttypes.models import ContentType

class ObjectViewed(models.Model):
    user            = models.ForeignKey(settings.AUTH_USER_MODEL, blank=True, null=True)
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

### 2. Show basic commands to get specific data from that model

```python
qs = ObjectViewed.objects.all()
data_dict = qs.values("user", "user__username", "object_id", "content_type", "timestamp")[:2]
<ObjectViewedQuerySet [{'user': 3, 'user__username': 'jmitchel3', 'object_id': 1927, 'content_type': 57}, {'user': 3, 'user__username': 'jmitchel3', 'object_id': 125, 'content_type': 58}]>

print(list(data_dict))
[{'user': 3, 'user__username': 'jmitchel3', 'object_id': 1927, 'content_type': 57}, {'user': 3, 'user__username': 'jmitchel3', 'object_id': 125, 'content_type': 58}]
```

The `.values()` call allows us to turn our queryset into a list of dictionaries with key-value pairs. This is what we want for our CSV file.

I'm going to transform this call into a function I can can anytime.


### 3. Parse a queryset of that model into CSV-ready rows
In here, we're going to make utility functions (methods) that turn any given queryset into a list of dictionaires. The key part here is just passing in the fields we want so it's a lot more simple and more easily reused.

So, take a look.

`cfehome/utils.py`
```python
def get_model_field_names(model, ignore_fields=['content_object']):
    '''
    ::param model is a Django model class
    ::param ignore_fields is a list of field names to ignore by default
    This method gets all model field names (as strings) and returns a list 
    of them ignoring the ones we know don't work (like the 'content_object' field)
    '''
    model_fields = model._meta.get_fields()
    model_field_names = list(set([f.name for f in model_fields if f.name not in ignore_fields]))
    return model_field_names


def get_lookup_fields(model, fields=None):
    '''
    ::param model is a Django model class
    ::param fields is a list of field name strings.
    This method compares the lookups we want vs the lookups
    that are available. It ignores the unavailable fields we passed.
    '''
    model_field_names = get_model_field_names(model)
    if fields is not None:
        '''
        we'll iterate through all the passed field_names
        and verify they are valid by only including the valid ones
        '''
        lookup_fields = []
        for x in fields:
            if "__" in x:
                # the __ is for ForeignKey lookups
                lookup_fields.append(x)
            elif x in model_field_names:
                lookup_fields.append(x)
    else:
        '''
        No field names were passed, use the default model fields
        '''
        lookup_fields = model_field_names
    return lookup_fields

def qs_to_dataset(qs, fields=None):
    '''
    ::param qs is any Django queryset
    ::param fields is a list of field name strings, ignoring non-model field names
    This method is the final step, simply calling the fields we formed on the queryset
    and turning it into a list of dictionaries with key/value pairs.
    '''
    
    lookup_fields = get_lookup_fields(qs.model, fields=fields)
    return list(qs.values(*lookup_fields))
```


Now, we can use the method `qs_to_dataset` to simply and quickly get us the data we need.

```python
qs = ObjectViewed.objects.all()[:2]
dataset = qs_to_dataset(qs, fields=['user__username', 'id'])
print(dataset)
{'id': 2, 'user__username': 'jmitch'},
{'id': 1, 'user__username': 'jmitch'}
```

#### QuerySet to Dataframe in Pandas

```python

import pandas as pd

def convert_to_dataframe(qs, fields=None, index=None):
    """
    ::param qs is an QuerySet from Django
    ::fields is a list of field names from the Model of the QuerySet
    ::index is the preferred index column we want our dataframe to be set to
    
    Using the methods from above, we can easily build a dataframe
    from this data.
    """
    lookup_fields = get_lookup_fields(qs.model, fields=fields)
    index_col = None
    if index in lookup_fields:
        index_col = index
    elif "id" in lookup_fields:
        index_col = 'id'
    values = qs_to_dataset(qs, fields=fields)
    df = pd.DataFrame.from_records(values, columns=lookup_fields, index=index_col)
    return df
```

Now we can test our newly created method

```python    
qs = ObjectViewed.objects.all()[:2]
df = convert_to_dataframe(qs, fields=['user__username', 'user__id', 'content_type', 'timestamp','object_id', 'id'])

print(df.head())
     object_id  content_type                        timestamp  user
id                                                                 
530       1927            57 2018-02-04 05:08:04.475104+00:00     3
529        125            58 2018-02-04 05:08:01.702909+00:00     3
```
   
#### A somewhat interesting note
Although these utilities work for this model, they *might not* work for every Django model. I'm sure there are more complex ulities out there to solve for all possible models but the point here was to think through how it might look.

### 4. Save CSV File locally

```python
import os
import csv
from django.conf import settings
from django.utils.text import slugify
from cfehome.utils import get_lookup_fields, qs_to_dataset

BASE_DIR = settings.BASE_DIR

def qs_to_local_csv(qs, fields=None, path=None, filename=None):
    if path is None:
        path = os.path.join(os.path.dirname(BASE_DIR), 'csvstorage')
        if not os.path.exists(path):
            '''
            CSV storage folder doesn't exist, make it!
            '''
            os.mkdir(path)
    if filename is None:
        model_name = slugify(qs.model.__name__)
        filename = "{}.csv".format(model_name)
    filepath = os.path.join(path, filename)
    lookups = get_lookup_fields(qs.model, fields=fields)
    dataset = qs_to_dataset(qs, fields)
    rows_done = 0
    with open(filepath, 'w') as my_file:
        writer = csv.DictWriter(my_file, fieldnames=lookups)
        writer.writeheader()
        for data_item in dataset:
            writer.writerow(data_item)
            rows_done += 1
    print("{} rows completed".format(rows_done))

```

Test it!

```python
from analytics.models import ObjectViewed
qs = ObjectViewed.objects.all()

qs_to_local_csv(qs, fields=['user__username', 'timestamp', 'object_viewed', 'id'])

```

### 5. Force-download CSV file in a view

This portion is pretty simple as it builds off of part 4. The key parts are using `StringIO` and setting the response headers.

`analytics/views.py`
```python
import csv
from io import StringIO
from django.contrib.auth.mixins import LoginRequiredMixin
from django.core.files import File
from django.http import HttpResponse, StreamingHttpResponse
from django.utils.text import slugify
from django.views.generic import View

from analytics.models import ObjectViewed
from cfehome.utils import get_lookup_fields, qs_to_dataset



class Echo:
    """An object that implements just the write method of the file-like
    interface.
    """
    def write(self, value):
        """Write the value by returning it, instead of storing in a buffer."""
        return value
        

class CSVDownloadView(LoginRequiredMixin, View):
     def get(self, request, *args, **kwargs):
        qs = ObjectViewed.objects.all()
        model_name = slugify(qs.model.__name__)
        filename = "{}.csv".format(model_name)
        fp = StringIO()
        pseudo_buffer = Echo()
        outcsv = csv.writer(pseudo_buffer)
        writer = csv.DictWriter(my_file, fieldnames=lookups)
        writer.writeheader()
        for data_item in dataset:
            writer.writerow(data_item)                     
        stream_file = File(fp)
        response = StreamingHttpResponse(stream_file,
                                         content_type="text/csv")
        response['Content-Disposition'] = 'attachment; filename="{}"'.format(filename)
        return response
           
```

### 6. Save CSV File to a FileField in a Model
This portion assumes you used the pandas dataframe from above. You don't have to it's just easier.

`datasets/models.py`
```python
from io import StringIO
from django.core.files import File
from django.db import models
from django.utils import timezone

from cfehome.utils import convert_to_dataframe

class DatasetManager(models.Manager):
    def create_new(self, qs, fields=None):
        df = convert_to_dataframe(qs, fields=fields)
        fp = StringIO()
        fp.write(df.to_csv())
        date = timezone.now().strftime("%m-%d-%y")
        model_name = slugify(qs.model.__name__)
        filename = "{}-{}.csv".format(model_name, date)
        obj = self.model(
                name = filename.replace('.csv', ''),
                app = slugify(qs.model._meta.app_label),
                model = qs.model.__name__,
                lables = fields,
                object_count = qs.count()
            )        
        obj.save()
        obj.csvfile.save(filename, File(fp)) #saves file to the file field
        return obj
    
class DatasetModel(models.Model):
    name                = models.CharField(max_length=120)
    app                 = models.CharField(max_length=120, null=True, blank=True)
    model               = models.CharField(max_length=120, null=True, blank=True)
    lables              = models.TextField(null=True, blank=True)
    object_count        = models.IntegerField(default=0)
    csvfile             = models.FileField(upload_to='datasets/', null=True, blank=True)
    timestamp           = models.DateTimeField(auto_now_add=True)
 ```
