---
title: Go Live with Django &amp; Heroku
slug: go-live-with-django-project-and-heroku

publish_timestamp: July 29, 2020
url: https://www.codingforentrepreneurs.com/blog/go-live-with-django-project-and-heroku/

---


Make your Django project live using [Heroku.com](https://www.heroku.com)'s web hosting services. This uses Heroku's standard build process.

Interested in Docker containers for Django on Heroku? Check it out [here](https://www.codingforentrepreneurs.com/blog/deploy-django-on-docker-to-heroku-opencv).

_Updated on July 29th, 2020. Originally posted in 2017._

-------

### 1. Stage Django Project for Production
Overall, you need to be able to use different settings modules at least 1 for development and production. Here's a useful [guide](/blog/staging-django-production-development) on how to do it.

Here's [another guide option](/blog/create-a-blank-django-project/) for a production-ready Blank Django project.

### 2. Setup Production-ready Security Settings
When in production, you need a number of security settings setup. Here's our post on Django [SSL/TLS in Production](/blog/ssltls-settings-for-django/).

### 3. Add git for version control and deployment
You can do so with [this guide](/blog/setup-git-github-repo/) or watch [here](https://www.youtube.com/watch?v=dlz6Qyp4ADE).

### 4. In your project, ensure git repo exists:
```
$ cd /path/to/your/venv/mvp-landing/
$ ls
mvp_landing manage.py ...
$ git status
fatal: Not a git repository (or any of the parent directories): .git
```
If you see exactly above do this:

```
$ git init
$ git status
mvp_landing/
manage.py
```

### 5. Install Heroku CLI & Login
Download and install Heroku Command Line Interface (heroku cli): [here](https://devcenter.heroku.com/articles/heroku-cli)
```
$ heroku login
```

### 6. Create Heroku Project

##### `heroku create <your-project-name>`
Creates a heroku project you declare so replace `project-name` below with yours.
```
$ heroku create project-name
https://project-name.herokuapp.com |  https://git.heroku.com/project-name.git
```

##### `heroku create`
Creates a random project name.
```
$ heroku create
https://remembrance-inukshuk-24674.herokuapp.com/ | https://git.heroku.com/remembrance-inukshuk-24674.git
```

### 7. Create `.gitignore` and `Procfile`
`gitignore` is so you don't push all files your local system as
`Procfile` is for heroku projects that are not [container-based](/blog/django-docker-production-heroku).

```
$ echo ".py[cod]" > .gitignore
$ echo "web: gunicorn mvp_landing.wsgi" > Procfile
```

`gitignore`: Add the contents from [this gitignore](https://raw.githubusercontent.com/codingforentrepreneurs/Try-Django-1.11/master/.gitignore) to your *.gitignore* from above. Be sure to ignore any *local*, **non-production**, settings modules you may have. 

Replace **mvp_landing** above with your project name or where the `.wsgi` file exists


### 8. Provision Database Add-on:
```
heroku addons:create heroku-postgresql:hobby-dev
```




### 9. Use Heroku Environment Variables for Django keys:
Create a one-off Django `SECRET_KEY` with [this guide](/blog/create-a-one-off-django-secret-key).
```
$ heroku config:set SECRET_KEY=<your-django-secret-key>
$ heroku config:set EMAIL_HOST_PASSWORD = 'youremailpassword'
```

### 10. Update your Django Production Settings `DJANGO_SETTINGS_MODULE` to include:

```python
DEBUG = False
ALLOWED_HOSTS =  ['project-name.herokuapp.com', '.yourdomain.com']
SECRET_KEY = os.environ.get('SECRET_KEY', 'SOME+RANDOM+KEY(z9+3vnm(jb0u@&w68t#5_e8s9-lbfhv-')  
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

# add this
import dj_database_url
db_from_env = dj_database_url.config()
DATABASES['default'].update(db_from_env)
DATABASES['default']['CONN_MAX_AGE'] = 500
```


### 11. Scale Dyno:
```
heroku ps:scale web=1
```
### 12. Disable Collectstatic
Collectstatic runs on *every* push unless you disable it. I recommend disabling it.

I recommend using [S3 for static files](/blog/s3-static-media-files-for-django/). 
```
heroku config:set DISABLE_COLLECTSTATIC=1
```

### 13. Update requirements.txt
```
$ pip freeze > requirements.txt
$ git add requirements.txt
$ git commit -m "Updated requirements.txt"
```
Heroku requirements:
- Django: `pip install django`
- psycopg2: `pip install psycopg2-binary`
- dj-database-url: `pip install dj-database-url`
- gunicorn: `pip install gunicorn`

```
$ pip install django psycopg2-binary dj-database-url gunicorn
$ pip freeze > requirements.txt
```
> You can absolutely use `pipenv` instead.

### 14. Add Python Runtime
```
$ python -V # notice the capital V
3.6.2
```
Create a file (next to `manage.py` and `requirements.txt`) called `runtime.txt` with the contents:
```
python-3.6.2
```
The numbers are directly releated to the `python -V` call

### 15. Commit & Push
```
$ git add --all
$ git commit -m "Initial Heroku commit"
$ git push heroku master
```




### 17. Run migrations
```
$ heroku run python manage.py migrate
```

### 18. Other common commands:

- Create superuser: `heroku run python manage.py createsuperuser`
- Enter Heroku's bash: `heroku run bash` for shell access
- Live Python-Django shell: `heroku run python manage.py shell`

### 19. Get Heroku configuration:
```
$ heroku config
```

<hr/>

# Custom Domains & SSL on Heroku

### 1. Add a Custom Domain (buy one [here](https://kirr.co/babeaj))
```
heroku domains:add <your-custom-domain.com>
heroku domains:add www.<your-custom-domain.com>
```

### 2. Update your DNS to what heroku says above

### 3. Upgrade your Hosting account with at least a Hobby account:
```
heroku ps:resize web=hobby
```

### 4. Enable Heroku to Handle Let's Encrypt Certificate 
```
heroku certs:auto:enable
```
More on this from the Heroku docs [here](https://devcenter.heroku.com/articles/automated-certificate-management).

-------
## Video
Watch how this is done on [youtube](https://www.youtube.com/embed/4DggiEkbCTg). This video is the older method on how to do for a newer one, consider the [MVP Landing](https://cfe.sh/courses/mvp-landing) project's Heroku deployment tutorial.
