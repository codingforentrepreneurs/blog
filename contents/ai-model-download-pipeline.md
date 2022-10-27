---
title: AI Model Download Pipeline
slug: ai-model-download-pipeline

publish_timestamp: Oct. 1, 2021
url: https://www.codingforentrepreneurs.com/blog/ai-model-download-pipeline/

---

In [Build a Spam Classifier](https://www.codingforentrepreneurs.com/blog/build-a-spam-classifier-with-keras), we create a deep learning model and upload it to an object storage provider (AWS S3, Linode Object Storage, DigitalOcean Spaces, etc). Each one of these providers uses the python package `boto3`.

Below is a pipeline for downloading the model and it is reusable. In fact, it is made as a reusable pipeline to download nearly *anything* from an object storage that supports `boto3`.

This post is brought to in you in partnership with [DataStax](https://dtsx.io/3nRWZEG).

The reference code & project is on [github](https://github.com/codingforentrepreneurs/AI-as-an-API).

## Configure `pypyr` & Pipeline

Install packages `pypyr`, `python-dotenv`, `python-dotenv`:
```
pip install pypyr python-dotenv python-dotenv
```


Update `.env` with:
```
AWS_ACCESS_KEY_ID="your_object_storage_access_key"
AWS_SECRET_ACCESS_KEY="your_object_storage_secret_key"
BUCKET_NAME="see below"
ENDPOINT_URL="see below"
REGION_NAME="see below"
```

**BUCKET_NAME** must be a valid bucket on _AWS sS3_ otherwise you can use an arbitrary slugified name

**ENDPOINT_URL** is only required on DigitalOcean and Linode

**REGION_NAME** is required

## Setup Pipeline

It's true that [pypyr](https://pypyr.io/) has many capabilities to make pipelines even better. This one is about as simple as they come: run a boto3 client to download some files (`file_keys`) to some local destination (`dest_dir`).

`pipelines/sms-spam-model-download.yaml`

```yaml
context_parser: pypyr.parser.keyvaluepairs
steps:
  - name: pypyr.steps.contextsetf
    comment: set some arbitrary values in context
    in:
      contextSetf:
        dest_dir: models/spam-sms
        file_keys: [
            'exports/spam-sms/spam-model.h5', 
            'exports/spam-sms/spam-classifer-tokenizer.json', 
            'exports/spam-sms/spam-classifer-metadata.json'
          ]
  - name: pypyr.steps.py
    comment: Run python code to download the above file keys.
    in:
      py: |
          import os
          import pathlib
          import boto3
          from dotenv import load_dotenv
          load_dotenv()
          session = boto3.session.Session()
          bucket_name = os.environ.get('BUCKET_NAME')
          region_name = os.environ.get('REGION_NAME')
          endpoint_url = os.environ.get('ENDPOINT_URL') or None
          if not os.environ.get('AWS_ACCESS_KEY_ID') or not os.environ.get('AWS_SECRET_ACCESS_KEY'):
            raise Exception("AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY are required environment variables.")
          if not bucket_name or not region_name:
            raise Exception("BUCKET_NAME and REGION_NAME are required environment variables.")
          client = session.client('s3', region_name=region_name, endpoint_url=endpoint_url)
          for x in file_keys:
            dest_path = pathlib.Path(dest_dir)
            if not dest_path.exists():
              dest_path.mkdir(parents=True, exist_ok=True)
            download_path = dest_path / pathlib.Path(x).name
            client.download_file(bucket_name, x, str(download_path))
```

## Run Pipeline

```
python -m pypyr pipelines/sms-spam-model-download.yaml
```

Now you should be able to easily download the model data anytime you need.