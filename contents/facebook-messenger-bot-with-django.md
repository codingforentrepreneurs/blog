---
title: Facebook Messenger Bot with Django
slug: facebook-messenger-bot-with-django

publish_timestamp: March 23, 2018
url: https://www.codingforentrepreneurs.com/blog/facebook-messenger-bot-with-django/

---


We still have to do a live test on this one so keep in mind you might find a bug or two until we do a live test. In the meantime... enjoy!


First things first. This is **not** a AI bot. Instead, it's a messenging bot between your Django project, your team, and your a facebook user. I would love to create a bot that has human-level comprehension in the future so please let me know of resources you find to do so in the comments.

With that said, let's get started.

###  Requirements
- A solid base of Django skill. Unsure? Take a look at the code [here](https://github.com/codingforentrepreneurs/Try-Django-1.11). If you can make sense of that code, you're good. If not, watch [this entire series](https://www.youtube.com/watch?v=yDv5FIAeyoY)
- A [facebook developer](https://developers.facebook.com/) account
- A live HTTPs web application (something like what we build [in this series](https://www.youtube.com/watch?v=yDv5FIAeyoY)); with [letsencrpyt](https://letsencrypt.org/) you have no reason to not have HTTPs on a live site anymore. It's free.


Here's the order of operations
1. The Logic & Django Setup
2. Facebook Developer App setup
3. Django Setup Part 2
4. Local Tests
5. Submit approval to Facebook


### 1. The Logic & Django Setup
Django will be handling the backend logic for our messenger system. This logic can be reapplied to other messenger-like apps such as Google Home, Alexa, and any others that come out.

#### The Logic
Giving automatic responses to a user's requests is a non-trival task. With that in mind, we can still automate parts of the auto-responder to be a littler smarter than just the same messenger sending back over and over again. You'll want to think through this a lot because it's a key part to making your bot function, well, almost good enough to feel like an authentic human response.

Below is the actual logic we've been using for our [messenger bot](http://joincfe.com/facebook). Why does it look like garbage? Well, we don't receive that many facebook messages and we'll still in R&D for making a true ai-based chatbot. But that's a story for another time.


```python
# bot.logic.py
LOGIC_RESPONSES = { 
    'account': [
        "You can find your account information on https://www.codingforentrepreneurs.com/account/"
    ], 
    'enroll': [
        "You can simply enroll on https://www.codingforentrepreneurs.com/enroll/"
    ], 
    "projects": [
        "All of our projects are on http://joincfe.com/projects and courses are on http://joincfe.com/courses/"
    ],
    "free": [
        "All of our free content is on http://joincfe.com/youtube/."
    ],
    "membership": [
        "You can enroll for a pro membership on http://joincfe.com/enroll/ to get access to all of our content in HD!"
    ],
    "afford": [
         "Have you thought about subscribing to our free YouTube channel?  It's on http://joincfe.com/youtube/."
    ],
    "blog": [
        "Our blog is on http://joincfe.com/blog/"
    ],
    "guides": [
        "You can access our guides blog is on http://joincfe.com/blog/"
    ],
    "thank": [
         "Of course!",
        "Anytime!",
        "You're welcome",
         "You are so welcome!"
    ],
    "thanks": [
        "Of course!",
        "Anytime!",
        "You're welcome",
         "You are so welcome!"
    ],
    'help': [
        "A good place to get help is by going to our forums on http://joincfe.com/ask/", 
        "You can always ask questions in our videos or on http://joincfe.com/ask/"
    ],
    'code': [
        "Have you considered looking at our code on http://joincfe.com/github/? That might help you",
        "We don't review code at this time, but you can consider looking at our open-source repo http://joincfe.com/github/"
    ],
    'human': [
        'A real person will get back to you shortly!'
    ]
}
```

Eventually, we'll implement code like 
```
for token in tokens:
    if token in responses:
       msg = random.choice(responses[token])
       break
```
Which will select random "responses" for any given `token` (more on that later). So far simple enough? 

#### Create a webook view

This webhook view will be where Facebook sends a user's message. This view will consume the message and then relay to facebook's servers with the reponse for that user; if there is one.

> You can absolutely use this to just create a "inbox" of all the messages your page's messenger receives. This might be a great way to just start figuring out common questions and patterns your page gets.

To get started, let's generate a couple hext strings.


```python
# Generate a 2 random hex string for your webhook endpoint and the VERIFY_TOKEN
# You only need to create these 1 time; you can always change them later but
# you don't need to generate them constantly

import os, binascii
webhook_endpoint = (binascii.b2a_hex(os.urandom(30))).decode("utf-8")
print(webhook_endpoint)

VERIFY_TOKEN = (binascii.b2a_hex(os.urandom(25))).decode("utf-8") # put this value in bot.views.py
print(VERIFY_TOKEN)
```

    45c66b3a06d11ad094a28d752f174d756d44e8cc242f44fe05d6193980e0
    7711df715abcfa28ace91507da2d28d907a2d2db3c7c6639b0



```python
# bot.views.py
import json
import requests, random, re
from django.http import HttpResponse, JsonResponse
from django.views.generic import View
from django.views.decorators.csrf import csrf_exempt
from django.utils.decorators import method_decorator

from .logic import LOGIC_RESPONSES


VERIFY_TOKEN = "7711df715abcfa28ace91507da2d28d907a2d2db3c7c6639b0" # generated above

"""
FB_ENDPOINT & PAGE_ACCESS_TOKEN
Come from the next step.
"""
FB_ENDPOINT = 'https://graph.facebook.com/v2.12/'
PAGE_ACCESS_TOKEN = "EAACh8LwRraIBAB6wNqmxZC2K2bA55GURfPNWDEFSfQNSUiRa3I49oVgZCxcuun6zTrsw0s4ZCUhXQ33kPb7zN3NSmGZAoJYZBw0ZB762DDnAp3TD9eqVN3yUwcrpyOsBldoFWZAxCmELsSUza8teXsZAYr07VodZAlFWlMttmqWv1DewZDZD"  




def parse_and_send_fb_message(fbid, recevied_message):
    # Remove all punctuations, lower case the text and split it based on space
    tokens = re.sub(r"[^a-zA-Z0-9\s]",' ',recevied_message).lower().split()
    msg = None
    for token in tokens:
        if token in LOGIC_RESPONSES:
            msg = random.choice(LOGIC_RESPONSES[token])
            break
        
    if msg is not None:                 
        endpoint = f"{FB_ENDPOINT}/me/messages?access_token={PAGE_ACCESS_TOKEN}"
        response_msg = json.dumps({"recipient":{"id":fbid}, "message":{"text":msg}})
        status = requests.post(
            endpoint, 
            headers={"Content-Type": "application/json"},
            data=response_msg)
        print(status.json())
        return stats.json()
    return None
        
        
class FacebookWebhookView(View):
    @method_decorator(csrf_exempt) # required
    def dispatch(self, request, *args, **kwargs):
        return super().dispatch(request, *args, **kwargs) #python3.6+ syntax
    
    '''
    hub.mode
    hub.verify_token
    hub.challenge
    Are all from facebook. We'll discuss soon.
    '''
    def get(self, request, *args, **kwargs):
        hub_mode   = request.GET.get('hub.mode')
        hub_token = request.GET.get('hub.verify_token')
        hub_challenge = request.GET.get('hub.challenge')
        if hub_token != VERIFY_TOKEN:
            return HttpResponse('Error, invalid token', status_code=403)
        return HttpResponse(hub_challenge)
            

    def post(self, request, *args, **kwargs):
        incoming_message = json.loads(request.body.decode('utf-8'))
        for entry in incoming_message['entry']:
            for message in entry['messaging']:
                if 'message' in message:
                    fb_user_id = message['sender']['id'] # sweet!
                    fb_user_txt = message['message'].get('text')
                    if fb_user_txt:
                        parse_and_send_fb_message(fb_user_id, fb_user_txt)
        return HttpResponse("Success", status=200)
```


```python
# bot.urls.py

from django.urls import re_path
from django.contrib import admin

from .views import (
    FacebookWebhookView
    )

app_name ='bot_webhooks'
urlpatterns = [
    re_path(r'^<webhook_endpoint>/$', FacebookWebhookView.as_view(), name='webhook'),
]

# replace <string_endpoint> with the one you created above
```

### Setup Facebook

1. Go to [https://developers.facebook.com/apps/](https://developers.facebook.com/apps/) (create account if needed)

2. Create a new app

3. Select to setup a Messenger App

4. Go to `Token Generation` for an API Testing token. Select a page or create a new one, generate the token, copy the token. Add this to the code above as `PAGE_ACCESS_TOKEN`.

5. In your app, go to `Settings` > `Advanced` and look for the numbers in `Upgrade API Version`. Mine say `v2.12`. That's where the `FB_ENDPOINT` came from.

> Now is the time to bring your Django project, with the new webhook, live.

6. Setup webooks.
- `https://www.yourdomain.com/api/fb/webhook/<webhook_endpoint>/` should now exist for real 
- In your app, go to `messenger` > `settings` > `webhooks` > `setup webhooks` 
    - Add in your entire url endpoint for the webhook such as `https://www.yourdomain.com/api/fb/webhook/<webhook_endpoint>/`
    - Add in the VERIFY_TOKEN created set above.
    - You can select all `Subscription Fields` or the ones you want. We chose to catch all. 

That should be it! We'll have to run a few tests to make sure your Django Project can now respond to Facebook messages.
