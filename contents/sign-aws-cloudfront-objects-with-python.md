---
title: Sign AWS CloudFront Objects with Python
slug: sign-aws-cloudfront-objects-with-python

publish_timestamp: Jan. 9, 2021
url: https://www.codingforentrepreneurs.com/blog/sign-aws-cloudfront-objects-with-python/

---


How do you enable faster downloading of your private media assets everywhere in the world? The short answer is using a Content Delivery Network (CDN). In this guide, I'll show you how to do exactly that using AWS CloudFront.

CloudFront is an Amazon Web Services (AWS) API that turns your AWS S3 Bucket into a reliable, fast, and inexpensive way to distribute your media (images, videos, pds, javascript, css, etc) around the world as quickly as possible (aka a CDN). 

Object storage like AWS S3 typically have your assets stored in 1 location. Using a CDN distributes your assets around the world so many copies of your assets are physically really close to your users anywhere in the world. Less distance for bits to travel means faster download speeds and thus a better experience.


S3 *can deliver content anywhere* but it can be much slower for users on the other side of the world. In other words, using CloudFront isn't required unless you need the speed bump. At [CFE](https://codingforentrepreneurs.com), we definitely do since our customers are everywhere in the world.


> Are you super confused right now? Watching [Dive into AWS](https://www.codingforentrepreneurs.com/courses/aws) is probably for you. 


This guide is not:
- For beginners in Python or AWS. 
- If you don't already have objects in an S3 Bucket (watch [this](https://www.codingforentrepreneurs.com/courses/aws/aws-s3) for that)
- If you don't already have a CloudFront Distribution (watch [this](https://www.codingforentrepreneurs.com/courses/cloudfront-x-s3) for that)
- We won't be signing S3 Objects directly; we'll be signing objects through CloudFront.


This guide is to **sign existing CloudFront distributed objects via Python**.


## Get your cloudfront security credentials.

As of this writing, you cannot use standard IAM user policies to sign cloudfront urls. You must download security credentials (that include a unique id and private and public keys). 

Download these credentials here:

[https://console.aws.amazon.com/iam/home#/security_credential](https://console.aws.amazon.com/iam/home#/security_credentials)

__What do do with these keys?__

If you use `git` for deploying your projects, it is a *very bad* idea to commit these keys in your repo. In fact, I'd argue it's preferred to use environment variables.

So, we'll have to convert the `pem` key values into environment variables. Let's see how we do that.

## Turn `pem` file into 1-Line Environment Variable String
Unfortunately you cannot just copy and paste the value of a `pem` file to an environment variable. You must format it as a single string. Python can help.

Open the python shell:

```console
python
```

```python
# assumes you have a dir called "keys
import pathlib
BASE_DIR = pathlib.Path(".").parent.resolve()

private_key = BASE_DIR / 'keys' / "pk-<your-key>.pem"
public_key = BASE_DIR / 'keys' / "rsa-<your-key>.pem"

private_key_val = private_key.read_text().encode().decode()
private_key_val
```
Copy the output of `private_key_val` and __do not use python's `print()`__
```
public_key_val = public_key.read_text().encode().decode()
public_key_val
```
Copy the output of `public_key_val` and __do not use python's `print()`__

> This method allows you to copy a string that includes the `\n` and what not. We need that since we'll reformat it below. Environment variables do not work well with multi-line strings.


## Environment variables (`.env` or production)
```
AWS_CLOUDFRONT_KEY_ID='<your-cloudfront-key->'
AWS_CLOUDFRONT_DISTRIBUTION_ID='<your-cloudfront-distritubion>'

# Optional if you have a custom domain
AWS_CLOUDFRONT_CUSTOM_DOMAIN='https://cdn.yourdomain.com'

AWS_CLOUDFRONT_PUBLIC_KEY='-----BEGIN PUBLIC KEY-----\n<your-key-data->\n-----END PUBLIC KEY-----'

AWS_CLOUDFRONT_PRIVATE_KEY='-----BEGIN RSA PRIVATE KEY-----\n<your-key-data->\n-----END RSA PRIVATE KEY-----\n'
```
> Remember when adding environment variables to services like `Heroku`, Cloud Run, etc, you need to remove the leading and trailing quotes (`'` or `"`) especially on the keys. 

## Install requirements
```console
pip install rsa botocore
```

## cloudfront/signers.py
```python
# cloudfront/signers.py
import os
import datetime
import rsa

from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import padding
from botocore.signers import CloudFrontSigner


AWS_CLOUDFRONT_KEY_ID = os.environ.get('AWS_CLOUDFRONT_KEY_ID')
AWS_CLOUDFRONT_DISTRIBUTION_ID = os.environ.get('AWS_CLOUDFRONT_DISTRIBUTION_ID')
AWS_CLOUDFRONT_CUSTOM_DOMAIN = os.environ.get('AWS_CLOUDFRONT_CUSTOM_DOMAIN')
AWS_CLOUDFRONT_PUBLIC_KEY = os.environ.get('AWS_CLOUDFRONT_PUBLIC_KEY')
AWS_CLOUDFRONT_PRIVATE_KEY = os.environ.get('AWS_CLOUDFRONT_PRIVATE_KEY')

def rsa_signer(message):
    env_private_key = AWS_CLOUDFRONT_PRIVATE_KEY
    private_key_cleaned = env_private_key.replace("\\n", "\n").replace("\\t", "\t")
    private_key_encoded = private_key.encode("utf-8")
    return rsa.sign(
        message,
        rsa.PrivateKey.load_pkcs1(private_key.encode("utf-8")),
        "SHA-1")

def get_cloudfront_signer_instance():
    cloudfront_signer = CloudFrontSigner(AWS_CLOUDFRONT_KEY_ID, rsa_signer)

def cloudfront_sign(s3_key_path, expires_days=20):
    expire_date = datetime.datetime.now() + datetime.timedelta(days=expires_days)
    cloudfront_signer_instance = get_cloudfront_signer_instance()
    url_base = f'https://{domain_name}.cloudfront.net/'
    if s3_key_path.startswith('/'):
        s3_key_path = s3_key_path[1:]
    if AWS_CLOUDFRONT_CUSTOM_DOMAIN:
        url_base = AWS_CLOUDFRONT_CUSTOM_DOMAIN
    url = f"{url_base}{s3_key_path}"
    signed_url = cloudfront_signer_instance.generate_presigned_url(
        url, date_less_than=expire_date)
    return signed_url
```
