---
title: CSV file to Django Models
slug: csv-file-to-django-models

publish_timestamp: June 9, 2017
url: https://www.codingforentrepreneurs.com/blog/csv-file-to-django-models/

---


Handling and parsing CSV (comma separated values) files using Python and Django will enable a whole new way to use data in your project. The concept is pretty simple since the CSV file is so common and very easily parsed in Python. The code below is an example implementation of using a Django Model with a csv file. 


### Models.py (`yourapp.models.py`)
```
from __future__ import unicode_literals
import csv
import io
from django.conf import settings
from django.db import models
from django.db.models.signals import post_save, pre_save
from django.utils.text import slugify

from yourapp.signals import csv_uploaded
from yourapp.validators import csv_file_validator

def upload_csv_file(instance, filename):
    qs = instance.__class__.objects.filter(user=instance.user)
    if qs.exists():
        num_ = qs.last().id + 1
    else:
        num_ = 1
    return f'csv/{num_}/{instance.user.username}/{filename}'

class CSVUpload(models.Model):
    user        = models.ForeignKey(settings.AUTH_USER_MODEL)
    file        = models.FileField(upload_to=upload_csv_file, validators=[csv_file_validator])
    completed   = models.BooleanField(default=False)

    def __str__(self):
        return self.user.username


def convert_header(csvHeader):
    header_ = csvHeader[0]
    cols = [x.replace(' ', '_').lower() for x in header_.split(",")]
    return cols


def csv_upload_post_save(sender, instance, created, *args, **kwargs):
    if not instance.completed:
        csv_file = instance.file
        decoded_file = csv_file.read().decode('utf-8')
        io_string = io.StringIO(decoded_file)
        reader = csv.reader(io_string, delimiter=';', quotechar='|')
        header_ = next(reader)
        header_cols = convert_header(header_)
        parsed_items = []
        
        '''
        if using a custom signal
        '''
        for line in reader:
            parsed_row_data = {}
            i = 0
            row_item = line[0].split(',')
            for item in row_item:
                key = header_cols[i]
                parsed_row_data[key] = item
                i+=1
            parsed_items.append(parsed_row_data)
        csv_uploaded.send(sender=instance, user=instance.user, csv_file_list=parsed_items)
        ''' 
        if using a model directly
        for line in reader:
            new_obj = YourModelKlass()
            i = 0
            row_item = line[0].split(',')
            for item in row_item:
                key = header_cols[i]
                setattr(new_obj, key) = item
                i+=1
            new_obj.save()
        '''
        instance.completed = True
        instance.save()


post_save.connect(csv_upload_post_save, sender=StaffCSVUpload)
```


### Signals.py (`yourapp.signals.py`)
```
import django.dispatch

csv_uploaded = django.dispatch.Signal(providing_args=["user", "csv_file_list"])
```

### Field Validator (`yourapp.validators.py`):

```
import csv
import os
import io

from django.core.exceptions import ValidationError

REQUIRED_HEADER  = ['First Name','Last Name','Email','Username']

def csv_file_validator(value):
    filename, ext = os.path.splitext(value.name)
    if  str(ext) != '.csv':
         raise ValidationError("Must be a csv file")
    decoded_file = value.read().decode('utf-8')
    io_string = io.StringIO(decoded_file)
    reader = csv.reader(io_string, delimiter=';', quotechar='|')
    header_ = next(reader)[0].split(',')
    if header_[-1] == '':
        header_.pop()
    required_header = REQUIRED_HEADER
    if required_header != header_:
        raise ValidationError("Invalid File. Please use valid CSV Header and/or Staff Upload Template.")
    return True
```
