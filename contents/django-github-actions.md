---
title: Django &amp; Github Actions
slug: django-github-actions

publish_timestamp: Jan. 23, 2022
url: https://www.codingforentrepreneurs.com/blog/django-github-actions/

---

Github actions is an amazing way to automate how your code is pushed into production. In this post, we'll explore how to automate running Django tests with a PostgreSQL database.

It's important to note that Github Actions are incredibly flexible and can really change how you or your team work but it's not the only tool. Gitlab's CI/CD Workflows are also excellent and do roughly the same thing we see below. 

Let's start by creating a workflow that tests our Django code. In the root of your project create the following folders:

```
mkdir -p .github/workflows/
```
Make sure that the `.github` folder (directory) is in the root of your git repo (next to `.git`) otherwise github actions will ignore the folder.

My project looks something like this:

```
.
├── .github
|   └── workflows
|       ├── django-test.yaml
├── .gitignore
├── requirements.txt
├── src
|   ├── manage.py
|   └── myproj
|       ├── __init__.py
|       ├── asgi.py
|       ├── settings.py
|       ├── urls.py
|       └── wsgi.py
└── venv
```

Now let's create our workflow.

### 1. Create `django-test.yaml` in `.github/workflows`
Yes, you can use `django-test.yaml` or `django-test.yml`


### 2. Add when to run this workflow
The complete example is at the bottom of the page but for now, we'll go step by step.

Update `django-test.yaml` with:

```yaml
name: Django Test with PostgreSQL Example

on:
    workflow_call:
    workflow_dispatch:
    push:
       branches: [main]
    pull_request:
        branches: [main]
```

Let's break this down:

- `name` is what you want to call this workflow
- `on` is the argument you use to tell this workflow when to run
- `workflow_call` is the value you set when you want to allow this workflow to run from another workflow within your repo. I'll show a basic example at the bottom
- `workflow_dispatch` this allows you to trigger this workflow right on github.com but also with API calls to github.com
- `push` this allows you to trigger this workflow when a `git push` happens. In this case, we also specified the branch `main` which means this triggers when `git push origin main` happens (replacing `origin` with whatever remote name you have for github)
- `pull_request` is identical to `push` except it's based on when a pull request occurs instead of a push.



### 3. Configure the Workflow Jobs

At the most basic level, your workflow can be just the following:

```yaml
name: Django Test with PostgreSQL Example

on:
    workflow_call:
    workflow_dispatch:
    push:
       branches: [main]
    pull_request:
        branches: [main]

jobs:
  simple_build:
    runs-on: ubuntu-latest
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v2
        with:
           python-version: 3.8
      - name: Install requirements
        run: |
            pip install -r requirements.txt
      - name: Run tests
        run: |
            cd src
            python manage.py test
```
Let's break this down:

- `jobs` can have many phases. In our case, our first and only phase is called `simple_build`. This name can be nearly anything as long as it's lower cased and has no spaces.
- `simple_build` name of the job
- `runs-on` select with Container Image (aka Docker Image) you want this phase to run on. `ubuntu-latest` is a great choice. It's a good idea to *test* you code on the same system your project will run on in production.
- `steps` this is the declaration for the list of steps you want this phase to take. They all run in order.
- `name: Checkout code` is merely the name of the step
- `uses: actions/checkout@v2` this will bring your code into the workflow environment. It's a required step
- `uses: actions/setup-python@v2` and `with: python-version: 3.8` allows you to setup the Python version you want to test this code on.
- `run: |` allows you to write multi-line commands as you see fit.
- `pip install -r requirements.txt` this will install all packages listed in my `requirements.txt` file in the root of my repo
- `cd src` and `python manage.py test` will change the current working directory to run this command within the root of my Django project where `manage.py` lives.


### 4. Using Environment Variables

One of the most common ways to use environment variables is to set them on the step you need them. Such as my `Run tests` step (aka **step-level**):

```yaml
      - name: Run tests
        env:
          DEBUG: "0"
          DJANGO_SECRET_KEY: ${{ env.DJANGO_SECRET_KEY }}
        run: |
            cd src
            python manage.py test
```
Let's break this down:

- `env` is the key name to add environment variables
- `DEBUG: "0"` is merely an environment variable I want to set for my Django project so I can use `os.environ.get("DEBUG")` wherever I need.
- `DJANGO_SECRET_KEY` is currently set to `${{ env.DJANGO_SECRET_KEY }}`. This is actually referencing another type of environment variable: **job-level** environment variables.


To use **job-level** environment variables we can do:

```yaml
jobs:
  simple_build:
    runs-on: ubuntu-latest
    env:
      DJANGO_SECRET_KEY: some-test-key-not-good-for-prod
      PYTHON_VERSION: 3.8
    steps:
      ...
```
Now in our `simple_build` job has two environment variables `DJANGO_SECRET_KEY` and `PYTHON_VERSION` that we can use on each step just by referencing `${{ env.DJANGO_SECRET_KEY }}` or `${{ env.PYTHON_VERSION }}`.  


### 5. Use a PostgreSQL Database with Django on Github Actions
Testing your project with the same database you'll use in production is a great idea. In this step, we'll see how to implement a `postgres` service within our workflow job.


```yaml
jobs:
  simple_build:
    runs-on: ubuntu-latest
    env:
      ...
      MY_POSTGRES_USER: someuser
      MY_POSTGRES_PASSWORD: somepassword
      MY_POSTGRES_DB: somedbname
    services:
      postgres_main:
        image: postgres:12
        env:
          POSTGRES_USER: ${{ env.MY_POSTGRES_USER }}
          POSTGRES_PASSWORD: ${{ env.MY_POSTGRES_PASSWORD }}
          POSTGRES_DB: ${{ env.MY_POSTGRES_DB }}
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready 
          --health-interval 10s 
          --health-timeout 5s 
          --health-retries 5
    steps:
      ...
```
Let's break this down

- `services` is the declaration you must use in order to have other images run during this workflow execution
- `postgres_main` is what I decided to name this service. It's more common to just call it `postgres` but I renamed it to make it more clear what's required and what's not.
- `image: postgres:12` This simply means what postgres version we want.
- `env` items `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` are the items we want to set for this image. We set them so we can re-use them within our Django Project. Notice how each one is set to a **job-level** environment variable. The default POSTGRES_HOST is `localhost`.
- `ports` you can change this mapping if you choose but it's not necessary for many projects
- `options` these are optional values that are commonly set for `postgres` on github actions. 


### 6. Update Django Settings for our workflow

In your Django `settings.py` add the following:

```python
import os

DB_USERNAME=os.environ.get("DB_USERNAME")
DB_PASSWORD=os.environ.get("DB_PASSWORD")
DB_HOST=os.environ.get("DB_HOST")
DB_PORT=os.environ.get("DB_PORT")
DB_DATABASE=os.environ.get("DB_DATABASE")
DB_IS_AVAIL = all([
        DB_USERNAME, 
        DB_PASSWORD, 
        DB_HOST,
        DB_PORT,
        DB_DATABASE
])

if DB_IS_AVAIL:
    DATABASES = {
        "default": {
            "ENGINE": "django.db.backends.postgresql",
            "NAME": DB_DATABASE,
            "USER": DB_USERNAME,
            "PASSWORD": DB_PASSWORD,
            "HOST": DB_HOST,
            "PORT": DB_PORT,
        }
    }
```
This means we need to update our workflow to include:

- `DB_USERNAME` 
- `DB_PASSWORD` 
- `DB_HOST`
- `DB_PORT`
- `DB_DATABASE`

At least on the step that runs `python manage.py test`

```yaml

...
   steps:
      ...
      - name: Run Tests
        env:
          DEBUG: "0"
          DJANGO_SECRET_KEY: ${{ env.DJANGO_SECRET_KEY }}
          DB_USERNAME: ${{ env.MY_POSTGRES_USER }}
          DB_PASSWORD: ${{ env.MY_POSTGRES_PASSWORD }}
          DB_HOST: localhost
          DB_DATABASE: ${{ env.MY_POSTGRES_DB }}
          DB_PORT: 5432
        run: |
          python manage.py test
...
```
This step should be pretty straightforward now. 


### 7. Next Steps

We left out a few steps that you might want to consider using:

- `python manage.py loaddata my_fixtures.json` to load in fixtures as a way to setup your database prior to running tests
- *Adding in Github Secrets* which will allow you to use sensitive information so you can have a step like `python manage.py collectstatic --noinput` to put your static files into production
- `python manage.py sendtestemail --admins` This might be another item you want to test in conjunction with Environment Variables and Github Secrets.
- Using `workflow_call` within another workflow to run this one (example below).



## Complete Example

`.github/workflows/django-test.yaml`

```yaml
name: Example CI Django & Postgres Tests

# Controls when the workflow will run
on:
  # Allows you to call this workflow within another workflow
  workflow_call:
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:
  # Triggered based on the git event type
  push:
    branches: [main]
  pull_request:
    branches: [main]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    # Add in environment variables for the entire "build" job
    env:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_HOST: localhost # default host value for the database
      POSTGRES_DB: djtesting
      POSTGRES_PORT: 5432
      DJANGO_SECRET_KEY: test-key-not-good
    services:
      postgres_main:
        image: postgres:12
        env:
          POSTGRES_USER: ${{ env.POSTGRES_USER }}
          POSTGRES_PASSWORD: ${{ env.POSTGRES_PASSWORD }}
          POSTGRES_DB: ${{ env.POSTGRES_DB }}
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready 
          --health-interval 10s 
          --health-timeout 5s 
          --health-retries 5
    # If you want to test multiple python version(s)
    strategy:
      matrix:
        python-version: ["3.8"] # ["3.8", "3.9", "3.10"]
    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install requirements
        run: |
          pip install -r requirements.txt
      - name: Run Tests
        # Step specific environment variables
        env:
          DEBUG: "0"
          DJANGO_SECRET_KEY: ${{ env.DJANGO_SECRET_KEY }}
          DB_USERNAME: ${{ env.POSTGRES_USER }}
          DB_PASSWORD: ${{ env.POSTGRES_PASSWORD }}
          DB_HOST: ${{ env.POSTGRES_HOST }}
          DB_DATABASE: ${{ env.POSTGRES_DB }}
          DB_PORT: ${{ env.POSTGRES_PORT }}
        run: |
          python manage.py test
```



### Sample `workflow_call` 
`.github/workflows/container-build.yaml`

```yaml
name: Build Docker Container after Django Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test_django:
    uses: your-username/your-repo/.github/workflows/django-test.yaml@main
  container_builder:
    runs-on: ubuntu-latest
    needs: [test_django]
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Build container image
        run: |
          docker build \
          -t your-username/your-image-name:${GITHUB_SHA::7} \
          -t your-username/your-image-name:latest \
          .
```
This sample is mostly incomplete but shows that you can run our testing workflow first. That's the `test_django` portion. 

Pretty neat huh? Any questions? Comments? Let us know.