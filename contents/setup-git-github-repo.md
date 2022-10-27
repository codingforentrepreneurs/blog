---
title: Setup Git &amp; A Github Repo
slug: setup-git-github-repo

publish_timestamp: June 22, 2017
url: https://www.codingforentrepreneurs.com/blog/setup-git-github-repo/

---


# Git is a version control system. It allows you to track any/all changes to your project.

> [Watch on YouTube](https://youtu.be/dlz6Qyp4ADE)

It's also a very easy way to share code with everyone. Like we do on our [Github repo](https://kirr.co/vf3249).

You can also use Git to deploy (aka send) your code to a live production service on services like [Heroku](https://www.codingforentrepreneurs.com/projects/heroku/), [AWS Elastic Beanstalk](https://www.codingforentrepreneurs.com/projects/elastic-beanstalk/), and others.

(Optional) Follow this guide using [our blank Django project](https://www.codingforentrepreneurs.com/blog/create-a-blank-django-project/)

### 1. Install *git* via:
- [Download Heroku CLI](https://kirr.co/5wd1ay) (preferred)
- [Homebrew](https://kirr.co/qvzmnv)
- From [git-scm](https://git-scm.com/downloads) itself.

### 2. Open Terminal and Verify installation
```
$ git --version
git version 2.13.1
```

### 3. Initialize *git* in a directory (folder) you want to "track" changes with.
```
$ cd ~/Dev/cfehome/
$ git init 
```
### 4. Create *.gitignore*
The purpose of this is to "ignore" files from being tracked with git. This helps save space and removes unnecessary files.  All kinds of pre-built software gitignore files can be found [here](https://kirr.co/4h30b1)
```
$ echo "*.py[cod]
.DS_Store
__pycache__/
*.py[cod]
*$py.class
" > .gitignore
```

### 5. Check files status
```
$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	db.sqlite3
	manage.py
	cfehome/
	requirements.txt

nothing added to commit but untracked files present (use "git add" to track)
```

### 6. Add all files and commit
```
$ git add --all
$ git commit -m "Add commit message"
```

### 7. With a remote repo ready (create below), you just have to push changes
```
$ git push origin master
```


----------
## Create a Github Repository (remote git repo)

1. Create Account & Login on [https://github.com](https://github.com)

2. [Add New Repository](https://github.com/new)

3. Give it a name and description. Do not add `.gitignore` or `readme` unless you know what you're doing.

4. Follow instructions for "adding to existiing respository".

5. Push Local File!
```
$ git push origin master
```

----------
[Watch](https://youtu.be/dlz6Qyp4ADE):
<iframe width="560" height="315" src="https://www.youtube.com/embed/dlz6Qyp4ADE" frameborder="0" allowfullscreen></iframe>
