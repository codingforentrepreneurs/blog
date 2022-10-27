---
title: Local Domain &amp; Subdomain Testing in Mac &amp; Linux
slug: local-domain-subdomain-testing-in-mac-linux

publish_timestamp: Jan. 30, 2017
url: https://www.codingforentrepreneurs.com/blog/local-domain-subdomain-testing-in-mac-linux/

---


What to do when you want to test a domain name locally? Such as `www.yourawesomeproject.com` or what about a subdomain (aka wildcard domain) like `blog.yourawesomeproject.com` or `jmitchel3.yourawesomeproject.com`? Here's a quick guide to do it:

1.  Open up **terminal**

2. `sudo nano /etc/hosts`
You should see something like:
```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
```

3. Add each subdomain you want to use
```
127.0.0.1       localhost
127.0.0.1       www.yourawesomeproject.com
127.0.0.1       yourawesomeproject.com
127.0.0.1       blog.yourawesomeproject.com
```

4. Run your project such as with Django:
```
python manage.py runserver
```

5. Open your browser to:
```
http://www.yourawesomeproject.com:8000
```
or
```
http://blog.yourawesomeproject.com:8000
```
Notice that `8000` is the default port for running Django. You can change this port number as needed.

Enjoy!
