---
title: RDS Database for Serverless Django + Zappa on AWS Lambda
slug: rds-database-serverless-django-zappa-aws-lambda

publish_timestamp: Nov. 9, 2018
url: https://www.codingforentrepreneurs.com/blog/rds-database-serverless-django-zappa-aws-lambda/

---

This is a critical piece of having a serverless Django application. There are several moving parts that you might have to do several times before you get it right. I know I did.

Please comment below if you find solutions that we missed or experience errors different from below. 

##### Requirements
- Complete [Prepare AWS IAM User, Role, and Policies for Zappa and Serverless Python](https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python)
- Complete [Serverless Django with Zappa on AWS Lambda](https://www.codingforentrepreneurs.com/blog/serverless-django-with-zappa-on-aws-lambda)



## 1. Create RDS Database
- Go to AWS [RDS](https://us-west-2.console.aws.amazon.com/rds/)
- Click "Create database"
- Select the option "Only enable options eligible for RDS Free Usage Tier" -- it should be at the bottom of the page
- Choose `PostgreSQL` or `MySQL` as the bindings for Django are simple. I'll be using `PostgreSQL`
- Ensure `Free tier` is selected. As of now, it's the `db.t2.micro` **DB instance class**. Using the `Free tier` while testing is *highly recommended*.

##### Add database settings

The below settings are what I used. **Keep these safe** Keep in mind that you'll want to have a decent unique password. I suggest using lowercase for username and database name.


**DB instance identifier**: zapdjangodb

**Master username**: zapdjangouser

**Master password**: 15422715-547f-4eec-93b5-faa4f90e1d48

**Confirm password**: 15422715-547f-4eec-93b5-faa4f90e1d48


> How do you create a UUID4 string? 
```
(zappaCFE-LnWEDIxd) bash-3.2$ python 
Python 3.6.6 (v3.6.6:4cf1f54eb7, Jun 26 2018, 19:50:54) 
[GCC 4.2.1 Compatible Apple LLVM 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import uuid
>>> uuid.uuid4()
UUID('15422715-547f-4eec-93b5-faa4f90e1d48')
```

- Click "Next"
- On "Configure advanced settings": you don't need to change anything unless you know what you're doing. 
- Click "Create database"


###### >>> The creation process will take several minutes

While it's creating, look for the following settings in the tab `Details`:


**VPC** my value is:

vpc-221e2845


**Security groups** my value is:
rds-launch-wizard-1 (sg-0cab7e25f83e3e46b)

The key we need is `sg-0cab7e25f83e3e46b`

**Subnets** my values are:
- subnet-449a6c1f
- subnet-55c3a61c
- subnet-779fdd10

Or, as a list we'll use in a bit: `["subnet-449a6c1f", "subnet-55c3a61c", "subnet-779fdd10"]`


Once AWS is *done creating* your new db, make note of the `endpoint` it will look like: 

`zapdjangodb.cfak8zfgryqc.us-west-2.rds.amazonaws.com`

Is your port `5432`? If not, make note of it for later.


## 2.  Add the Security Group to Lambda
- Go to [AWS Lamda](https://console.aws.amazon.com/lambda/home)
- Ensure your in the right region (I am using `us-west-2` so it says `Oregon` in the top right next to `Support` and my name)
- Open sidebar, click `Functions` 
- Navigate to your Zappa Deployment. Mine is named `zappacfe-dev`, which is the same name as my pipenv with `-dev` appended. Click on the function name.
- Scroll all the way down to `Network`
- In `VPC` select the **VPC** of your RDS DB (done above)
- Add your `Subnets` (again from above); I suggest all 3 of them
- In `Security groups` be sure to use the same value as your **Security groups** above.
- Save settings.

## 3. Add Inbound Rule to Security Group
- Go to [AWS RDS](https://console.aws.amazon.com/rds/home)
- Open sidebar, click `Instances`
- Select your db instance, ours is `zapdjangodb`
- Scroll to `Details`, under `Security groups` select your security group, ours is `rds-launch-wizard-1 (sg-0cab7e25f83e3e46b)`

- After that opens, click `Inbound` tab
- Click `Edit`
- Click `Add Rule`
- Click dropdown (current value is likely `Custom TCP Rule`), select `PostgreSQL` if that's the database type you set a few steps ago.
- Change `source` to match your security group, ours is `sg-0cab7e25f83e3e46b`



## 4. Update Django
```
(zappaCFE-LnWEDIxd) bash-3.2$ pipenv install psycopg2
```
Or you can try: 
```
(zappaCFE-LnWEDIxd) bash-3.2$ pipenv install psycopg2-binary
```
The `psycopg2-binary` solved a `gcc` issue during my recent tests.


Update your `settings.py` to:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'zapdjangodb',
        'USER': 'zapdjangouser',
        'PASSWORD': '15422715-547f-4eec-93b5-faa4f90e1d48',
        'HOST': 'zapdjangodb.cfak8zfgryqc.us-west-2.rds.amazonaws.com',
        'PORT': 5432,
    }
}

```


## 5. Zappa Django Utils
Unfortunately, RDS doesn't automatically create our database for us, so we have to do it. We'll use the package [`zappa-django-utils`](https://github.com/Miserlou/zappa-django-utils) to do this for us (as that's what it was created for) by the Zappa creator. 

```
(zappaCFE-LnWEDIxd) bash-3.2$ pipenv install zappa-django-utils
```

```python
# settings.py
INSTALLED_APPS += ['zappa_django_utils']
```


## 6. Update `zappa_settings.json`
We're going to add our `vpc_config` to our `zappa_settings.json`. That's pretty much all we need to do here. Each item has been referenced above (or in the [this post](https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python) or [this one](https://www.codingforentrepreneurs.com/blog/serverless-django-with-zappa-on-aws-lambda)).

```
{
    "dev": {
        "aws_region": "us-west-2",
        "django_settings": "zapdj.settings",
        "profile_name": "default",
        "project_name": "zappacfe",
        "runtime": "python3.6",
        "s3_bucket": "my-awesome-zappa-bucket",
        "timeout_seconds": 900,
        "manage_roles": false,
        "role_name": "ZappaDjangoRole",
        "role_arn": "arn:aws:iam::908424941159:role/ZappaDjangoRole",
        "vpc_config" : {
            "SubnetIds": ["subnet-449a6c1f", "subnet-55c3a61c", "subnet-779fdd10"],
            "SecurityGroupIds": [ "sg-0cab7e25f83e3e46b" ]
        }
    }
}
```


## 7. Update Deployment
```
(zappaCFE-LnWEDIxd) bash-3.2$ zappa update dev
```



## 8. Create DB
You only have to do this _1 time_. It's not the same as `migrate`. In case you don't know/realize, `migrate` does create a sqlite database but it *will not do* the same for RDS.

```
(zappaCFE-LnWEDIxd) bash-3.2$ zappa manage dev create_pg_db
```
**create_pg_db** command comes from [`zappa-django-utils`](https://github.com/Miserlou/zappa-django-utils). Yes you can create your own custom ones but we'll skip that for now.


Possible errors:
If things don't go smoothly, something above is probably off. Check everything!


`psycopg2.ProgrammingError: database "zapdjangodb" already exists`: You database is done. You don't need to create it again. Move to the next step!

`django.db.utils.OperationalError: FATAL:  password authentication failed ...` Reset your RDS Master password and ensure that your `settings.py` has the correct one for the database setting.

`psycopg2.OperationalError: could not connect to server: Connection timed out`: You'll see this if your Security Group and/or Inbound settings were done incorrectly.

`timeout`: The `zappa_settings.json` we have above has a 900 second timeout (aka 15 minutes) so there's a chance this will timeout if you didn't change it. 

- Did you have a different error? Comment below so we can help solve it.

- Did you solve any errors we didn't cover? Please share in comments below.


## 9. Migrations & Migrate
This is how you can migrate your live RDS database from here on out.

```
(zappaCFE-LnWEDIxd) bash-3.2$ python manage.py makemigrations
(zappaCFE-LnWEDIxd) bash-3.2$ zappa update dev
(zappaCFE-LnWEDIxd) bash-3.2$ zappa manage dev migrate
```
Do you see the `zappa manage` command? It's like running `python manage.py` but it doesn't work exactly like that. 


## 10. Create Super User

```
(zappaCFE-LnWEDIxd) bash-3.2$ zappa manage dev create_admin_user


[START] RequestId: bb93b8ca-e4a4-11e8-9f37-23d99ba78609 Version: $LATEST
[DEBUG] 2018-11-10T04:54:40.787Z bb93b8ca-e4a4-11e8-9f37-23d99ba78609 Zappa Event: {'manage': 'create_admin_user'}
Creating a new admin superuser...
Created user "admin", email: "admin@admin.com", password: 10FK9N87SN
Log in and change this password immediately!
[END] RequestId: bb93b8ca-e4a4-11e8-9f37-23d99ba78609
[REPORT] RequestId: bb93b8ca-e4a4-11e8-9f37-23d99ba78609
Duration: 485.60 ms
Billed Duration: 500 ms 
Memory Size: 512 MB
Max Memory Used: 49 MB
```

**create_admin_user** command comes from [`zappa-django-utils`](https://github.com/Miserlou/zappa-django-utils); this command allows you to create a superuser much like `python manage.py createsuperuser`.



##### Why doesn't `zappa manage dev createsuperuser` work?
Zappa is serverless so shell commands like `python manage.py createsuperuser` won't work (and thus `zappa manage dev createsuperuser` won't work.  You can, however, create your own custom commands like what you see on [the django docs](https://docs.djangoproject.com/en/2.1/howto/custom-management-commands/). At the time of this writing, that's pretty much all the [`zappa-django-utils`](https://github.com/Miserlou/zappa-django-utils) is.




#### Boom! Serverless Django with Zappa, Lambda, and a PostgreSQL Database with Amazon RDS. Stay tuned for me.