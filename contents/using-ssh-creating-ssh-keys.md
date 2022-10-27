---
title: Using SSH &amp; Creating SSH Keys
slug: using-ssh-creating-ssh-keys

publish_timestamp: March 15, 2019
url: https://www.codingforentrepreneurs.com/blog/using-ssh-creating-ssh-keys/

---


#### [Video](https://www.codingforentrepreneurs.com/projects/ssh/ssh-ssh-keygen)

Using a Secure Shell (`ssh`) will allow your local computer to communicate with a live server. `ssh` will allow you to run commands, install software, setup a server, and do all kinds of interesting things.

Let's setup ssh locally.

> Note for Windows Users, it's recommended that you use PowerShell or [PuTTY](https://www.putty.org/)

Mac and Linux often have the ability to do `ssh` by default because they both have a _bash_ shell within the `Terminal` application.



#### 1. Generate a ssh key

Open the *Terminal* app an run
```
ssh-keygen
```
> On linux, you may see an error requiring you to install OpenSSL, do all installations it suggests.


Assuming you're new to this, __just hit enter a few times__ after you run the command `ssh-keygen` and you'll see:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/cfe/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/cfe/.ssh/id_rsa.
Your public key has been saved in /Users/cfe/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:2eog3rVG7l+Hg01YcNmJqTzoD4vD7dHPv0BTx1Lbf7E cfe@Justins-MacBook-Pro.local
The key's randomart image is:
+---[RSA 2048]----+
|          . .= ..|
|           o+ ooo|
|         o .. oo+|
|        .o+o . o+|
|       .S o.+  Eo|
|        +o = o  .|
|    ...=++o * .  |
|   . o++=o.+ +   |
|    . .==.. o.o. |
+----[SHA256]-----+
```

In this case, I didn't set a passphrase. If you want extra security you can, just run `ssh-keygen` again and overwrite your old key.


> Overwriting ssh keys is perfectly fine as long as you know what it means: it's like changing your password so old `ssh` connections won't work any more.



#### 2. Copy ssh public key (the file ending in `.pub`)

Navigate to where ssh keys are stored.

```
cd ~/.ssh
```

This is where you should always be able to find your keys (if you have them) and `known_hosts` (which we'll cover shortly).

If you run `ls` you should see this:
```
$ cd ~/.ssh
$ ls
id_rsa      id_rsa.pub
```

`id_rsa.pub` is our public key. This is the one we can add to services that need our `ssh public key`. 

Open `id_rsa.pub` in your text editor. It should look something like:
```
ssh-rsa AAAAB3Nzde3c1yc2EAAAADAQABAAABAQDnBBPCh9S4VvCrHDvUIFEApZYS8U1834p1qr1zDUDKrfrP9RZjIjD1tnobl/SCJZZtXaatGBzZdKK3XRk9yVNQy3ogTe/7aaddsafddM6tfb7Idk6ghEr4JyvBOdL/lSFtpT16+B7ol7LFNECpwerLbhaeE7Olgl/kOmNrweQQNuleCbDk/rQ3f0VZHEgdevHkcBQrhIB12mhOg20XI9iXb1szdE3ewerweEv/CI4rRew3Ndj5R7 cfe@Justins-MacBook-Pro.local
```
A shortcut to copy the above text is to run:
```
cat ~/.ssh/id_rsa.pub | pbcopy
```

Now, *add this key* to any service that requires it. Here are a few that come to mind:

- Digital Ocean
- Github
- Bitbucket
- Linode
- AWS 
- GCP

#### 3. Test an ssh connection.

This will definitely require you to have something to `ssh` in to. Let's say you set up a Ubuntu droplet on Digital Ocean, like what we do in [hello linux](https://www.codingforentrepreneurs.com/blog/hello-linux/), you can now run:

```
ssh <your-user>@<your-server-ip>
```

Like
```
ssh root@104.248.231.241
```

If this works, you should have access to your server. Keep in mind that you can also use these keys for doing `git push` to places like `github` so you don't have to authenticate each time.


Did we miss anything? Let us know in the comments.
