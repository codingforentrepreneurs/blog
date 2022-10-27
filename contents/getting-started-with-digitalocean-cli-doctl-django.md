---
title: Getting Started with DigitalOcean CLI `doctl` &amp; Django
slug: getting-started-with-digitalocean-cli-doctl-django

publish_timestamp: Sept. 19, 2021
url: https://www.codingforentrepreneurs.com/blog/getting-started-with-digitalocean-cli-doctl-django/

---

In [Try Django 3.2](https://www.codingforentrepreneurs.com/projects/try-django-3-2) we cover how to deploy to DigitalOcean App Platform via the console. In this blog post, we're going to cover how to deploy via the command line tool called `doctl`.

There are a few reasons to learn about the command line tool but the primary one is this: monitoring app changes via git.

As you know, git is an amazing tool that allows us to track changes in code. Should the services our deployed applications use be any different?  No, of course not.

When you use the console to manage your apps, tracking down the history of changes is a bit more complex to do -- not to mention time consuming.

Using the `doctl` tool from DigitalOcean helps simply deployment & tracking of app-level service changes.


### 1. Register for a DigitalOcean account & get a $100 credit [here](https://do.co/cfe-sh).


###  2. Install `doctl` following [this guide from DigitalOcean](https://docs.digitalocean.com/reference/doctl/how-to/install/)

If youre on _macOS with Homebrew_ you can:
```
brew install doctl
```
Everyone else, install [here](https://docs.digitalocean.com/reference/doctl/how-to/install/).


###  3. Get an API Token at:

[https://cloud.digitalocean.com/account/api/tokens](https://cloud.digitalocean.com/account/api/tokens)



###  4. Initialize API CLient:
Copy your api token from step 3 and run
```
doctl auth init
```

### 5. Create the App Spec File (`app.yaml`)
Here's the [reference](https://docs.digitalocean.com/products/app-platform/references/app-specification-reference/) docs for the App Specification. 

Below does the following:

- Initializes a DigitalOcean managed database
- Creates a python-based service (ie to run our try django project)
- Runs a one-off job (during deployment) for our `python-based` service.
- Configures app-level environment variables

```yaml
name: Try Django 3.2
databases:
    - engine: PG
      name: db
      num_nodes: 1
      size: db-s-dev-database
      version: "12"
services:
    - environment_slug: python
      envs:
        - key: DATABASE_URL
          scope: RUN_TIME
          value: ${db.DATABASE_URL}
      git:
        branch: production-3
        repo_clone_url: https://github.com/codingforentrepreneurs/Try-Django-3.2.git
      http_port: 8080
      instance_count: 1
      instance_size_slug: basic-xxs
      name: try-django-3-2
      routes:
        - path: /
      run_command: gunicorn --worker-tmp-dir /dev/shm trydjango.wsgi
      source_dir: /
jobs:
    - environment_slug: python
      envs:
        - key: DATABASE_URL
            scope: RUN_TIME
            value: ${db.DATABASE_URL}
      git:
        branch: production-3
        repo_clone_url: https://github.com/codingforentrepreneurs/Try-Django-3.2.git
      instance_count: 1
      instance_size_slug: basic-xxs
      kind: PRE_DEPLOY
      name: django-migrate-job
      run_command: python manage.py migrate --noinput
      source_dir: /
envs:
    - key: DISABLE_COLLECTSTATIC
      scope: RUN_AND_BUILD_TIME
      value: "1"
    - key: DEBUG
      scope: RUN_AND_BUILD_TIME
      value: "0"
    - key: DJANGO_ALLOWED_HOST
      scope: RUN_AND_BUILD_TIME
      value: .ondigitalocean.app
    - key: DJANGO_SUPERUSER_EMAIL
      scope: RUN_AND_BUILD_TIME
      value: hello@teamcfe.com
    - key: DJANGO_SUPERUSER_USERNAME
      scope: RUN_AND_BUILD_TIME
      value: cfe
    - key: DJANGO_SECRET_KEY
      scope: RUN_AND_BUILD_TIME
      type: SECRET
      value: hellosecret
    - key: DJANGO_SUPERUSER_PASSWORD
      scope: RUN_AND_BUILD_TIME
      type: SECRET
      value: ThisIsDOCLI_Deploy
    - key: POSTGRES_DB
      scope: RUN_AND_BUILD_TIME
      value: ${db.DATABASE}
    - key: POSTGRES_HOST
      scope: RUN_AND_BUILD_TIME
      value: ${db.HOSTNAME}
    - key: POSTGRES_USER
      scope: RUN_AND_BUILD_TIME
      value: ${db.USERNAME}
    - key: POSTGRES_PASSWORD
      scope: RUN_AND_BUILD_TIME
      value: ${db.PASSWORD}
    - key: POSTGRES_PORT
      scope: RUN_AND_BUILD_TIME
      value: ${db.PORT}
    - key: AWS_ACCESS_KEY_ID
      scope: RUN_AND_BUILD_TIME
      type: SECRET
      value: abc123
    - key: AWS_SECRET_ACCESS_KEY
      scope: RUN_AND_BUILD_TIME
      type: SECRET
      value: abc1234
    - key: AWS_STORAGE_BUCKET_NAME
      scope: RUN_AND_BUILD_TIME
      value: trydjango3
```


### 6. Key Commands:


__Create__
```bash
echo "$(doctl apps create --spec app.yaml --format ID --no-header)" > app-id.txt
```
> I use this command so I can easily reference the created app's `ID`. You can always run  `doctl apps create --spec app.yaml` if you need.

If you intend to use the *exact same* setup for many apps, be sure to change the `name` in `app.yaml`. from above.


__Update__
```
doctl apps update "$(cat app-id.txt)"
```
> This command works because in __Create__ I added the `app-id.txt` file.



__Delete__
```
doctl apps delete "$(cat app-id.txt)" && rm app-id.txt
```
> This command works because in __Create__ I added the `app-id.txt` file.

I run `rm app-id.txt` because I no longer need it after the app is deleted. 

__list__

```
doctl apps list --format "Spec.Name, ID"
```