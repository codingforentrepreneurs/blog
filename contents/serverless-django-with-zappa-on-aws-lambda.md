---
title: Serverless Django with Zappa on AWS Lambda
slug: serverless-django-with-zappa-on-aws-lambda

publish_timestamp: Nov. 9, 2018
url: https://www.codingforentrepreneurs.com/blog/serverless-django-with-zappa-on-aws-lambda/

---


Going serverless with Django and Zappa is *mostly* painless. It can be tedious but implementing Zappa with Django could have some serious impact on your bottom line and user experience. I'll cover Zappa / Serverless benefits another time but for now, let's just make it happen.

## First things first, Setup AWS
- Complete the **entire** post called [Prepare AWS IAM User, Role, and Policies for Zappa and Serverless Python](https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python)


## 1. Setup Virtualenv with Requirements
Missing [pipenv](https://www.codingforentrepreneurs.com/blog/pipenv-virtual-environments-for-python)?
```
$ cd /path/to/your/dev/folder/
$ pipenv install --python 3.6 zappa django django-rest-framework awscli
```

or

```
$ cd /path/to/your/dev/folder/
$ python3 -m pipenv install --python 3.6 zappa django django-rest-framework awscli
```


> If pipenv's cli fails for anything other than `pip.basecommand` (see below), try: `python3 -m pipenv install --three zappa django django-rest-framework`


Example:
```
$ mkdir /dev
$ cd /dev
$ mkdir zapdj
$ cd zapdj
$ pipenv install --python 3.6 zappa django django-rest-framework
```

###### Error Handling for pipenv / pip:
From the example above, if you see `ModuleNotFoundError: No module named 'pip.basecommand'`, then do the following:
```
$ cd /dev/zapdj
$ pipenv shell
(zappaCFE-LnWEDIxd) bash-3.2$ pip install pip==10.0.1
(zappaCFE-LnWEDIxd) bash-3.2$ exit
$ pipenv install zappa django django-rest-framework
```

It seems that pip version 18.1, isn't playing well with my system. Perahps 18.2 will solve the issue.




## 2. Activate virtual environment
```
$ pipenv shell
```
If you installed `pip==10.0.1`, ignore the warning about upgrading pip (for now at least).


## 3. Start Django project
```
(zapdj-RaNDom123) $ django-admin startproject zapdj .
```

## 4. Add AWS User Credentials to the Command Line (using AWS CLI)
On [this blog post](https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python) we setup our IAM user for our Zappa project. This **user is going to be Zappa specific** (*not for Django Static/Media files*), just Zappa.

It will look something like below. Keep in mind that `<YOUR_AWS_ACCESS_KEY>` `<YOUR_AWS_SECRET_KEY>` will be your actual keys. Also take note of your default region from this guide as well as the post above. Ensure they are the same.

```
$ cd /dev/zapdj
$ pipenv shell
(zappaCFE-LnWEDIxd) bash-3.2$ aws configure
AWS Access Key ID [****************GVEA]: <YOUR_AWS_ACCESS_KEY>
AWS Secret Access Key [****************VxPd]: <YOUR_AWS_SECRET_KEY>
Default region name [us-west-2]: 
Default output format [json]: 
```

Did you mess up? Not to worry, just run `aws configure` again. As far as I know, you can rerun this anytime with little to no effect on Zappa. The only exception I would image is the `region name` you choose.


## 5. Start Zappa
Below we'll use `zappa init` to create our zappa's settings file (`zappa_settings.json`), we'll have to modify these settings based on our actions taken in [this guide](https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python), but this nice command `zappa init` will get us started.


```
(zappaCFE-LnWEDIxd) bash-3.2$ zappa init

███████╗ █████╗ ██████╗ ██████╗  █████╗
╚══███╔╝██╔══██╗██╔══██╗██╔══██╗██╔══██╗
  ███╔╝ ███████║██████╔╝██████╔╝███████║
 ███╔╝  ██╔══██║██╔═══╝ ██╔═══╝ ██╔══██║
███████╗██║  ██║██║     ██║     ██║  ██║
╚══════╝╚═╝  ╚═╝╚═╝     ╚═╝     ╚═╝  ╚═╝

Welcome to Zappa!

Zappa is a system for running server-less Python web applications on AWS Lambda and AWS API Gateway.
This `init` command will help you create and configure your new Zappa deployment.
Let's get started!

Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
```

##### Select an environment name, since we're testing things out, use `dev`. You can easily add new stages later.
```
What do you want to call this environment (default 'dev'): dev
```

##### Select your AWS Profile. `aws configure` will set your default profile, so we'll use that below.
```
AWS Lambda and API Gateway are only available in certain regions. Let's check to make sure you have a profile set up in one that will work.
We found the following profiles: eb-cli, cfe, and default. Which would you like us to use? (default 'default'): default
```

> AWS user settings are stored in ` ~/.aws/credentials` and `~/.aws/config` if you need to add other profiles. That is covered on [this offical AWS guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html)


##### S3 bucket selection. 
What you put here doesn't actually matter because, assuming you followed [this guide](https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python), we already created one. 
```
Your Zappa deployments will need to be uploaded to a private S3 bucket.
If you don't have a bucket yet, we'll create one for you too.
What do you want to call your bucket? (default 'zappa-pznkrhk0z'): 
```


##### Application settings
By default, you'll notice Zappa recognizes Django! That's awesome. We can keep the default here by pressing enter. Not to worry, we'll change a few settings shortly.
```
It looks like this is a Django application!
What is the module path to your projects's Django settings?
We discovered: zapdj.settings
Where are your project's settings? (default 'zapdj.settings'): 
```

##### Region Availability
We're learning. Use the default of 'n'
```
You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]: n
```


##### Confirm settings

```
Okay, here's your zappa_settings.json:

{
    "dev": {
        "aws_region": "us-west-2",
        "django_settings": "zapdj.settings",
        "profile_name": "default",
        "project_name": "zappacfe",
        "runtime": "python3.6",
        "s3_bucket": "zappa-pznkrhk0z"
    }
}

Does this look okay? (default 'y') [y/n]: y
```

Now we have our auto settings generated, we have to make some modifications.



## 6. Update `zappa_settings.json` to our specific project

Open up your `zappa_settings.json`, and you'll see:

```json
{
    "dev": {
        "aws_region": "us-west-2",
        "django_settings": "zapdj.settings",
        "profile_name": "default",
        "project_name": "zappacfe",
        "runtime": "python3.6",
        "s3_bucket": "zappa-pznkrhk0z"
    }
}
```

We want to update it to look more like:

```json
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
        "role_arn": "arn:aws:iam::908424941159:role/ZappaDjangoRole"
    }
}
```


`role_name` and `role_arn`: We set these in [this post](https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python), if you haven't used roles before, it should be easy to find your roles on [IAM Roles](https://console.aws.amazon.com/iam/home#/roles)

`s3_bucket`: We also created this in [this post](https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python), so grab that bucket name. 

`aws_region`: Double check this is accurate to the region you selected. **AWS Lambda** is not available in all regions.



## 7. Setup complete? Let's deploy!

```
(zappaCFE-LnWEDIxd) bash-3.2$ zappa deploy dev
```
`dev` above is related to the nave of the stage. You can have multiple stages which is why you have to write out the stage name you gave during the `zappa init` process. If you forget that name, check `zappa_settings.json` for the key name that holds the actual settings.


## 8. Update Allowed Hosts

After you run `zappa deploy dev` you should see something like:
```
(zappaCFE-LnWEDIxd) bash-3.2$ zappa update dev
Calling update for stage dev..
Downloading and installing dependencies..
 - sqlite==python36: Using precompiled lambda package
Packaging project as zip.
Uploading zappacfe-dev-1541810147.zip (17.7MiB)..
100%|█████████████████████████████████████████████████████| 18.5M/18.5M [00:09<00:00, 2.56MB/s]
Updating Lambda function code..
Updating Lambda function configuration..
Uploading zappacfe-dev-template-1541810174.json (1.5KiB)..
100%|█████████████████████████████████████████████████████| 1.56K/1.56K [00:00<00:00, 13.5KB/s]
Deploying API Gateway..
Scheduling..
ERROR:Unexpected client error An error occurred (AccessDeniedException) when calling the GetPolicy operation: User: arn:aws:iam::908424941159:user/ZappaUser is not authorized to perform: lambda:GetPolicy on resource: arn:aws:lambda:us-west-2:908424941159:function:zappacfe-dev
Unscheduled zappacfe-dev-zappa-keep-warm-handler.keep_warm_callback.
Scheduled zappacfe-dev-zappa-keep-warm-handler.keep_warm_callback with expression rate(4 minutes)!
Your updated Zappa deployment is live!: https://mdqre8rfwl.execute-api.us-west-2.amazonaws.com/dev
```

The very last line includes the url to your live django project: `https://mdqre8rfwl.execute-api.us-west-2.amazonaws.com/dev` Yours will be different.

We need to add this host to our `ALLOWED_HOSTS` setting in Django (aka in `settings.py`)

So it will look like
```python
# zapdj.settings.py
ALLOWED_HOSTS = ['mdqre8rfwl.execute-api.us-west-2.amazonaws.com']
```
Save the `settings.py` file and update zappa with:
```
(zappaCFE-LnWEDIxd) bash-3.2$ zappa update dev
```

Now from here on out, you just need to run `zappa update <your-sage>` when you need to update your live lambda project.


## 9. Next step: Using an [RDS Database for Serverless Django]( https://www.codingforentrepreneurs.com/blog/rds-database-serverless-django-zappa-aws-lambda) ([View Post](https://www.codingforentrepreneurs.com/blog/rds-database-serverless-django-zappa-aws-lambda))
As of now, our Django project works and is deployed. It's just missing a live database. Setting up a live database in a serverless environment is not trivial. Soon we'll have a post covering how to setup AWS RDS with Zappa to ensure you can have a functional database for all your users.

See you soon!



## Useful Zappa Commands

##### Updating Code / Environment
```
$ zappa update dev
```

##### Un-deploy Project
```
$ zappa undeploy dev
```

##### Log-like tail
```
$ zappa tail dev
```

Replace `dev` above with your zappa stage (like to `production`)



## Common Deploy/Update/Undeploy Errors

```
botocore.exceptions.ParamValidationError: Parameter validation failed:
Invalid type for parameter restApiId, value: None, type: <class 'NoneType'>, valid types: <class 'str'>
```
This error could be related to bad json data (ie in `zappa_settings.json`). Check that file for valid json.

If that's not it, try:
```
$ zappa undeploy dev
$ zappa deploy dev
```


I recommend that you use **`Python 3.6`** and up. You might be able to use older versions but _why_?


> Did you find any other errors? How about solutions to some too? Please post in the comments below. We'll update this guide if needed based on your comments.
