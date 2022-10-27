---
title: Version Control Basics &amp; Git for Try Django 3.2
slug: version-control-with-git-basics-for-try-django-32

publish_timestamp: July 28, 2021
url: https://www.codingforentrepreneurs.com/blog/version-control-with-git-basics-for-try-django-32/

---


Putting code into production is made much easier thanks to the concept of version control and the technology known as `git`.

Think of version control as a way to track & review your *"save"* history across many files and many lines of text.

This is great for several reasons:
- Ease of deploying code
- Ease of sharing code
- Ease of reversing typos/glitches/bugs/failed features/etc
- Ease of reviewing changes prior to going live
- Ease of remotely testing code
- Ease of reversing releases
- Ease of protecting code

This post is a reference guide for [this video](https://www.codingforentrepreneurs.com/projects/try-django-3-2/managing-code-git-github) in the [Try Django 3.2](https://www.codingforentrepreneurs.com/projects/try-django-3-2) series. I highly recommend you watch that video if you've never done `git` before. 

## Step 1: Download + Configure Git & GitHub

1. Signup for a [github](https://github.com/signup) account.

2. Download git [here](https://git-scm.com/downloads)

3. Add in your credentials ([reference](https://docs.github.com/en/get-started/getting-started-with-git/caching-your-github-credentials-in-git)):
 
Mac users:
```
git config --global credential.helper osxkeychain
```

Windows users:
```
git config --global credential.helper wincred
```


## Step 2: Initialize Git
This is to setup your folder to be tracked by `git`. Nearly *anything* can be tracked by `git`. If the file is too large, then you'll run into issues, everything else is pretty much fair game.

Run this command 1 time per root folder you want to track.
```bash
cd path/to/your/project/folder/
git init
```


## Step 3: Git Ignore File
Most of the time, you'll want to ignore files from being tracked. Sometimes these files hold sensitive information, sometimes these files are related to environment setup (python virtual environments, npm packages, etc) either way, having a `.gitignore` for *every project you track with git* is **highly recommended**.

1. Go to [this link](https://github.com/github/gitignore) as it is a list of common `.gitignore` examples.

2. I'll use the [python `.gitignore`](https://github.com/github/gitignore/blob/master/Python.gitignore) for my Python projects or the [Node.js `.gitignore`](https://github.com/github/gitignore/blob/master/Node.gitignore) for my JavaScript projects.

3. Once you pick the correct `.gitignore` for your project, add a `.gitignore` file to your recently tracked `git` folder (aka *repository*). It will look start with something like:

```
# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/

...
```
> Note: the above `.gitignore` example is incomplete on purpose since it's so long.

## Step 4: Configure
If you're using `github`, I recommend you use the following settings (replace `teamcfe` with your username):

```bash
git config user.name "teamcfe"
git config user.email "teamcfe@users.noreply.github.com"
```

## Step 5: Ongoing Usage.

Now that you have your folder tracked by git (this folder is also called your local repository), you can use the most common beginner-friendly git commands.

### `git add <filename/foldername>`

### `git commit -m "<your message>"`

### `git log`
This will show you a history of all of your commits.

### `git reset <commit-id> --hard`
Long term, you want to avoid using this command. I think it's good for beginners since doing `git` well is a bit tricky. This will reset your code to a previous commit id. You can get these ids from `git log`.

### `git remote add origin https://github.com/teamcfe/Try-Django-3.2`
The url `https://github.com/teamcfe/Try-Django-3.2` is unique to my project & my account. This is merely how you let your local git project to know about where to send/receive your code (ie doing `git push` and `git pull`).


### `git push origin main`
This command will push your local code into your remote repository (if it exists).

Given the settings from above, this should prompt you for your github password each time.

### `git pull origin main`
This will pull your code from your remote repository to your local one.


### `git clone https://github.com/codingforentrepreneurs/Try-Django-3.2 .`
This will download the code from a remote repo (aka repository)which may or may not be a public repo. It will also download a copy of the entire commit history (aka code change history).

Use this command on a git-less folder/directory. It will download the code so you can use it as needed.
