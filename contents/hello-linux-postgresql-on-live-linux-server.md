---
title: Hello Linux: PostgreSQL on Live Linux Server
slug: hello-linux-postgresql-on-live-linux-server

publish_timestamp: March 14, 2019
url: https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server/

---

<div class='alert alert-success'>This is the third post of a many part series. <a href='https://www.codingforentrepreneurs.com/blog/hello-linux/'>This post</a> is the starter post for the whole series.</div>


PostgreSQL is a production database. It's powerful and integrates with a lot of different kinds of web frameworks. Our goal is to use Django but you could easily use Flask, Masonite, aiohttp, and many others.

> Don't have a live server? Pick one up for $5/mo on [Digital Ocean](https://kirr.co/l8v1n1) ([no referral](https://www.digitalocean.com/))

Once you set it up 1 time successfully. You'll never want to user another method (I'm looking at you SFTP).

Let's do this.

*********
**Requirments**

Be sure to complete
- [Git Push Local Code to Live Linux Server](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Virtual Environment in Working Directory](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)

**Our Live Server**
- **Ubuntu 18.04**

*********

> It's recommended to use the same database technology locally as you use in production. So yeah, you'll want to install PostgreSQL on your development/local environment as well.

For the **entire post**, we'll be using the live **Ubuntu 18.04** server for later integration to **Django** so we'll have Django-specific information as well.


#### 1. SSH into your server

```
ssh root@104.248.231.241
```
Replace `root@104.248.231.241` with your user / ip



#### 2. Update & Install on Server

```
sudo apt-get update -y

sudo apt-get install postgresql postgresql-contrib -y
```


#### 3. Create Default User for postgreSQL

```
cd  /tmp

sudo -u postgres createuser $USER

sudo -u postgres createdb $USER
```
If you're not familiar with `$USER` in the command line, it defaults to your currently logged in user.


#### 4a. Create Database. The Bash Method

Setup your defaults:

```
projectDB="your_project_db_name"
projectDBuser="your_project_db_username"
dbUserPassword="your_db_user_password"
```

**The following commands will**

- Create a database
- Create a user with the password from above
- Alter the user's role to fit with the requirements of Django
- Grant all privileges for this user to do what's needed on the database in PostgreSQL

```
sudo -u postgres psql --command="CREATE DATABASE ${projectDB};"

sudo -u postgres psql --command="CREATE USER ${projectDBuser} WITH PASSWORD '${dbUserPassword}';"

sudo -u postgres psql --command="ALTER ROLE ${projectDBuser} SET client_encoding TO 'utf8';"

sudo -u postgres psql --command="ALTER ROLE ${projectDBuser} SET default_transaction_isolation TO 'read committed';"

sudo -u postgres psql --command="ALTER ROLE ${projectDBuser} SET timezone TO 'UTC';"

sudo -u postgres psql --command="GRANT ALL PRIVILEGES ON DATABASE ${projectDB} TO ${projectDBuser};"
```



#### 4b. Create Database. The `psql` method
This is just another way to do part `4a`

**Enter a postgres session**
```
sudo -u postgres psql
```

You now should see `postgres@localhost:` before `$` on the command line. Something like `postgres@localhost:~$`. It's okay if it's different but it should not be the same as the normal bash shell.

Now, in the postgres session create your database

```
CREATE DATABASE your_database_name;
```

Create user:
```
CREATE USER your_database_user WITH PASSWORD 'database_user_password';
```
Be sure to include the single quotes `'` around the password and not around the `your_database_user`


Alter User role to prep for Django:
```
ALTER ROLE your_database_user SET client_encoding TO 'utf8';
ALTER ROLE your_database_user SET default_transaction_isolation TO 'read committed';
ALTER ROLE your_database_user SET timezone TO 'UTC';
```

Grant privileges
```
GRANT ALL PRIVILEGES ON DATABASE ${projectDB} TO ${projectDBuser};
```

Exit postgres
```
\q
```

#### 5. Update Your Django `production settings`
See [this post](https://www.codingforentrepreneurs.com/blog/staging-django-production-development) for more on **production** vs **development**settings.

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': '<projectDB>',
        'USER':  '<projectDBuser>',
        'PASSWORD': '<newPassword>',
        'HOST': 'localhost',
        'PORT': '',

    }
}
```

Replace `<projectDB>`, `<projectDBuser>`, and `<dbUserPassword>` with the correct values.




#### 6. Run migrations & Create Admin User.
```
python manage.py migrate
python manage.py createsuperuser
```
That's it. Your Django project should now be setup.



#### Wrap Up

**We now have**
- [Git Setup for Push/Pulling Code From Local to Production](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Working Directory with a Virtual Environment](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- PostgreSQL on Live Linux Server (this post)

**Now we need to**
- [Setup Gunicorn and Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor)