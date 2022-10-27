---
title: Git Push Local Code to Live Linux Server
slug: git-push-local-code-to-live-linux-server

publish_timestamp: March 13, 2019
url: https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server/

---

<div class='alert alert-success'>This is the first post of a many part <a href='https://www.codingforentrepreneurs.com/blog/hello-linux/'>Linux/Ubuntu deployment series</a>. Part two is <a href='https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory/'>here</a>.</div>


Git is a version control system. It's a simple way to track changes in your code.

It also can be a simple way to deploy your code to a live server. 

> Don't have a live server? Pick one up for $5/mo on [Digital Ocean](https://kirr.co/l8v1n1) ([no referral](https://www.digitalocean.com/))

Once you set it up 1 time successfully. You'll never want to user another method (I'm looking at you SFTP).

Let's do this.

*********
**Basic Requirments**

**Mac / Linux** 
- You'll need to use Terminal.

**Windows**
- use PowerShell or possibly [PuTTY](https://www.putty.org/) (for SSH). This guide *might* not work for your version Windows too. :()

**Live Server Details**
- **Ubuntu 18.04** is what we use here but not required.
- **Other Linux Distro?** Check how to install git, most/all other things are the same.

*********

### 1. Install git locally and create local repo
> Did I mention this step is local? 

Download and install via [https://git-scm.com/downloads](https://git-scm.com/downloads) 

Then,
```
mkdir Dev
cd Dev
mkdir myproject
cd myproject
git init
```

### 2. SSH into Live Server
```
ssh root@104.248.231.241
```
Replace `root@104.248.231.241` with your user / ip


### 3. Install git on Server
```
sudo apt-get install git -y 

```



### 4. Create bare git repo

```
cd /var/
mkdir repo
cd /var/repo/
mkdir myproject.git
cd myproject.git
```

#### Starting fresh?
```
git init --bare
```

#### Using previous?
```
git clone https://github.com/codingforentrepreneurs/CFE-Blank-Project . --bare
```


### 5. Add `post-receive` hook
Hooks are very useful for doing things automatically while working with git. 

In our case, we need a hook for after a successful push aka `post-receive` to unpack our code into a server-side working directory.

#### 1. Make initial working directory
```
mkdir /var/www/
mkdir /var/www/myproject/
```

#### 2. View all hook samples:
```
ls -al /var/repo/myproject.git/hooks/
```

#### 3. Create the actual `post-receive` hook.
```
cd /var/repo/myproject.git/hooks/
nano post-receive
```

In `post-receive`:
```
git --work-tree=/var/www/myproject/ --git-dir=/var/repo/myproject.git/ checkout -f
```

#### 4. Make hook executable
```
cd /var/repo/myproject.git/hooks/
chmod +x post-receive
```


### 6. Get host name
```
myip=$(hostname  -I | cut -f1 -d' ')
echo $myip
```

This will yield a value of something like `104.248.231.241` but with your actual value.


#### 7. Add to local git repo
1. Exit live server
```
exit
```

2. Change into local working directory

```
# on local computer
cd ~/Dev/myproject
```

3. Check git is working... No errors right?
```
git status
```

4. Add live remote
```
git remote add live ssh://root@104.248.231.241/var/repo/myproject.git
```

The format is this:

`git remote add <remote-name> ssh://<user>@<ip-or-host>/var/repo/<your-project>.git`


#### 8. Make Changes Locally and Push

```
cd ~/Dev/myproject
echo "hello there" > test.txt
git add 'test.txt'
git commit -m "Local file test"
git push live master
```


#### 9. SSH into your server.... how did it go?
If it worked. How cool is that? A simple, yet effective way to push code to a server. 

#### 10. Need that code on another local computer?
```
cd path/to/local/dir
git init
git remote add live ssh://root@104.248.231.241/var/repo/myproject.git
git pull live master
```

#### Next steps
Part two is [here](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory/). 

We'll be covering how to go from local to fully running production code as outline on [the official repo](https://github.com/codingforentrepreneurs/Hello-Linux). We started with git because every great project should start with `git init`.