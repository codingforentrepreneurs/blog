---
title: Django x MongoDB
slug: django-x-mongodb

publish_timestamp: Sept. 15, 2020
url: https://www.codingforentrepreneurs.com/blog/django-x-mongodb/

---

Here's a simple guide to using MongoDB with Django. For this post, I'll be using macOS. If you have a different OS use the install docs [here](https://docs.mongodb.com/manual/administration/install-community/)


## 1. Install [Homebrew](https://brew.sh/)
Open `Terminal` and run:

```shell
% /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

Verify:
```shell
% brew --version
Homebrew 2.5.1
```

## 2. Install MongoDB

### 1. Tap MongoDB 

```shell
% brew tap mongodb/brew
```


### 2. Install MongoDB with homebrew

```shell
% brew install mongodb-community
```

### 3. Start Homebrew service for MongoDB

```shell
% brew services start mongodb/brew/mongodb-community
```

> Or, if you don't want/need a background service you can just run: `mongod --config /usr/local/etc/mongod.conf`

### 4. Verify installation

```shell
% mongo --version
MongoDB shell version v4.4.0
Build Info: {
    "version": "4.4.0",
    "gitVersion": "563487e100c4215e2dce98d0af2a6a5a2d67c5cf",
    "modules": [],
    "allocator": "system",
    "environment": {
        "distarch": "x86_64",
        "target_arch": "x86_64"
    }
}
```

## 3. MongoDB Basics

#### `use DB_NAME`
I think of this as a *get or create* command.
```shell
% mongo
> use cfeOnGo
```

#### `show dbs`
```shell
> show dbs
admin    0.000GB
config   0.000GB
local    0.000GB
```
At this point, our `cfeOnGo` has yet no information so it's not just listed yet.

#### `show collections`
`collections` are essentially `tables` in SQL.
```mongo
> use admin
> show collections
system.roles
system.users
system.version
```

#### `query collection`
I'll use the `system.users` collection. 

*find all*
```mongo
> db.system.users.find({}).pretty()
```
> remove `pretty()` for a more concise response

*find by key/value pair*
```mongo
> db.system.users.find({user:"djangoDBUser", db: "admin"}).pretty()
```


## 4. Create your MongoDB admin user & admin database:

```mongo
> use admin
> db.createUser(
  {
    user: "cfeDbAdmin",
    pwd: passwordPrompt(), // or cleartext password
    roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
  }
)
> quit()
```

Now that we have an admin user, we need to update our default configuration:
```shell
% sudo nano /usr/local/etc/mongod.conf
```
Add the following:
```
security:
    authorization: enabled
```

Now restart mongodb
```shell
% brew services restart mongodb/brew/mongodb-community
```

Now, this is how you'll login:
```shell
% mongo -u "cfeDbAdmin"
```
> This will prompt you for your password

To be more explicit about logging into mongo, use:
```shell
% mongo --port 27017  --authenticationDatabase "admin" -u "cfeDbAdmin" -p
```



## 5. Create Mongo User for Django DB & Project

Let's make 1 user and 1 database for our django project. Generally, you'll repeat this for each new Django project in the future.

Here's what we'll use:

```
user_database='admin'
database_user='djangoDBUser'
django_database_name='cfeOnGo'
```


The `database_user` will be made to manage the `django_database_name` database and only that database (ideally).

The `user_database` is where you'll want to create all mongodb users in the future for future projects. It's your ***authentication*** database.


#### Login
```shell
% mongo -u "cfeDbAdmin"
```

#### Select database, create user
```mongo
> use admin
> db.createUser(
  {
    user: "djangoDBUser",
    pwd: passwordPrompt(),  // or cleartext password
    roles: [
       { role: "readWrite", db: "cfeOnGo" }
    ]
  }
)
```
> Since I'm just working on local development, I just used the password `learncode`. 

#### Add Django/Djongo Required Role
Django/Djongo users need the `listCollections` privilege so we need to create a role for our newly created user.

```mongo
> db.createRole(
   {
     role: "listCollectionsRole",
     privileges: [
       { resource: { db: "cfeOnGo", collection: "" }, actions: [ "listCollections" ] }
    ],
     roles: [
       { role: "readWrite", db: "cfeOnGo" }
     ]
   }
)
> db.grantRolesToUser("djangoDBUser", ["listCollectionsRole"] )
```


## 6. Django & Djongo setup

[Djongo](https://github.com/nesdis/djongo) is the great package that allows us to use all the native Django features with a MongoDB backend. ([docs](https://github.com/nesdis/djongo))


```shell
% cd path/to/dev/folder
```

Create virtual environment
```shell
% pipenv install django djongo
```
> Yes `pipenv` is optional. You just need a Python virtual environment

Activate virtual environment
```shell
% pipenv shell
```

Start django project
```shell
django-admin startproject .
```

In `settings.py` add:
```python
DATABASES = {
    'default': {
        'ENGINE': 'djongo',
        'NAME': 'cfeOnGo',
        'CLIENT': {
            'host': 'localhost',
            'port': 27017,
            'username': 'djangoDBUser',
            'password': 'learncode',
            'authSource': 'admin',
            'authMechanism': 'SCRAM-SHA-1'
        }
    }
}
```

Migrate database
```shell
python manage.py migrate
```

Create a Django superuser
```shell
python manage.py createsuperuser
```

Now, in your all `models.py` you'll, use 

```python
from djongo import models
```
Instead of
```python
from django import models
```

Now, you can use the Djongo reference [here](https://nesdis.github.io/djongo/get-started/) to build out your MongoDB-based Django projects.