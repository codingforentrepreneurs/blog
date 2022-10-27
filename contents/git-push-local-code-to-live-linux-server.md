---
title: Git Push Local Code to Live Linux Server
slug: git-push-local-code-to-live-linux-server

publish_timestamp: March 13, 2019
url: https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server/

---


_This is the first post of a many part [Linux/Ubuntu deployment series](https://www.codingforentrepreneurs.com/blog/hello-linux/). Part two is [here](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory/)._


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

### Step 1. Install git locally and create local repo
> Did I mention this step is local? 

Download and install **git** from [https://git-scm.com/downloads](https://git-scm.com/downloads) .

Then,
```bash
mkdir Dev
cd Dev
mkdir myproject
cd myproject
git init
```

*********

### Step 2. SSH into Live Server
```bash
ssh root@104.248.231.241
```
Replace `root@104.248.231.241` with your user and ip

*********

### Step 3. Install git on Server
```bash
sudo apt-get install git -y 

```

*********

### Step 4. Create bare git repo

```bash
cd /var/
mkdir repo
cd /var/repo/
mkdir myproject.git
cd myproject.git
```

__Starting fresh?__
```bash
git init --bare
```

__Using previous?__
```bash
git clone https://github.com/codingforentrepreneurs/CFE-Blank-Project . --bare
```

*********

### Step 5. Add `post-receive` hook
Hooks are very useful for doing things automatically while working with git. 

In our case, we need a hook for after a successful push aka `post-receive` to unpack our code into a server-side working directory.

__Make initial working directory__
```bash
mkdir /var/www/
mkdir /var/www/myproject/
```

__View all hook samples__
```bash
ls -al /var/repo/myproject.git/hooks/
```

__Create the actual `post-receive` hook__
```bash
cd /var/repo/myproject.git/hooks/
nano post-receive
```

__In `post-receive`__
```bash
git --work-tree=/var/www/myproject/ --git-dir=/var/repo/myproject.git/ checkout -f
```

__Make hook executable__
```bash
cd /var/repo/myproject.git/hooks/
chmod +x post-receive
```

*********

### Step 6. Get host name
```bash
myip=$(hostname  -I | cut -f1 -d' ')
echo $myip
```

This will yield a value of something like `104.248.231.241` but with your actual value.

*********

### Step 7. Add to local git repo

Exit live server
```bash
exit
```

Change into local working directory

```bash
# on local computer
cd ~/Dev/myproject
```

Check git is working... No errors right?
```bash
git status
```

Add live remote
```bash
git remote add live ssh://root@104.248.231.241/var/repo/myproject.git
```

The format is this:

`git remote add remote-name ssh://your-user@your-host-or-ip-address/var/repo/your-project.git`

*********

### Step 8. Make Changes Locally and Push

```bash
cd ~/Dev/myproject
echo "hello there" > test.txt
git add 'test.txt'
git commit -m "Local file test"
git push live master
```

*********

### Step 9. SSH again into your server.... 

```bash
ssh root@104.248.231.241
```
Can you find your code? How did it go?

*********

### Step 10. Need that code on another local computer?
```bash
cd path/to/local/dir
git init
git remote add live ssh://root@104.248.231.241/var/repo/myproject.git
git pull live master
```

*********

### Next steps
Part two is [here](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory/). 

We'll be covering how to go from local to fully running production code as outline on [the official repo](https://github.com/codingforentrepreneurs/Hello-Linux). We started with git because every great project should start with `git init`.
