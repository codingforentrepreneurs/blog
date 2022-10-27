---
title: Deploy Django to DigitalOcean App Platform
slug: deploy-django-to-digitalocean-app-platform

publish_timestamp: July 27, 2021
url: https://www.codingforentrepreneurs.com/blog/deploy-django-to-digitalocean-app-platform/

---


In [Prepare Django 3  for DigitalOcean App Platform](https://www.codingforentrepreneurs.com/blog/prepare-django-for-digital-ocean-app-platform) we used the `production-1` branch which is what we'll use here.

The goal of this post is to deploy the first production-ready version of the [Try Django 3.2](https://www.codingforentrepreneurs.com/projects/try-django-3-2) project. 

## Step 1: Deploy an App Component
This setup guide is a simple way to deploy your project without touching code.

### 1. Sign up on [Digital Ocean](https://do.co/cfe-sh) (sign up a new account for a $100 promo)

### 2. [Sign in](https://github.com/login) or [sign up](https://github.com/signup?source=cfe.sh) for Github.

### 3. Fork [Try Django 3.2](https://github.com/codingforentrepreneurs/Try-Django-3.2) or [click here to fork](https://github.com/codingforentrepreneurs/Try-Django-3.2/fork).

### 4. Create new [app on App Platform](https://cloud.digitalocean.com/apps/new)

1. Under Source, Select `Github`

2. Click `Next`

2. Under *Repository*, Select your forked `Try Django 3.2` repo

3. Under *Branch*. Select `production-1` (this branch has been designed to handle everything in this guide with no code changes)

4. Ensure `Autodeploy code changes` is **selected**

> It's easy to change a branch later, but not any of the other settings.

### 5. Configure your app 
Use the following settings:

**Source Directory**: `/` (no change needed)

**Type**: `Web Services` (no change needed)

**Environment Variables**:
Leave blank. We'll add our environment variables as app-level environment variables.


**Build Command**: 
Leave blank. We'll create a *job* for running the various build commands later.


**Run Command**:
```
gunicorn --worker-tmp-dir /dev/shm trydjango.wsgi
```
> `trydjango.wsgi` is from our Django project referencing `trydjango/wsgi.py` ([code](https://github.com/codingforentrepreneurs/Try-Django-3.2/blob/main/trydjango/wsgi.py))

**HTTP Port**: `8080` (no change needed)

**HTTP Request Routes**: `/` (no change needed)


## Step 2. Provision App-Level Environment variables. 
App Platform treats your web application (like above) as one component of your entire app. Another component is your database. Another component is your static files server. Another is a data store like redis. 

Given this, we have 2 places to add environment variables:

- component-level
- app-level

For Django projects, we're going to use the `app-level` environment variables. 

**Update App Settings**
1. Go to [https://cloud.digitalocean.com/apps](https://cloud.digitalocean.com/apps)

2. Click on your project (mine is try-django-3-2)

3. Click **Settings**

4. Scroll down to **App-Level Environment Variables** (if you don't see `App-Level`), you're in the wrong place.

5. Click **edit** to add the following key/value pairs
- `DEBUG` = `0`
- `DISABLE_COLLECTSTATIC` = `1`
- `DJANGO_ALLOWED_HOST` = `.ondigitalocean.app`
- `DJANGO_SECRET_KEY` = `create-a-secure-one` (& choose `encrypt`)
- `DJANGO_SUPERUSER_EMAIL` = `your@email.com`
- `DJANGO_SUPERUSER_USERNAME` = `yourusername`
- `DJANGO_SUPERUSER_PASSWORD` = `create-a-secure-one` (& choose `encrypt`)

> To make secure keys/passwords, consider [this post](https://www.codingforentrepreneurs.com/blog/create-a-one-off-django-secret-key/).

6. Click **Save**.


> After you change these settings your project will be rebuilt all over again (literally every time).

## Step 3. Provision Database
Or, simply **Add a Component** to your project:

1. Go to [https://cloud.digitalocean.com/apps](https://cloud.digitalocean.com/apps)

2. Click on your project (mine is try-django-3-2)

3. Click **Settings**

4. Click **+ Add Component**.

5. Click **Database**

6. Under **Choose Name** you can leave the default `db`. If you change the name, be sure to *remember the change for the next part*.

7. Click **Create and attach**

8. Navigate back to update your **App-Level Environment Variables** with the following:
- `POSTGRES_DB` = `${db.DATABASE}`
- `POSTGRES_HOST` = `${db.HOSTNAME}`
- `POSTGRES_USER` = `${db.USERNAME}`
- `POSTGRES_PASSWORD` = `${db.PASSWORD}`
- `POSTGRES_PORT` = `${db.PORT}`

> In each value above (ie `${db.HOSTNAME}`), the `db` portion is directly related to the name you chose in step 6. So you if you used `mydb` update each database environment variable to reflect that (such as `${db.DATABASE}` as `${mydb.DATABASE}`).

9. Click **Save**


## Step 4. Create a pre-deploy Job
It's a good idea to run `python manage.py migrate` prior to deploying a new version of your code -- this ensures your database(s) and code are in sync.

In App Platform, we'll do this using a *Job* Component with your app project.

1. Go to [https://cloud.digitalocean.com/apps](https://cloud.digitalocean.com/apps)

2. Click on your project (mine is `try-django-3-2`)

3. Click **Settings**

4. Click **+ Add Component**.

5. Click **Job**

6. Under **Choose Source**, Click `Github` 

7. Under **Repository**, Select your forked `Try Django 3.2` repo (this will be same source as *Step 1* from above)

8. Under **Branch**. Select `production-1` (this branch has been designed to handle everything in this guide with no code changes)

9. Click **Next**

10. Under **Type Jobs**, click **Edit** and change **When to Run** to `Before every Deploy`

11. Under **Run command**, click **Edit** and add:

```
python manage.py migrate --noinput
python manage.py createsuperuser --noinput --email hello@teamcfe.com
```

> The reason `python manage.py createsuperuser --noinput --email <youremail>` works, has to do with `DJANGO_SUPERUSER_USERNAME`, and `DJANGO_SUPERUSER_PASSWORD` in your `app-level` environment variables (see [docs](https://docs.djangoproject.com/en/3.2/ref/django-admin/#createsuperuser)).

12. Click **Next**

13. Under *Name your job** call it `django migrate` or something similar 

14. Click **Next**

15. Select the minimum resources to run this job. (mine was Basic with $0.007/hr container size)

16. Click **Launch**

## Step 5. Force Rebuild & Deploy

1. Go to [https://cloud.digitalocean.com/apps](https://cloud.digitalocean.com/apps)

2. Click on your project (mine is `try-django-3-2`)

3. Under **Actions** click **Force Rebuild.& Deploy**

4. Wait about 10-15 minutes

5. Open your app and verify!

## Next step: Static Files 
** Coming soon*
