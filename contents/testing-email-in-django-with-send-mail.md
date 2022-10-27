---
title: Testing Email in Django with send_mail
slug: testing-email-in-django-with-send-mail

publish_timestamp: Jan. 3, 2017
url: https://www.codingforentrepreneurs.com/blog/testing-email-in-django-with-send-mail/

---


Sometimes you need to test that your Django project is sending email correctly. Here's a way to test it.

First, in your `settings.py` (or Django settings module), you should have the following set to your email provider:

```
# gmail settings
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = 'youremail@gmail.com' 
EMAIL_HOST_PASSWORD = 'yourpassword'
EMAIL_PORT = 587
EMAIL_USE_TLS = True


#sendgrid settings
EMAIL_HOST = 'smtp.sendgrid.net'
EMAIL_HOST_USER = 'yourusername@youremail.com'
EMAIL_HOST_PASSWORD = 'your password'
EMAIL_PORT = 587
EMAIL_USE_TLS = True


DEFAULT_FROM_EMAIL = 'Your Name <you@email.com>'
ADMINS = (
    ('You', 'you@email.com'),
)

MANAGERS = ADMINS

```

Once you have that setup, you can do a test mail in `python manage.py shell` to use [Django send_mail](https://docs.djangoproject.com/en/1.10/topics/email/#send-mail)

```
from django.conf import settings
from django.core.mail import send_mail

subject = 'Some subject'
from_email = settings.DEFAULT_FROM_EMAIL
message = 'This is my test message'
recipient_list = ['mytest@gmail.com', 'you@email.com']
html_message = '<h1>This is my HTML test</h1>'


send_mail(subject, message, from_email, recipient_list, fail_silently=False, html_message=html_message)

```

Happy coding!
