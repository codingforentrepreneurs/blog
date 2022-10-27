---
title: AWS S3, IAM Group &amp; Policy Setup
slug: aws-s3-iam

publish_timestamp: April 17, 2020
url: https://www.codingforentrepreneurs.com/blog/aws-s3-iam/

---

This post is related to an upcoming series that covers making Serverless Apps using Python, AWS Lambda, and API Gateway. 

This S3 & IAM Policy works for nearly *any* project that uses IAM Groups, Users, or Roles that require access to a specific bucket (or buckets). 

Enjoy.

__Related Posts__:
- [Setup AWS CLI for Local Development](https://www.codingforentrepreneurs.com/blog/aws-cli-setup)



#### 1. Create an S3 Bucket
1. Navigate to [S3](https://console.aws.amazon.com/s3/)
2. Select `Create bucket`
3. Name the bucket something easy to remember for your lambdas. I named mine `cfe-lambdas`. Bucket names are unique across AWS. 
4. Under `region` pick a region that's close to you as it will be faster to upload files. We'll use this same region for our `lambda` functions later. In my case, I am still using `us-west-1.
5. Allow all the default values. This bucket is mostly internal (aka just for moving our files around AWS)

#### 2. Add new policy to our IAM group (or IAM user)
1. Navigate to [IAM](https://console.aws.amazon.com/iam)
2. Select `Policies` in the sidebar.
3. Select `Create Policy`
4. Select `JSON`. Enter what you see below changing `cfe-lambdas` to your bucket name.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:*Object*",
                "s3:ListBucket",
                "s3:GetObjectAcl",
                "s3:PutObjectAcl",
                "s3:PutObjectVersionAcl",
                "s3:GetBucketLocation",
                "s3:ListBucketMultipartUploads",
                "s3:ListBucketVersions"
            ],
            "Resource": [
                "arn:aws:s3:::cfe-lambdas"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:*Object*",
                "s3:PutObject",
                "s3:GetObjectAcl",
                "s3:PutObjectAcl",
                "s3:PutObjectVersionAcl",
                "s3:PutObjectTagging",
                "s3:ListMultipartUploadParts",
                "s3:AbortMultipartUpload"
            ],
            "Resource": [
                "arn:aws:s3:::cfe-lambdas/*"
            ]
        }
    ]
}
```
This policy will enable our group to perform actions on this particular bucket (and no other ones). You can always add more buckets if you need to this policy or create a new one too.

5. Select `Review policy` and give the policy a name. I used `CFE_LAMBDAS_S3_FULL_ACCESS_POLICY`.
6. Select `Create policy`
7. Select `Groups` in the Sidebar
8. Navigate to the group you created in [this post](https://www.codingforentrepreneurs.com/blog/aws-cli-setup). Mine was `LocalDevUsers`
9. Under `Permissions` > `Managed Policies` select `Attach Policy`
10. Search & select the policy you just created. Mine is `CFE_LAMBDAS_S3_FULL_ACCESS_POLICY`
11. Select `Attach policy`

You should now be able to upload directly to your s3 bucket.

The [aws cli](https://aws.amazon.com/cli/) command we'll end up using is:
```
aws s3api put-object --bucket cfe-lambdas --key helloWorld/helloWorldLambda.zip --body helloWorldLambda.zip
```