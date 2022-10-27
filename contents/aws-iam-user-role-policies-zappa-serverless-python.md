---
title: Prepare AWS IAM User, Role, and Policies for Zappa and Serverless Python
slug: aws-iam-user-role-policies-zappa-serverless-python

publish_timestamp: Nov. 9, 2018
url: https://www.codingforentrepreneurs.com/blog/aws-iam-user-role-policies-zappa-serverless-python/

---


This is [Zappa](https://www.zappa.io/) specific guide. Zappa is amazing for creating a serverless architecture. We'll be exploring *much* more about Zappa in the future so be sure to check back soon.

Zappa in AWS needs all of these permissions to function correctly. If you know AWS well and how to limit your resources, you can do so but what you see below is the bare minimum that zappa needs to create.

If all fails, open a new AWS account (seriously), give the IAM User Full Programmic Admin previliges, use `awscli` to login that user and create/deploy a Zappa project. It should work without issues. If these permissions are given, Zappa will automotically create some of the items you see below. The reason this guide exists is to **limit** what the AWS IAM User (or User Group) can do as a security measure.

Oh and I promise you, you'll hit snags along the way. Please comment below so this guide can help others.

### Requirements
- Create an AWS account, if you don't have one

### 1. Create AWS Policy for AWS Role
1. Go to [IAM](https://console.aws.amazon.com/iam)
1. Click "Policies"
1. Click "Create policy"
1. Click "JSON", add in:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "logs:*"
                ],
                "Resource": "arn:aws:logs:*:*:*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "lambda:GetFunctionConfiguration",
                    "lambda:UpdateFunctionConfiguration",
                    "lambda:InvokeFunction"
                ],
                "Resource": [
                    "*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "xray:PutTraceSegments",
                    "xray:PutTelemetryRecords"
                ],
                "Resource": [
                    "*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "ec2:AttachNetworkInterface",
                    "ec2:CreateNetworkInterface",
                    "ec2:DeleteNetworkInterface",
                    "ec2:DescribeInstances",
                    "ec2:DescribeSecurityGroups",
                    "ec2:DescribeNetworkInterfaces",
                    "ec2:DetachNetworkInterface",
                    "ec2:ModifyNetworkInterfaceAttribute",
                    "ec2:ResetNetworkInterfaceAttribute"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:*"
                ],
                "Resource": "arn:aws:s3:::*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "kinesis:*"
                ],
                "Resource": "arn:aws:kinesis:*:*:*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "sns:*"
                ],
                "Resource": "arn:aws:sns:*:*:*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "sqs:*"
                ],
                "Resource": "arn:aws:sqs:*:*:*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "dynamodb:*"
                ],
                "Resource": "arn:aws:dynamodb:*:*:*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "route53:*"
                ],
                "Resource": "*"
            }
        ]
    }
    ```

1. Name the policy something like: `ZappaRolePolicy`

1. Select `Create Policy`


### 2. Create AWS IAM ROLE
1. Go to [IAM](https://console.aws.amazon.com/iam)
1. Click "Roles"
1. Click "Create Role"
1. For "Select Type of trusted entity" select "AWS service"
1. For "Choose the service that will use this role" select "Lambda"
1. Click "Next: Permissions"
1. Find the newly created policy from above `ZappaRolePolicy` and select it
1. Click `Next: Review`
1. Give a role name such as  `ZappaDjangoRole`
1. Add/change `Role description` as needed.
1. Click `Create role`
1. Click on your newly created role (such as `ZappaDjangoRole`) in [IAM roles](https://console.aws.amazon.com/iam/home?#/roles)
1. Click `Trust Relationships`
1. Click `Edit trust relationship` and add the following:
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "",
          "Effect": "Allow",
          "Principal": {
            "Service": [
              "lambda.amazonaws.com",
              "apigateway.amazonaws.com",
              "events.amazonaws.com"
            ]
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    ```
1. Click Update Trust Policy

1. Copy the `Role ARN` such as `arn:aws:iam::908424941159:role/ZappaDjangoRole`



### 3. Group-Level Permissions Policy for Lambda Role and Zappa Related Events

1. Go to [IAM](https://console.aws.amazon.com/iam)
1. Click "Policies"
1. Click "Create policy"
1. Click "JSON", add in:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "iam:GetRole",
                    "iam:PutRolePolicy"
                ],
                "Resource": [
                    "*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iam:PassRole"
                ],
                "Resource": [
                    "arn:aws:iam::[ROLE_ID]:role/[ROLENAME]"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "apigateway:DELETE",
                    "apigateway:GET",
                    "apigateway:PATCH",
                    "apigateway:POST",
                    "apigateway:PUT",
                    "cloudformation:CreateStack",
                    "cloudformation:DeleteStack",
                    "cloudformation:DescribeStackResource",
                    "cloudformation:DescribeStacks",
                    "cloudformation:ListStackResources",
                    "cloudformation:UpdateStack",
                    "events:DeleteRule",
                    "events:DescribeRule",
                    "events:ListRules",
                    "events:ListRuleNamesByTarget",
                    "events:ListTargetsByRule",
                    "events:PutRule",
                    "events:PutTargets",
                     "events:RemoveTargets",
                    "lambda:*",
                    "lambda:AddPermission",
                    "lambda:CreateFunction",
                    "lambda:DeleteFunction",
                    "lambda:GetFunction",
                    "lambda:GetFunctionConfiguration",
                    "lambda:ListVersionsByFunction",
                    "lambda:UpdateFunctionCode",
                    "logs:DescribeLogStreams",
                    "logs:FilterLogEvents",
                    "route53:ListHostedZones",
                    "route53:ChangeResourceRecordSets",
                    "route53:GetHostedZone"
                ],
                "Resource": [
                    "*"
                ]
            }
        ]
    }
    ```
1. Replace `arn:aws:iam::[ROLE_ID]:role/[ROLENAME]` with your above `Role ARN` 
1. Add the name, such as `ZappaUserGeneralPolicy`
1. Click `Create policy`

> If unsuccessful, check your json above.


### 4. Create an S3 Bucket
This is where zappa will upload your code deployments (aka the zipped folder of your code) for Lambda to use.

1. Go to [S3](https://console.aws.amazon.com/s3/)
1. Click `+ Create Bucket`
1. Add `Bucket name` mine was `my-awesome-zappa-bucket` (yours will have to be different due to aws thankfully making this unique enforced)
1. Choose region, I used `us-west-2` (_US West (oregon)_)
> Be sure to use the region you want to use. I've tested `us-west-2` (_US West (oregon)_) and `us-east-1`(_US East (N. Virginia)_) and both work well. Make note of the url-encoded name (like `us-west-2`. Often, you'll find this in the `region=` in your url. The region, at this point, is important to note. You can use the table on [this page](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html) as reference if you can't find the Region's name.

1. Keep all defaults (so just click next until it's created.)
1. Be sure to remember your bucket name for the next step. 




### 5. Group-Level Permissions Policy for S3 for Zappa
1. Go to [IAM](https://console.aws.amazon.com/iam)
1. Click "Policies"
1. Click `Create policy`
1. Click "Create policy"
1. Click "JSON", add in:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:ListBucket"
                ],
                "Resource": [
                    "arn:aws:s3:::[YOUR_BUCKET_NAME]"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:DeleteObject",
                    "s3:GetObject",
                    "s3:PutObject",
                    "s3:CreateMultipartUpload",
                    "s3:AbortMultipartUpload",
                    "s3:ListMultipartUploadParts",
                    "s3:ListBucketMultipartUploads"
                ],
                "Resource": [
                    "arn:aws:s3:::[YOUR_BUCKET_NAME]/*"
                ]
            }
        ]
    }
    ```
1. Replace `[YOUR_BUCKET_NAME]` with the name you choose for your bucket in the previous step (mine was `my-awesome-zappa-bucket`).
1. Add your policy name, such as `ZappaUserS3Policy`



### 6. Create AWS IAM Group
1. Go to [IAM](https://console.aws.amazon.com/iam)
1. Click "Groups"
1. Click "Create New Group"
1. Set a name for your group, I used `ZappaGroup`
1. Find your policy `ZappaUserS3Policy` and `ZappaUserGeneralPolicy`

> **do not** add the `ZappaRolePolicy`


> Creating a group is optional. You can just add these polices to the user only (below) if you prefer. I tend to want to work with groups that way I can add/remove users if needed. This is standard practice because then I can worry more about managing Groups and their policies than individual users. This becomes espeically important if your team is going to grow (or shrink).



### 7. Create AWS IAM User. 
1. Go to [IAM](https://console.aws.amazon.com/iam)
1. Click "Users"
1. Click "Add user"
1. Add no policies to the user directly, the group has them.
1. Set a user name, I used `ZappaUser`
1. Check access type: `Programmatic access`
1. Click "Next: Permissions"
1. Add user to the `ZappaGroup` created above.
1. Click "Next: Review"
1. Click "Create user"
1. Make note of your `Access key id` and `Secret access key` and ensure it's safely kept. We'll be using it in the next guide.


### 8. Grab IAM User ARN
1. Click on 'Users'
1. Click `ZappaUser` (or your previously created user)
1. Grab the `User ARN`, something like `arn:aws:iam::123342844315:user/ZappaUser`, we'll be using it in the next guide.



### 9. Proceed to the [Django with Zappa on AWS Serverless](https://www.codingforentrepreneurs.com/blog/serverless-django-with-zappa-on-aws-lambda) guide.
