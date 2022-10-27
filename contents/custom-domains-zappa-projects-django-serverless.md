---
title: Custom Domains for Zappa Projects
slug: custom-domains-zappa-projects-django-serverless

publish_timestamp: Nov. 11, 2018
url: https://www.codingforentrepreneurs.com/blog/custom-domains-zappa-projects-django-serverless/

---

For web applications, a custom domain is crucial. Doing so with AWS is easy enough. This guide will help you set up your Zappa project to have a custom domain.

First things first, do you own a domain? If so, you'll follow this guide exactly.

The general goal is to 

1. Setup your domain in AWS [Route53](https://console.aws.amazon.com/route53/)
2. Get a https certificate on AWS [ACM](https://console.aws.amazon.com/acm/home) using `us-east-1` regardless of any other settings
3. Add your newly ACM certified domain to your API Gateway
4. Update your Zappa settings to be configured for above.


###### Setup Suggestions
Below are related articles to this post. Overall, it should function without them but it's nice to know they exist. Good luck.
- [Prepare AWS IAM User, Role, and Policies for Zappa and Serverless Python](https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python)
- [Serverless Django with Zappa on AWS Lambda](https://www.codingforentrepreneurs.com/blog/serverless-django-with-zappa-on-aws-lambda)
- [RDS Database for Serverless Django + Zappa on AWS Lambda](https://www.codingforentrepreneurs.com/blog/rds-database-serverless-django-zappa-aws-lambda)



## 1. Domain Name Required.
Associating a custom domain name (with https nonetheless) for your Zappa project is pretty straight forward but first you're going to want to make sure you have one.

- Buying a new domain name? The easiest way is to use [Route53](https://console.aws.amazon.com/route53/)
- Have a domain name? Just follow the setup process below


## 2. Setup your Domain in Route53
- Go to [Route53](https://console.aws.amazon.com/route53/)
- Click "Hosted Zones"
- Click "Create hosted zone"
- Add in your domain name `joincfe.com` (not `www.joincfe.com` or `somethingelse.joincfe.com`)
- Click "Create"

After it's created, you'll see 4 records for the `NS` type. This means `nameserver`

You'll use these 4 records to update your nameservers for your domain. If you bought your domain on [name.com](https://www.name.com/referral/5a470):
- Login to your account on [name.com](https://www.name.com/referral/5a470)
- Click the domain you'll be using (it **must** match what you put in Route53)
- Click on 'Nameservers'
- Delete all pre-existing nameservers. (Or keep track of your old ones if you need to revert for some reason)

Allow for a little time to pass to ensure Route53 has appropriate control over your domain.  

## 3. AWS Certificate Manager (ACM)
This allows you to have *free https certificates* from AWS (assuming you use a public certificate)

- Go to [ACM](https://console.aws.amazon.com/acm)
- In the top right ensure it says N. Virginia for `us-east-1` as your region
- Click 'Request a certificate'
- Ensure `Request a public certificate` is checked.
- Click 'Request a certificate'
- Add in your domain name including subdomains. I recommend also including the wildcard so it's easier to make subdomain changes if you need. You could also just use `www`. What I did:
```
joincfe.com
www.joincfe.com
*.joincfe.com
```
- Click "next"

- For validation method, choose the option for using Route53 automatically. This makes things more simple.

- Finish issuing setup
- Make note of your `ARN` for you certificate, something like `arn:aws:acm:us-east-1:531312223:certificate/asdf3c1a-4aab-8d6e-asdfads331`


## 4. Setup API Gateway to Custom Domain
This is the key part to ensure you're custom domain is working for Zappa.

- Go to [API Gateway](https://console.aws.amazon.com/apigateway/home)
- In the top right ensure it says the correct region you used in your Zappa project. Mine is `Oregon` and `us-west-2`; I had to switch over from the ACM step.
- You may have to click `Get started` to see the sidebar. 
- Look for your zappa project under `API`, mine is named `zappacfe-dev`, just ensure it's there otherwise you might be in the wrong region
- Click `Custom Domain Names`
- Add each domain (including subdomain) you want to use. Keep in mind you'll have to update this in the future if you add subdomains.
- First one, I did the following:
```
Domain Name: www.joincfe.com
Endpoint Configuration: Edge Optimized
ACM Certificate (us-east-1): joincfe.com (someID131) 
```
Notice the last part of `ACM Certificate`, that is the one we just created. If you don't see it, there was an error in the previous step.

## 5. Setup Route 53 for your new `Target Domain Name`s from API Gateway
- Navigate to [API Gateway](http://console.aws.amazon.com/apigateway)
- Click `Custom Domain Names`
- Locate `Target Domain Name` for each subdomain you have (even if it's just one).
Example:

```
www.joincfe.com

Target Domain Name
adfddplocbkjhd.cloudfront.net

Hosted Zone ID
Md13TNDATAQYW2
```
_Each subdomain_ will have a `Target Domain Name`, copy this value.

- Navigate to  [Route53](https://console.aws.amazon.com/route53/)
- Select your domain name
- Click "Create Record Set"
- Add the following:

```
**Name**: www.joincfe.com

**Type**: A - IPv4 Address

**Alias** Yes 

**Alias Target**  Paste in the *target domain name* from above, mine was `adfddplocbkjhd.cloudfront.net`
```
- Click "Create"

Repeat this process for each subdomain you have. (such as `joincfe.com`, `blog.joincfe.com`, `course.joincfe.com`, etc)


##  6.  Update `zappa_settings.json`
Add the `domain` and `certificate_arn` key/value pairs like I have below. These are all coming from [this post], [this post], and [this post](https://www.codingforentrepreneurs.com/blog/rds-database-serverless-django-zappa-aws-lambda).
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
        },
        "domain": "joincfe.com",
        "certificate_arn": "arn:aws:acm:us-east-1:531312223:certificate/asdf3c1a-4aab-8d6e-asdfads331"
    }
}
```


## 7. Update Django and Zappa
Add your domain to `ALLOWED_HOSTS` in Django settings. Such as:
```
ALLOWED_HOSTS = ['24phrejind.execute-api.us-west-2.amazonaws.com', '.joincfe.com']
```
Notice I used a `.` before `joincfe.com` so the the domain is treated as a wildcard according to django (basically use `.joincfe.com` instead of `*.joincfe.com`)

And finally, update Zappa

```
(zappaCFE-LnWEDIxd) bash-3.2$ zappa update dev
```
## 8. Update Base Mappings
- Navigate to [API Gateway](http://console.aws.amazon.com/apigateway)
- Click `Custom Domain Names`
- Find your newly added domain name(s)
- Click "Edit"
- Click "Add mapping"
- Click "Destination"'s dropdown
- Click your API project (mine is `zappacfe-dev`
- Add the "dev" stage, click "save"

Repeat for any other custom domain names for this project.

## 9. Go to your newly created custom domain and enjoy!


##### Did you find any errors? Any solutions? Please comment below.