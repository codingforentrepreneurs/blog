---
title: S3 Static &amp; Media Files for Django
slug: s3-static-media-files-for-django

publish_timestamp: Oct. 15, 2017
url: https://www.codingforentrepreneurs.com/blog/s3-static-media-files-for-django/

---

Using Amazon Web Services (AWS) S3 For storing static and media files for a Django Project.


## AWS Set up
### 1. **Create AWS Account**: [here](http://aws.amazon.com/)

### 2. **Create AWS User Credentials**
    
1. Navigate to [IAM Users](https://console.aws.amazon.com/iam/home?#users)

2. Select `Create New Users`

3. Enter `awsbean` as a user name (or whatever you prefer)

4. Ensure `Programmatic Access` is **selected**, hit `Next`.

5. Select `Download credentials` and keep them safe. 

6. Open the `credentials.csv` file that was just downloaded/created 

7. Note the `Access Key Id` and `Secret Access Key` as they are needed for a future step. These will be referred to as `<your_access_key_id>` and `<your_secret_access_key>`

### 3. **Create new S3 Bucket**

1. Navigate to S3 through the [Console](https://console.aws.amazon.com) or this [Link](https://console.aws.amazon.com/s3)
2. Click `Create Bucket`
3. Create a unique `Bucket Name` such as `your-project-bucket` or any other name you choose.
4. Select `Region` relative to your primary users' location.
5. Click `Create`

Make note of the `Bucket Name` you created here. It will replace `<your_bucket_name>` below.

### 4. **Add Default Access Policies to your IAM User**:
1. Navigate to the user's account such as: [https://console.aws.amazon.com/iam/home?#/users](https://console.aws.amazon.com/iam/home?#/users)
2. Select user.
3. Click on `Permissions` tab.
4. Click on `Attach Existing Policies Directly` and add any policies you'd like to give this user access to or add a custom policy.

### 5. **Add Custom Policy / Permissions to your IAM User**
1. Navigate IAM Home: [https://console.aws.amazon.com/iam/home?#/users](https://console.aws.amazon.com/iam/home?#/users)
2. Select user and click on click on `Permissions` tab.
3. Click tab for `Add Permissions`
4. Click on `Attach Existing Policies Directly` and add any policies you'd like to give this user access to or add a custom policy.
5. Click `Create Policy` then select `Create Your Own Policy`.
6. Set `Policy Name` to `S3Django` (or any name you decide)
7. Set `Policy Document` to below. Change all `<your_bucket_name>` to the name of your bucket in S3 (set above). Do not change `version` date.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation",
                "s3:ListBucketMultipartUploads",
                "s3:ListBucketVersions"
            ],
            "Resource": "arn:aws:s3:::<your_bucket_name>"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:*Object*",
                "s3:ListMultipartUploadParts",
                "s3:AbortMultipartUpload"
            ],
            "Resource": "arn:aws:s3:::<your_bucket_name>/*"
        }
    ]
}
```

8. The `Action`s that we choose to set are based on what we want this user to be able to do. The line `"s3:*Object*",` will handle a lot of our permissions for handling objects for the specified bucket within the `Recourse` Value.

9. Add CORS policy to bucket:
    1. Click on `bucket`
    2. `Permissions` > `Cors Configuration`
    3. Paste:

```
<CORSConfiguration>
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>Authorization</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
```


## **Django Setup**

### 1. Install [boto](http://boto.readthedocs.io/en/latest/), [boto3](http://boto3.readthedocs.io/en/latest/) and [Django Storages](https://django-storages.readthedocs.org/en/latest/):

```
pip install boto boto3 django-storages
```
### 2. Update `INSTALLED_APPS` in `settings.py`:
```
INSTALLED_APPS = [
    ...
    'storages',
    ...
]
```

### 3. Run migrate:
```
$ python manage.py migrate 
```
### 4. Create `aws` module in same directory as `settings.py`:
```
$ pwd
/path/to/<your-virtualenv>/src/<your-project>/
$ ls
__init__.py settings.py wsgi.py urls.py
$ mkdir aws && cd aws
$ touch __init__.py
$ touch utils.py
$ touch conf.py
```
### 5. In `utils.py` add the following:

```
from storages.backends.s3boto3 import S3Boto3Storage

StaticRootS3BotoStorage = lambda: S3Boto3Storage(location='static')
MediaRootS3BotoStorage  = lambda: S3Boto3Storage(location='media')
```

### 6. In your `conf.py` add the following:
```
import datetime
AWS_ACCESS_KEY_ID = "<your_access_key_id>"
AWS_SECRET_ACCESS_KEY = "<your_secret_access_key>"
AWS_FILE_EXPIRE = 200
AWS_PRELOAD_METADATA = True
AWS_QUERYSTRING_AUTH = True

DEFAULT_FILE_STORAGE = '<your-project>.aws.utils.MediaRootS3BotoStorage'
STATICFILES_STORAGE = '<your-project>.aws.utils.StaticRootS3BotoStorage'
AWS_STORAGE_BUCKET_NAME = '<your_bucket_name>'
S3DIRECT_REGION = 'us-west-2'
S3_URL = '//%s.s3.amazonaws.com/' % AWS_STORAGE_BUCKET_NAME
MEDIA_URL = '//%s.s3.amazonaws.com/media/' % AWS_STORAGE_BUCKET_NAME
MEDIA_ROOT = MEDIA_URL
STATIC_URL = S3_URL + 'static/'
ADMIN_MEDIA_PREFIX = STATIC_URL + 'admin/'

two_months = datetime.timedelta(days=61)
date_two_months_later = datetime.date.today() + two_months
expires = date_two_months_later.strftime("%A, %d %B %Y 20:00:00 GMT")

AWS_HEADERS = { 
    'Expires': expires,
    'Cache-Control': 'max-age=%d' % (int(two_months.total_seconds()), ),
}
```
### 7. In Django `settings.py`:
```
from <your-project>.aws.conf import *
```

### 8. Run `python manage.py collectstatic` and you should be all setup.



Subscribe on our [YouTube Channel](https://kirr.co/7l2sv4) and join us for more in-depth tutorials on [Django development](http://joincfe.com/enroll).


Cheers!