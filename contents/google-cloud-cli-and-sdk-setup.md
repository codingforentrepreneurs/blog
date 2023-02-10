---
title: Google Cloud CLI &amp; SDK Setup
slug: google-cloud-cli-and-sdk-setup

publish_timestamp: April 21, 2020
url: https://www.codingforentrepreneurs.com/blog/google-cloud-cli-and-sdk-setup/

---


Google Cloud has a number of awesome services that make creating your projects much more robust. Google pioneered and open-sourced Kubernetes so their cloud service is an obvious choice for deploying Kubernetes as well as many other services.

Before we can do anything with GCloud, I recommend you setup the GCloud SDK. Which is why I wrote this post:

## Login to [cloud.google.com](https://cloud.google.com)
If you have a `gmail` account, you do not need to sign up, just login with that gmail account.


## Download Google Cloud CLI Package (`gcloud`)
Regardless of the system you're own, you'll need to download the Google Cloud CLI. You can follow the official insructions from google [here](https://cloud.google.com/sdk/docs/downloads-versioned-archives#installation_instructions).

Find your system version and download the best suited option. Most modern operating systems are 64-bit so download the correct 64 bit version of the gcloud SDK for your system. 

For my computers, I downloaded:
- On my Apple Silicon MacBook Pro: `google-cloud-cli-417.0.1-darwin-arm.tar.gz`
- On my Apple Intel MacBook Pro: `google-cloud-cli-417.0.1-darwin-x86_64.tar.gz`
- On my 64-bit Windows 10: `google-cloud-cli-417.0.1-windows-x86_64.zip` (I do _not_ recommended downloading it with Python bundled. Download python from python.org instead.)

The version numbers you download here will likely be different then mine -- which is fine. We will be updating the Google Cloud CLI in a few steps anyway.


### macOS/Linux CLI Installation
macOS and Linux users have a few more steps to run to install the `Google Cloud CLI`. It's possible that homebrew has a solution to install `gcloud` but I'm going to install directly from the source.


__Open up `Terminal` or Command Line__
```bash
cd ~/Downloads
mv google-cloud-cli-417.0.1-darwin-arm.tar.gz ~/
cd ~/
```
Replace `google-cloud-cli-417.0.1-darwin-arm.tar.gz` with the version you downloaded

__Unpack the `tar` file__
```bash
tar xopf google-cloud-cli-417.0.1-darwin-arm.tar.gz
rm google-cloud-cli-417.0.1-darwin-arm.tar.gz
```

__Install `gcloud` on your `PATH`__
```bash
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


### Windows Installation

Open up PowerShell's command line.

```powershell
cd ~\Downloads
mv google-cloud-cli-417.0.1-windows-x86_64.zip ~\
cd ~\
```

Then you can use `Expand-Archive` to unpack the download. This might take a while.
```powershell
Expand-Archive google-cloud-cli-417.0.1-windows-x86_64.zip .
```
Once done, remove the archive:
```powershell
rm google-cloud-cli-417.0.1-windows-x86_64.zip
```
Now:
```powershell
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


## Update `glcoud`
On your command line (`Terminal` or `PowerShell`), you should be able to run `gcloud` now:

```bash
gcloud --version
```

Let's update `cloud`:
```
gcloud components update
```
_Windows users_ you might have to right click on Powershell and `Run as Administrator` to use this command for the first time.

## Login to Google Cloud for gcloud

The following command will open a web browser and have you login to Google and accept that `Google Cloud SDK wants to access your Google Account`.

```
gcloud auth login
```


## Create your first Google Cloud Project
If you have never opend Google Cloud before, you will need to create a new project. Here's how you can do that:

1. Go to [cloud.google.com](https://cloud.google.com).

2. On the Navbar click `Select a project`

3. Click `NEW PROJECT`

4. Add a __Project Name__ - You can always add new projects. I'll be using `Serverless CFE`. Take note of the __Project ID__ (`project-id`), mine was auto generated as `serverless-cfe`. You'll use the `project-id` for your local gcloud setup.

5. (Optional) Select an `Organization` if you have one on your account.

6. Click `Create`

7. Open `terminal` / `powershell` and:

```
gcloud config set project serverless-cfe
```
Replace `serverless-cfe` with your `project-id` from step 4.

## All Setup Up!
Now your local system should be able to use the enabled services you have in Google Cloud. You'll have to use the Google Cloud [console](https://console.cloud.google.com]) from time to time to ensure services can/are running.
