---
title: Hello Linux: Nginx &amp; UFW Firewall
slug: hello-linux-nginx-and-ufw-firewall

publish_timestamp: March 14, 2019
url: https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall/

---


<div class='alert alert-success'>This is the fifth post of a many part series. <a href='https://www.codingforentrepreneurs.com/blog/hello-linux/'>This post</a> is the starter post for the whole series.</div>

Nginx is free, open-source, high-performance HTTP server and reverse proxy. Pronounced "Engine X"

UFW, aka uncomplicated firewall, which makes firewall configuration easy.

*********
**Requirments**

Be sure to complete
- [Git Push Local Code to Live Linux Server](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Virtual Environment in Working Directory](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- [Setup Gunicorn & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor)

**Our Live Server**
- **Ubuntu 18.04**

*********

#### 1. SSH into your server

```
ssh root@104.248.231.241
```
Replace `root@104.248.231.241` with your user / ip



#### 2. Install Nginx and ufw

```
sudo apt-get update -y

sudo apt-get install nginx ufw -y
```


#### 3. Enable `ufw` Firewall Defaults

We want ssh and Nginx to work. `Nginx Full` allows for both `http` (80) and `https` (443) connections.

```
sudo ufw allow ssh

sudo ufw allow 'Nginx Full'
```

Enable. Ensure you have `ssh` above otherwise you will lose your connection and *not* get it back.
```
sudo ufw enable
```
If you're not kicked off your ssh session, you can now proceed.

Check your status
```
sudo ufw status numbered
```
You should see this:
```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere                  
[ 2] Nginx Full                 ALLOW IN    Anywhere                  
[ 3] 22/tcp (v6)                ALLOW IN    Anywhere (v6)             
[ 4] Nginx Full (v6)            ALLOW IN    Anywhere (v6)  
```

#### 4. Remove Nginx Default
After you install `nginx` like we did in step 2, you can go directly to your server's ip address. You should see some default nginx page.

We can remove that with:

```
rm /etc/nginx/sites-enabled/default
```
#### 5. What's my ip?

```
localip=$(hostname  -I | cut -f1 -d' ')
echo $localip
```
Grab this value for the next step. Mine was `104.248.231.241`. This should match what you ssh into like in _step 1_.

#### 6. Nginx Configuration
Now, we have `gunicorn` as our project server. Nginx needs to use the `guincorn` socket. From [this post](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor/) we set the socket file to be located on `/var/www/myproject/src/myproject.sock`


We'll set our `server_name` to `104.248.231.241` for now. We'll do a custom domain as well as https in another post.

Create your nginx conf file
```
nano /etc/nginx/sites-available/myproject.conf
```

Add the following with your details.

Replace `proxy_pass` with `http://unix:/your/local/gunicorn/socket/file.sock`
Replace `server_name` your your ip address from _Step 5_.

```
server {
    server_name 104.248.231.241;
    listen 80;
    listen [::]:80;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/myproject/src/myproject.sock;
        proxy_buffer_size       128k;
        proxy_buffers           4 256k;
        proxy_read_timeout      60s;
        proxy_busy_buffers_size 256k;
        client_max_body_size    2M;
    }
}
```

Now that you have your configuration in `sites-available` it's time to link it to `sites-enabled`


```
ln -s /etc/nginx/sites-available/myproject.conf /etc/nginx/sites-enabled/myproject.conf
```

#### 6. Reload Daemon and Nginx
```
systemctl daemon-reload

sudo systemctl reload nginx
```



#### 7. Update our repo `post-receive`

Assuming you completed [this post](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server), you should do this:

```
cd /var/repo/myproject.git/hooks/
nano post-recieve
```

```
git --work-tree=/var/www/myproject/ --git-dir=/var/repo/myproject.git/ checkout -f


/var/www/myproject/bin/python -m pip -r /var/www/myproject/src/requirements.txt

systemctl daemon-reload

sudo systemctl reload nginx
```

Now when you push a new version of your code, you're nginx system will automatically reload (assuming no errors) without the system missing a beat.


#### Wrap Up

**We now have**
- [Git Setup for Push/Pulling Code From Local to Production](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Working Directory with a Virtual Environment](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- [Setup Gunicorn & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor/) 
- Nginx & UFW Firewall (this post)

**Now we need to**
- [Add a Custom Domain and Setup Let's Encrypt for HTTPs](https://www.codingforentrepreneurs.com/blog/custom-domain-and-https-with-lets-encrypt)
