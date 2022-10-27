---
title: Configure Email in Django
slug: configure-email-in-django

publish_timestamp: June 8, 2017
url: https://www.codingforentrepreneurs.com/blog/configure-email-in-django/

---

Email configuration is simple. You just go into your `DJANGO_SETTINGS_MODULE` (aka `settings.py`) and add the following:

```
EMAIL_HOST = 'smtp.email-host-provider-domain.com'
EMAIL_HOST_USER = 'yourusername@youremail.com'
EMAIL_HOST_PASSWORD = 'your password'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
DEFAULT_FROM_EMAIL = 'Your Name <you@email.com>'
```

### Do you want to receive server errors to your inbox? Add this:
```
ADMINS = (
    ('You', 'you@email.com'),
)
MANAGERS = ADMINS
```


### Additional Resources
- Test sending email with [Django's send_mail](https://www.codingforentrepreneurs.com/blog/testing-email-in-django-with-send-mail/)
- Use [Gmail with Django](https://www.codingforentrepreneurs.com/blog/use-gmail-for-email-in-django/)
- Setup [Sendgrid in Django](https://www.codingforentrepreneurs.com/blog/sendgrid-email-settings-for-django/)