---
title: CI/CD Github Workflow for Django &amp; DigitalOcean App Platform
slug: github-workflow-django-digitalocean-app-platform

publish_timestamp: Sept. 19, 2021
url: https://www.codingforentrepreneurs.com/blog/github-workflow-django-digitalocean-app-platform/

---

Whenever you aim to release code into production, it's best that you follow pre-defined steps to ensure your application is working as intended.

Perhaps the most important step is to run your automated tests. Below is a github workflow to help you do exactly that.

After we learn to run a github workflow to test our Django code, we'll implement a step to trigger an update on our production application.

Let's keep in mind that this is meant to be a basic working example. You can, and probably should, look into automating more tasks (such as linting, coverage reports, collecting static files, building JS apps, etc) using github workflows.

#### Requirements
- A DigitalOcean account: sign up [here](https://do.co/cfe-sh)) & get a $100 credit.
- A DigitalOcean API Token from 
- A Github Account
- Forked copy of [https://github.com/codingforentrepreneurs/Try-Django-3.2] including the `production-3` branch.


### 1. Store your DigitalOcean API Token

For Github Actions, you need to add the following Repository Secrets (located in your repo's settings under `settings/secrets/actions`):

- `PERSONAL_ACCESS_TOKEN` this is your Github Personal Access token. Create one [here](https://github.com/settings/tokens).
- `DO_ACCESS_TOKEN` this is your DigitalOcean access token created [here](https://cloud.digitalocean.com/account/api/tokens ).
- `DO_APP_ID`:  Your App Platform id -- Get this with `doctl apps list --format "Spec.Name, ID"` What's `doctl`? Check out [this guide](https://www.codingforentrepreneurs.com/blog/getting-started-with-digitalocean-cli-doctl-django).


### 2. Setup the continuous integration (`CI`) for Django using Postgres & Running Tests

`.github/workflows/main.yaml`
```yaml
# name of our workflow
name: Django CI/CD Workflow

# triggers for our workflow
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    services:
      postgres_main:
        image: postgres:12
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: djtesting
        ports:
          - 5432:5432
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
    steps:
        - name: Checkout code
          uses: actions/checkout@v2
        - name: Set up Python 3.6
          uses: actions/setup-python@v2
          with:
              python-version: 3.6
        - name: Install requirements
          run: pip install -r requirements.txt
        - name: Run Tests
          env:
            DJANGO_SECRET_KEY: CI_CD_TEST_KEY
            POSTGRES_DB: djtesting
            POSTGRES_HOST: localhost
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: postgres
            POSTGRES_PORT: 5432
            DEBUG: "0"
          run: |
            python manage.py test
```
One of the biggest keys here is that the `POSTGRES_HOST` is set to `localhost` and not the workflow service declaration `postgres_main`. Much of the service configuration emulates `docker-compose` but it's not an exact match while using.


### 3. Push Code to a new Branch.
_Append this to the end of your steps above_
```yaml
        - name: Push main branch into production
          uses: codingforentrepreneurs/action-branch-to-branch@main
          with:
              dest_branch: 'production-3'
              source_branch: 'main'
              commit_message: 'Release production version'
          env:
              GITHUB_TOKEN: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
```
The step above will override your destination branch (`dest_branch`) and uses [this action](https://github.com/codingforentrepreneurs/action-branch-to-branch). I only do this as a way to decide which branch I want digitalocean to run on (assuming it's different than `main`).

### 4. Add the deployment step (`cd`) for DigitalOcean `doctl`

In [this post](https://www.codingforentrepreneurs.com/blog/getting-started-with-digitalocean-cli-doctl-django) we show you the basics of using `doctl` and an App Specification file for DigitalOcean services. In our case here, we're using App Platform but you can adapt below (your your `app.yaml` file) for any of DigitalOcean's services.


_Append this to the end of your steps above_
```yaml
        - name: Install doctl
          uses: digitalocean/action-doctl@v2
          with:
            token: ${{ secrets.DO_ACCESS_TOKEN }}
        - name: Update DigitalOcean App Platform
          run: doctl apps update ${{ secrets.DO_APP_ID }} --spec app.yaml
```