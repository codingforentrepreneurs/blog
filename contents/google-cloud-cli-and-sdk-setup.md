---
title: Google Cloud CLI &amp; SDK Setup
slug: google-cloud-cli-and-sdk-setup

publish_timestamp: April 21, 2020
url: https://www.codingforentrepreneurs.com/blog/google-cloud-cli-and-sdk-setup/

---

<iframe width="560" height="315" src="https://www.youtube.com/embed/k-8qFh8EfFA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Google Cloud has a number of awesome services that make creating your projects much more robust. Google pioneered and open-sourced Kubernetes so their cloud service is an obvious choice for deploying Kubernetes as well as many other services.

Before we can do anything with GCloud, I recommend you setup the GCloud SDK. Which is why I wrote this post:

#### 1. Login or Create a Google Account on [cloud.google.com](https://cloud.google.com)
> Yes, you can use your gmail account


#### 2. Download GCloud SDK Package [Here](https://cloud.google.com/sdk/docs/downloads-versioned-archives#installation_instructions)
Find your system version and download the best suited option. Most modern operating systems are 64-bit so download the correct 64 bit version of the gcloud SDK for your system. 


#### 3. Move GCloud SDK and Unpack

##### __macOS / Linux__
Open up Terminal
```
cd ~/Downloads
mv google-cloud-sdk-290.0.0-darwin-x86_64.tar.gz ~/
cd ~/
```
Replace `google-cloud-sdk-290.0.0-darwin-x86_64.tar.gz` with the version you downloaded

Unpacking the `tar` file:
```
tar xopf google-cloud-sdk-290.0.0-darwin-x86_64.tar.gz
rm google-cloud-sdk-290.0.0-darwin-x86_64.tar.gz
```
Install `gcloud` on your `PATH`
```
cd google-cloud-sdk
./install.sh
```
You should see:
```
Modify profile to update your $PATH and enable shell command 
completion?

Do you want to continue (Y/n)? 
```
Type `Y` and hit `return` to continue. You definitely want this since it makes running glcoud as easy as typing `gcloud` anywhere in terminal. Now you should see:
```
The Google Cloud SDK installer will now prompt you to update an rc 
file to bring the Google Cloud CLIs into your environment.

Enter a path to an rc file to update, or leave blank to use 
[/Users/cfe/.zshrc]
```
Hit `return` to accept the default (this is what I recommend).


##### __Windows__
Open up Powershell
```
cd ~/Downloads
mv google-cloud-sdk-290.0.0-windows-x86_64.zip ~/
cd ~/
```

Then you can use `Expand-Archive` to unpack the download. This might take a while.
```
Expand-Archive google-cloud-sdk-290.0.0-windows-x86_64.zip .
```
Once done, remove the archive:
```
rm google-cloud-sdk-290.0.0-windows-x86_64.zip
```
Now:
```
cd google-cloud-sdk
.\install.bat
```

You should see:
```
Welcome to the Google Cloud SDK!

To help improve the quality of this product, we collect anonymized usage data
and anonymized stacktraces when crashes are encountered; additional information
is available at <https://cloud.google.com/sdk/usage-statistics>. This data is
handled in accordance with our privacy policy
<https://policies.google.com/privacy>. You may choose to opt in this
collection now (by choosing 'Y' at the below prompt), or at any time in the
future by running the following command:

    gcloud config set disable_usage_reporting false

Do you want to help improve the Google Cloud SDK (y/N)?
```
Type `y` or `N` depending on your preference and press `Enter` to continue. Now you should see:
```
Update %PATH% to include Cloud SDK binaries? (Y/n)?  .
Please enter 'y' or 'n':
```
Type `y` and hit `Enter` to continue. You definitely want this since it makes running glcoud as easy as typing `gcloud` anywhere in powershell.


#### 4. Update glcoud
Open a new terminal / powershell window:
```
gcloud components update
```
> Windows users: you might have to right click on Powershell and `Run as Administrator` to use this command

#### 5. Login on the Command Line

```
gcloud auth login
```
This will open a web browser and have you login to Google and accept that `Google Cloud SDK wants to access your Google Account`.

#### 6. Create your first Google Cloud Project

Assuming you have a new account:

1. Go to [cloud.google.com](https://cloud.google.com).

2. On the Navbar click `Select a project`

3. Click `NEW PROJECT`

4. Add a `Project Name` - You can always add new projects. I'll be using `CFE Project`. Take note of the `Project ID`. Mine was auto generated as `cfe-project-2`. You'll use the `project-id` for your local gcloud setup.

5. (Optional) Select an `Organization` if you have one on your account.

6. Click `Create`

7. Open `terminal` / `powershell` and:

```
gcloud config set project cfe-project-2
```
Replace `cfe-project-2` with your `project-id` from step 4.

#### 7. All Setup Up!
Now your local system should be able to use the enabled services you have in Google Cloud. You'll have to use the Google Cloud [console](https://console.cloud.google.com] from time to time to ensure services can/are running.