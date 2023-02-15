---
title: Using Google Secrets Manager in Python Projects with Django, FastAPI, Python Decouple and GitHub Actions
slug: google-secrets-python-decouple-github-actions

publish_timestamp: Feb. 15, 2023
url: https://www.codingforentrepreneurs.com/blog/google-secrets-python-decouple-github-actions/

---


We all know that hard-coding environment variables and secrets in an application is a _bad idea_. Google Secrets is a secure and safe way to store sensitive runtime configuration, API Keys, database credentials and so on in secure way.

This article will show you how to use [Google Secrets Manager](https://cloud.google.com/secret-manager) from Google Cloud using [python-decouple](https://pypi.org/project/python-decouple/). Once we have Google Secrets Manager configured with python-decouple, we can use this same approach for Django, FastAPI, Flask, and any other Python project as we see fit. We'll top it off using GitHub Actions to be our _source of truth_ for our production secrets.

## Getting started with Python Decouple `python-decouple` 

I use `python-decouple` in two primary ways:

- Loading environment variables from dotenv files (`.env`) during development
- Loading environment variables from Google Secrets Manager during production


As always, it's recommended you use a virtual environment for your Python projects; I use [venv](https://docs.python.org/3/library/venv.html). Let's say our project's folder structure is the following:

```
path/to/project/
    .env
    .git/
    .gitignore
    Dockerfile
    project.code-workspace
    src/
        __init__.py
        env.py
        main.py
        requirements.txt
    venv/
```

With this project structure, our `BASE_DIR` variable will eventually map to `path/to/project` which will done by using `pathlib` within our Python module.

Add `python-decouple to `requirements.txt`

```bash
echo \"python-decouple\" >> src/requirements.txt
```

Install the requirements:

```bash
python -m pip install -r src/requirements.txt
```

With `python-decouple` installed, I create a `env.py` module to handle the following use cases for `python-decouple`:

- `.env` file in my project root
- Any `os.environ` variables
- Loading and using Google Secrets Manager (configuration coming later)



In `src/env.py` let's add the base configuration:

```python
import pathlib 
from functools import lru_cache

# from python-decouple
from decouple import Config, RepositoryEnv 

# maps to "path/to/project"
BASE_DIR = pathlib.Path(__file__).parent.parent 

# if Using Django
# from django.conf import settings
# BASE_DIR = settings.BASE_DIR

# ensure that `.env` is listed in `.gitignore`
ENV_PATH = BASE_DIR / ".env"


# lru_cache() is used to cache the result of the function
# so that it doesn't have to be re-evaluated on every call
# https://docs.python.org/3/library/functools.html#functools.lru_cache
@lru_cache()
def get_config():
    """Get configuration from .env file or os.environ"""
    if ENV_PATH.exists():
        return Config(RepositoryEnv(ENV_PATH))
    from decouple import config
    return config


# return our new default `config` call to replace `config` from `decouple`
config = get_config()
```

Let's say your `.env` file looks like this:

```
MODE=dev
```

Now in your Python Modules, you can use our custom way to load `python-decouple`:
    
```python
from src.env import config

MODE = config("MODE", cast=str default="staging")
```

This method can be used across Python projects with or without Google Secrets Manager. Let's modify our `env.py` to use Google Secrets Manager.


## Getting Started with Google Secrets Manager

Using Google Secrets Manager assumes you have the following:

- A google account (e.g. Gmail)
- A valid Google Cloud project (e.g. `my-project-12345`)
- Gcloud CLI installed
- The [Secret Manager API enabled](https://console.cloud.google.com/marketplace/product/google/secretmanager.googleapis.com) in your Google Cloud project

If you need help with any of the above, please considering enrolling in our [Serverless Python Containers on Cloud Run](https://www.codingforentrepreneurs.com/courses/serverless-python-cloud-run/) course or review the [Google Cloud documentation](https://cloud.google.com/docs).


Create `.env-prod` file in your project root with the following:

```bash
MODE=prod
SECRETS_ACTIVE=true
SECRET_KEY=not-so-good-secret
```

With this in mind, we'll create a new secret called `my_secret_file` with the contents of `.env-prod`


### Creating a Secret
Now create a new Google Cloud Secret from a dotenv file (.env):

```bash
gcloud secrets create my_secret_file --data-file .env-prod
```

### Update a Secret by adding a new Version
After you create your secret, you can update it by adding a new version:
```
gcloud secrets versions add my_secret_file --data-file .env-prod
```


### View Latest version of your Google Cloud Secret with gcloud:

The following command will print the contents of the latest version of the secret named `my_secret_file`:

```
gcloud secrets versions access "latest" --secret "my_secret_file"
```


### View other versions of your Google Cloud Secret with glcoud:
You can also get specific versions of the secret with:

```
gcloud secrets versions list my_secret_file
```
Then grab a version number (e.g. 1, 2, 3, etc) and run:

```
gcloud secrets versions access 2 --secret=my_secret_file
```

You can also disable old versions of a secret with:

```
gcloud secrets versions disable 123 --secret=my-secret
```


Before we continue, we must grant access from our local computer to Google Cloud with the following:

```bash
gcloud auth application-default login
```
This will allow our local machine to access various Google Cloud APIs as if it were a service account. This is the same authentication method that Cloud Run uses to access Google Cloud APIs. Directly from the docs:

"This command is useful when you are developing code that would normally use a service account but need to run the code in a local development environment where it's easier to provide user credentials" [Source](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login)


## Google Secrets Manager with Python

At this point we have 3 items completed:

- `gcloud` installed, authenticated (`gcloud auth login`), and `gcloud auth application-default login` also completed.
- A default cloud project activated `gcloud config set project <your-default-project-id>`
- A Google Cloud Project with Secrets Manager API enabled (among other APIs)
- A Google Cloud Secret created and available.

Now, let's use pure python to access our Google Cloud Secret. First, let's install the following Google Python packages:

- `google-auth`[view on pypi](https://pypi.org/project/google-auth/)
- `google-cloud-secret-manager` [view on pypi](https://pypi.org/project/google-cloud-secret-manager/)


Update `src/requirements.txt`:

```bash
echo \"google-auth\" >> src/requirements.txt
echo \"google-cloud-secret-manage\" >> src/requirements.txt
```

Install the requirements:

```bash
python -m pip install -r src/requirements.txt
```

Now we can jump into the Python shell

```
python
```

Let's verify our default project_id for this machine:

```python
# import google-auth
import google.auth
try:
    _, project_id = google.auth.default()
except google.auth.exceptions.DefaultCredentialsError:
    project_id = None
print(project_id)
```

This block does two things for us:

- Verify that we can access Google Cloud from our machine without additional credentials
- Return our activated project's id for future commands

If this block fails for you, something with your `glcoud` cli is not working correctly and you might need to re-authenticate with `gcloud auth login` and `gcloud auth application-default login`. 

If you get a `project_id` from Python, it should match the cli command:
```bash
gcloud config get-value project
```

Now that we have a `project_id`, let's use it to load our secret from Google Cloud Secrets Manager within Python:

```python
# import google-cloud-secret-manager
from google.cloud import secretmanager


client = secretmanager.SecretManagerServiceClient()
# my_secret_file is the name of the secret you
# set with `gcloud secrets create ....`
secret_label = "my_secret_file"

# project_id comes from previous step
gcloud_secret_name = f"projects/{project_id}/secrets/{secret_label}/versions/latest"

# this should print the contents of your secret
payload = client.access_secret_version(name=gcloud_secret_name).payload.data.decode("UTF-8")

# print the contents
print(payload)
```
The results should be:

```
MODE=prod
```

One of the important parts to note is this is not a dictionary but a string making running `payload.get("MODE")` nearly impossible. If our secret was just a _string_ value we needed, we'ld be done but since we want to unpack a `.env` file, we need to do a little more work by implementing a custom `python-decouple` class.


## Integrating `python-decouple` with `google-cloud-secret-manager`

Python Decouple is by far my favorite way to leverage environment variables in Python projects. I have used other solutions in the past but I really like how `python-decouple` handles _casting data types_ as well as how it handles _default values_.

For example, we can have something like this:

```
DEBUG = config("DEBUG_PROJECT", cast=bool, default=False)
```

`DEBUG_PROJECT` can be set to `1`, `true`, or `True` and `DEBUG` will be a `True` or `False` boolean value. 

With this in mind, we'll agument `python-decouple`'s `RepositoryEmpty` to support a string value of a `.env` file as that will be the payload from Google Cloud Secrets Manager. 

Here's our final `src/env.py` file that integrates `python-decouple` with Google Cloud Secrets Manager ()`google-cloud-secret-manager`):


In `src/env.py`

```python
import io
import os
import pathlib
from functools import lru_cache

# from python-decouple
from decouple import Config, RepositoryEmpty, RepositoryEnv

# import google-auth
import google.auth
# import google-cloud-secret-manager
from google.cloud import secretmanager

# maps to "path/to/project"
BASE_DIR = pathlib.Path(__file__).parent.parent 

# if Using Django
# from django.conf import settings
# BASE_DIR = settings.BASE_DIR

# ensure that `.env` is listed in `.gitignore`
ENV_PATH = BASE_DIR / ".env"


# if Using Django
# from django.conf import settings
# BASE_DIR = settings.BASE_DIR


def get_google_secret_payload(gcloud_secret_name = "my_secret_file"):
    """
    Load a secret from Google Cloud Secrets Manager
    """
    try:
        _, project_id = google.auth.default()
    except google.auth.exceptions.DefaultCredentialsError:
        project_id = None
    if project_id:
        client = secretmanager.SecretManagerServiceClient()
        # use an injected operating system environment variable 
        # for the gcloud secret name or 
        # the value of the "gcloud_secret_name" argument
        secret_label = os.environ.get("GCLOUD_SECRET_NAME", gcloud_secret_name)
        # project_id comes from previous step
        gcloud_secret_name_path = f"projects/{project_id}/secrets/{secret_label}/versions/latest"
        # this should print the contents of your secret
        payload = client.access_secret_version(name=gcloud_secret_name_path).payload.data.decode("UTF-8")
        payload = client.access_secret_version(name=name).payload.data.decode("UTF-8")
        return payload
    return None

class RepositoryString(RepositoryEmpty):
    """
    Retrieves option keys from an ENV string file
    """
    def __init__(self, source):
        """
        Take a string source with the dotenv file format:

        KEY=value
        
        Then parse it into a dictionary
        """
        source = io.StringIO(source)
        if not isinstance(source, io.StringIO):
            raise ValueError("source must be an instance of io.StringIO")
        self.data = {}
        file_ = source.read().split("\n")
        for line in file_:
            line = line.strip()
            if not line or line.startswith("#") or "=" not in line:
                continue
            k, v = line.split("=", 1)
            k = k.strip()
            v = v.strip()
            if len(v) >= 2 and (
                (v[0] == "'" and v[-1] == "'") or (v[0] == '"' and v[-1] == '"')
            ):
                v = v[1:-1]
            self.data[k] = v

    def __contains__(self, key):
        return key in os.environ or key in self.data

    def __getitem__(self, key):
        return self.data[key]


@lru_cache()
def get_config(use_gcloud=True):
    if ENV_PATH.exists():
        return Config(RepositoryEnv(ENV_PATH))
    if use_gcloud:
        payload = get_google_secret_payload()
        if payload is not None:
            return Config(RepositoryString(payload))
    from decouple import config

    return config

config = get_config()


# lru_cache() is used to cache the result of the function
# so that it doesn't have to be re-evaluated on every call
# https://docs.python.org/3/library/functools.html#functools.lru_cache
@lru_cache()
def get_config():
    """Get configuration from .env file or os.environ"""
    if ENV_PATH.exists():
        return Config(RepositoryEnv(ENV_PATH))
    from decouple import config
    return config


# return our new default `config` call to replace `config` from `decouple`
config = get_config()
```


At some point, I would imagine that `python-decouple` will have a class to replace `RepositoryString` but for now, this custom class should do the trick on properly formatted `.env` files.

## Using GitHub Actions for Managing Secrets

This step requires some knowledge of GitHub actions but it's pretty straightforward to setup. First create a secret you want to manage with Github Actions:

```
gcloud secrets create my_prod_secrets --data-file=.env-prod
```

After you create that, update Github Actions will the following repository secrets:

- `GCP_SERVICE_ACCOUNT` with the service account you want to attach this secret to (e.g. `your-service-manager@your-google-cloud-project.iam.gserviceaccount.com`)
- `GCP_SERVICE_ACCOUNT_KEY` enter the JSON Access Key for the a GOOGLE Cloud Service Account that has permission to manage secrets (can be a different account than the `GCP_SERVICE_ACCOUNT`)
- `GCP_SECRET_LABEL` the name of the secret you want to manage (e.g. `my_prod_secrets` that we created just now)
- `MODE` with the value you want to set
- `SECRET_KEY` with a strong secret key value like one you might generate for Django in [this blog post](https://www.codingforentrepreneurs.com/blog/create-a-one-off-django-secret-key/)
 

Next create the following workflow:

`.github/workflows/secrets.yaml`
```yaml
---
name: Update Google Cloud Secrets Manager
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  add-repo-secrets:
    name: Update Repo SEcrets
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - id: "auth"
        uses: "google-github-actions/auth@v0"
        with:
          credentials_json: "${{ secrets.GCP_SERVICE_ACCOUNT_KEY }}"
      - name: "Set up Cloud SDK"
        uses: "google-github-actions/setup-gcloud@v0"
      - name: Configure env
        run: |
          cat << EOF > .env
          MODE=${{ secrets.MODE }}
          SECRET_KEY=${{ secrets.SECRET_KEY }}
          EOF
      - name: Configure Secrets on GCloud
        run: |
          gcloud secrets versions add ${{ secrets.GCP_SECRET_LABEL }} --data-file .env
      - name: Attach Secrets Permission to A Custom Service Account
        run: |
          gcloud secrets add-iam-policy-binding ${{ secrets.GCP_SECRET_LABEL }} \
          --member serviceAccount:${{ secrets.GCP_SERVICE_ACCOUNT }} \
          --role roles/secretmanager.secretAccessor
```

That's it! If you want to see this done in action and for a real project, consider watching [this course](https://www.codingforentrepreneurs.com/courses/serverless-python-cloud-run/) as it will also cover how to deploy a Python Application with Docker by deploying to the Google Cloud service Cloud Run.
