---
title: Setup AWS CLI for Local Development
slug: aws-cli-setup

publish_timestamp: April 16, 2020
url: https://www.codingforentrepreneurs.com/blog/aws-cli-setup/

---

This post is related to an upcoming series that covers making Serverless Apps using Python, AWS Lambda, and API Gateway. That's how we choose the policies we did but the overall setup process is the same regardless of the service (and policy) you decide to use.

__Related Posts__:
- [AWS S3, IAM Group & Policy Setup](https://www.codingforentrepreneurs.com/blog/aws-s3-iam)

## Setup your local system with the AWS CLI
#### 1. Create [AWS Account](https://aws.amazon.com)

#### 2. Download & Install AWS CLI [here](https://aws.amazon.com/cli/)

#### 3. Configure local IAM user credentials
1. Login and Navigate to [IAM](https://console.aws.amazon.com/iam)
2. Select `Groups` in the Sidebar
3. Select `Create New Group`
We create a group to add policies instead of directly to the user. That way we can remove users from groups instead of trying to remember which users have which policies.
4. Name Group `LocalDevUsers`, Click `Next Step`
5. Under Policies, select the ones that are best suited for this group's use cases; I only selected `AWSLambdaFullAccess` at this time since it's easy to add more later. Now select `Next Step`
6. Select `Create Group`
7. Select `Users` in the Sidebar
8. Select `Add user`
9. Add your `User name`
10. Check `Programmatic access`. This is required for using the AWS CLI
11. Select `Next: Permissions`
12. Under `Add user to Group`, find your recent group `LocalDevUsers` set above.
13. Select `Next: Tags`. Add tags if you want.
14. Select `Next: Review`. Review information is correct.
15. Select `Create user`
16. Make note of the `Access key id` and `Secret access key` or select `Download .csv`.
17. Hit `Close`
18. Select `Groups` in the Sidebar
19. Find and select your group. Mine is `LocalDevUsers`
20. Under the `Users` tab (not the sidebar), verify your recently created user is listed. If not, select `Add Users to this Group`, find and select your user and click `Add User`.

#### 4. Configure AWS CLI with your user
Open Terminal / PowerShell and do the following:

```
aws configure
```
You should see:
```
AWS Access Key ID [None]:
```
Enter your `Access key id` from above. Press `enter`/`return`. Now you'll see:
```
AWS Secret Access Key [None]:
```
Enter your `Secret access key` from above. Press `enter`/`return`. Now you'll see:
```
Default region name [None]: us-west-1
```
I'm going to use `us-west-1` but you should pick a region where you're closest to. Regions are listed [here](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html)


```
Default output format [None]: json
```


#### 5. AWS CLI Commands Reference

The AWS CLI reference is [here](https://docs.aws.amazon.com/cli/latest/reference/). Whenever you need a new service to be accessible by our user above. Just add a new policy that gives them the permissions they need. In our next post, we'll cover how to create an S3 Bucket with the necessary permissions for a IAM User Group (or individual user).


Now your AWS CLI should be set up. Enjoy.