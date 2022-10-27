---
title: Using Multiple SSH Keys
slug: using-multiple-ssh-keys

publish_timestamp: March 15, 2019
url: https://www.codingforentrepreneurs.com/blog/using-multiple-ssh-keys/

---


Every time you use a new host, it might be a good idea to create a key specific for that host. It's simple to do:

```
nano ~/.ssh/config
``` 

In `~/.ssh/config`
```
Host shortcut_name
  HostName <ip-address>
  User <username>
  IdentityFile <path/to/ssh/key/without/dotpub>
```


Example
```
Host do_app
  HostName 134.2109.124.156
  User root
  IdentityFile ~/.ssh/do_rsa

Host lin_app
  HostName 214.2109.124.156
  User root
  IdentityFile ~/.ssh/lin_rsa
```

Now you can do a simple ssh using the shortcut name(s):
```
ssh do_app
```

This will also work for git:

```
git remote add ssh://do_app:/path/to/live/repo/<project>.git
```
