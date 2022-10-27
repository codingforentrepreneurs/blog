---
title: Hello Linux: Custom Domain &amp; Https with Let&#x27;s Encrypt
slug: custom-domain-and-https-with-lets-encrypt

publish_timestamp: March 14, 2019
url: https://www.codingforentrepreneurs.com/blog/custom-domain-and-https-with-lets-encrypt/

---

<div class='alert alert-success'>This is the sixth post of a many part series. <a href='https://www.codingforentrepreneurs.com/blog/hello-linux/'>This post</a> is the starter post for the whole series.</div>

Let's Encrypt is one of the best things to happen to the internet in a long time. It is a provider of security certificates so we can have https for free. Yup free. These certificates _still__ can cost anywhere from $150 to $2k+ USD per year.

Since Let's Encrypt Exists, there is **no reason** to not have https on our site.

*********
**Requirements**

Be sure to complete
- [Git Push Local Code to Live Linux Server](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Virtual Environment in Working Directory](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- [Setup Gunicorn & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor)
- [Nginx & UFW Firewall](https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall)

**Our Live Server**
- **Ubuntu 18.04**

*********

#### 1. SSH into your server

```
ssh root@104.248.231.241
```
Replace `root@104.248.231.241` with your user / ip



#### 2. Point Your Custom Domain to Your Server's IP Address

```
localip=$(hostname  -I | cut -f1 -d' ')
echo $localip
```
Grab this value for the setting up your domain. Mine was `104.248.231.241`.

Set an `A Record` on your domain registrar for each subdomain you want to point to your server. 

Here's an example step-by-step on [name.com](https://www.name.com/referral/5a470)

1. Login on [name.com](https://www.name.com/referral/5a470)
2. Go to [my domains](https://www.name.com/account/domain)
3. Click your domain such as `https://www.name.com/account/domain/details/codingforawesome.com`
4. Click on `DNS Records`
5. Use the following settings:

Subdomain = `www` (aka host)
```
Type: A
Host: www
Answer: 104.248.231.241 (replace with your value)
TTL: 300 (or whatever default)
```
Click Add Record


Empty Host
```
Type: A
Host: 
Answer: 104.248.231.241 (replace with your value)
TTL: 300 (or whatever default)
```



#### 3. Install `certbot` by Let's Encrypt

Back in your ssh session, let's run:

```
sudo add-apt-repository ppa:certbot/certbot

sudo apt-get update -y

sudo apt-get install python-certbot-nginx -y
```

#### 4. Update Nginx Settings to New Domain
From [this post](https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall) we created the following:

`/etc/nginx/sites-available/myproject.conf`

Let's edit that file:

`nano /etc/nginx/sites-available/myproject.conf`

We'll just change server_name from your `ip_address` to your custom domain (including the hosts).

Let's say you added `www.codingforawesome.com`, `codingforawesome.com`, and `blog.codingforawesome.com`, then your conf file would look like:
```
server {
    server_name codingforawesome.com www.codingforawesome.com blog.codingforawesome.com;
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
Save and close.


Let's verify that the file in `sites-enabled` has the same data.

```
nano /etc/nginx/sites-enabled/myproject.conf
```
It should be the same.


#### 5. Run Let's Encrypt

It's important that you use your domain here and *very* important that your domain is already pointing to your server.

The root call is `sudo certbot --nginx` and just append `-d <your-domain>` for every domain and subdomain you want to include in this certificate.

```
sudo certbot --nginx -d codingforawesome.com -d www.codingforawesome.com -d blog.codingforawesome.com
```

This command will ask you if you want to _redirect_ http to https, I think you should **redirect  HTTP traffic to HTTPS**.

If you did this successfully, your nginx configuration file will _change_ to work with https. Since we have our `ufw` firewall settings to `Nginx Full` we should have no problems accessing https going forward.


Finally, verify auto renew is on:
```
sudo certbot renew --dry-run
```

#### 6. And we're live.

For many Django projects, that's all we need to do. We have almost everything we need to ensure we can run a production server with a production database along with push through git.

Pretty cool huh?

If you're using digital ocean. This entire process can start as low as $5 month. I think that's pretty awesome.


#### Wrap Up

**We now have**
- [Git Setup for Push/Pulling Code From Local to Production](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Working Directory with a Virtual Environment](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- [Setup Gunicorn & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor/) 
- [Nginx & UFW Firewall](https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall)
- Custom Domain + HTTPs with Let's Encrypt (this post)

**Still coming, but optional**
- Task Queues with Celery & Redis