---
title: Invictify - A Scheduler for Triggering Workflows
slug: invictify-scheduler-for-triggering-workflows

publish_timestamp: March 30, 2020
url: https://www.codingforentrepreneurs.com/blog/invictify-scheduler-for-triggering-workflows/

---

Hi there, 

I've recently created [Invictify.com](https://invictify.com/cfe) as a way to save time & money while helping increase running scheduled workflows and automations.

In [30 Days of Python](/projects/30-days-python-38/) we'll soon be showing you how exactly to use [Invictify](https://invictify.com/cfe) in your projects but here's a basic example in Django:

```python
# workflows.handlers.py
def my_awesome_workflow():
    print("hello world")


# webhooks.views.py
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt

from workflows.handlers import my_awesome_workflow # replace with yours

@csrf_exempt
def my_awesome_worfklow_view(request): 
     if not request.method == POST:
           return HttpResponse(status=400)
     # Run workflow
     my_awesome_workflow()
     return HttpResponse(status=200)


# urls.py

from django.urls import path, re_path, include

from webhooks.views import my_awesome_worfklow_view 

urlpatterns = [
    re_path(r"^webhooks/my-awesome-workflow/?$", my_awesome_worfklow_view),
]
```

Now, add your live project's domain `http://yourdomain.com/webhooks/my-awesome-workflow/` to Invictify.com and create a schedule for when it should be triggered.

That's it! Now the schedule you set (either an interval or crontab) will run as expected by calling the endpoint you designate in a trigger.

Okay so what?

This means you don't have to spin up your own worker process to run things like:

- A web scraping workflow every 30 minutes
- A home automation task every day at 1am
- A database backup every day at 3:45am UTC
- Image thumbnail creation every 2 hours
- Web page health lookup every 20 seconds

Invictify.com doesn't fully replace the use of workers yet (like Celery), but we hope that one day it will and for significantly less cost than running your own workers.

And yes, there's a [forever free tier](https://www.invictify.com/pricing/) to get started.

I hope you enjoy it. 

Cheers!