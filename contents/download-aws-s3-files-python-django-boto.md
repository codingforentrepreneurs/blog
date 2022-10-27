---
title: Download AWS S3 Files using Python &amp; Boto
slug: download-aws-s3-files-python-django-boto

publish_timestamp: Nov. 7, 2017
url: https://www.codingforentrepreneurs.com/blog/download-aws-s3-files-python-django-boto/

---


The purpose of this guide is to have a simple way to download files from any S3 Bucket. We're going to be downloading using [Django](http://djangoproject.com) but the majority of these methods can be used for any python project.

This guide uses Amazon Web Services (AWS) [Boto Library](https://github.com/boto/boto). Boto can be used side by side with Boto 3 according to their [docs](https://github.com/boto/boto#boto-3).

### Lean how this is implemented [here](https://www.codingforentrepreneurs.com/courses/ecommerce/selling-digital-items/)

### 1. Setup [AWS for Django](https://www.codingforentrepreneurs.com/blog/s3-static-media-files-for-django/). 


### 2. (Optional) Setup [Django/S3 for Large File Uploads](https://www.codingforentrepreneurs.com/blog/large-file-uploads-with-amazon-s3-django/)


### 3. Install Boto (likely done above)

```
pip install boto
```

### 4. AWS Configuration
Add the following settings to your python project. In this guide we're using Django but these can be implemented in any python project.

#### Django Settings Conf
```

AWS_STORAGE_BUCKET_NAME = '<your-aws-bucket-name>'
AWS_ACCESS_KEY_ID = '<your-aws-user-access-key-id>'
AWS_SECRET_ACCESS_KEY =  '<your-aws-user-secret-key>'

S3DIRECT_REGION =  'us-west-2' 

PROTECTED_DIR_NAME = '<your-in-bucket-dir-name>'
PROTECTED_MEDIA_URL = '//%s.s3.amazonaws.com/%s/' %( AWS_STORAGE_BUCKET_NAME, PROTECTED_DIR_NAME)

AWS_DOWNLOAD_EXPIRE = 5000 #(0ptional, in milliseconds)

```


### 5. Create the AWS Download Class
```
# aws.download.utils.py

import boto
import re
import os

from django.conf import settings
from boto.s3.connection import OrdinaryCallingFormat


class AWSDownload(object):
    access_key = None
    secret_key = None
    bucket = None
    region = None
    expires = getattr(settings, 'AWS_DOWNLOAD_EXPIRE', 5000)

    def __init__(self,  access_key, secret_key, bucket, region, *args, **kwargs):
        self.bucket = bucket
        self.access_key = access_key
        self.secret_key = secret_key
        self.region = region
        super(AWSDownload, self).__init__(*args, **kwargs)

    def s3connect(self):
        conn = boto.s3.connect_to_region(
                self.region,
                aws_access_key_id=self.access_key, 
                aws_secret_access_key=self.secret_key,
                is_secure=True,
                calling_format=OrdinaryCallingFormat()
            )
        return conn

    def get_bucket(self):
        conn = self.s3connect()
        bucket_name = self.bucket
        bucket = conn.get_bucket(bucket_name)
        return bucket

    def get_key(self, path):
        bucket = self.get_bucket()
        key = bucket.get_key(path)
        return key

    def get_filename(self, path, new_filename=None):
        current_filename =  os.path.basename(path)
        if new_filename is not None:
            filename, file_extension = os.path.splitext(current_filename)
            escaped_new_filename_base = re.sub(
                                            '[^A-Za-z0-9\#]+', 
                                            '-', 
                                            new_filename)
            escaped_filename = escaped_new_filename_base + file_extension
            return escaped_filename
        return current_filename

    def generate_url(self, path, download=True, new_filename=None):
        file_url = None
        aws_obj_key = self.get_key(path)
        if aws_obj_key:
            headers = None
            if download:
                filename = self.get_filename(path, new_filename=new_filename)
                headers = {
                    'response-content-type': 'application/force-download',
                    'response-content-disposition':'attachment;filename="%s"'%filename
                }
            file_url = aws_obj_key .generate_url(
                                response_headers=headers,
                                 expires_in=self.expires, 
                                method='GET') 
        return file_url
```


### 6. Usage
```
bucket = AWS_STORAGE_BUCKET_NAME
region = S3DIRECT_REGION
access_key = AWS_ACCESS_KEY_ID
secret_key = AWS_SECRET_ACCESS_KEY
path = 'path/to/object/to/download/in/your/bucket/somefile.jpg'

aws_dl_object =  AWSDownload(access_key, secret_key, bucket, region)
file_url = aws_dl_object.generate_url(path, new_filename='New awesome file')
```
