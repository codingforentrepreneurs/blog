---
title: Mailchimp Integration
slug: mailchimp-integration

publish_timestamp: Oct. 17, 2017
url: https://www.codingforentrepreneurs.com/blog/mailchimp-integration/

---

The Mailchimp API is extensive. In this post, we're going to be building a basic, yet effective, way to integrate the API to our Python project(s). This concept is covered in more detail in the eCommerce course [here](https://www.codingforentrepreneurs.com/courses/ecommerce/mailchimp-integration/).

Below is done using Django but can be done in any Python project.


### 1. Sign up on [Mailchimp](http://eepurl.com/cVNKAv)


### 2. Get your API Key and Data Center Location

```
MAILCHIMP_API_KEY           = '330ede5b430f6a550a6ab58002a8dc20-us17' 
# the format is '{api_key_base}-{data_center_loc}'
MAILCHIMP_DATA_CENTER       = 'us17'
```


### 3. Get LIST API ID
1. Navigate to [mailchimp.com/lists/](http://admin.mailchimp.com/lists/)

2. Select a List

3. Go to "Settings" > "List name and defaults"

4. Grab the "List ID" 


### 4. Update `settings.py` with your data:


```
MAILCHIMP_API_KEY           = "330ede5b430f6a550a6ab58002a8dc20-us17"
MAILCHIMP_DATA_CENTER       = 'us17'
MAILCHIMP_EMAIL_LIST_ID     = 'e2ef12efee'
```


### 5. Create a `Mailchimp` object for working with the API.

First, run:

`pip install requests`


We're placing this in our Django app `marketing` with the name `utils.py`:

```
# marketing.utils.py

import hashlib
import re
import json
import requests
from django.conf import settings


MAILCHIMP_API_KEY = getattr(settings, "MAILCHIMP_API_KEY", None)
if MAILCHIMP_API_KEY is None:
    raise NotImplementedError("MAILCHIMP_API_KEY must be set in the settings")

MAILCHIMP_DATA_CENTER = getattr(settings, "MAILCHIMP_DATA_CENTER", None)

if MAILCHIMP_DATA_CENTER is None:
    raise NotImplementedError("MAILCHIMP_DATA_CENTER must be set in the settings, something like us17")

MAILCHIMP_EMAIL_LIST_ID = getattr(settings, "MAILCHIMP_EMAIL_LIST_ID", None)

if MAILCHIMP_EMAIL_LIST_ID is None:
    raise NotImplementedError("MAILCHIMP_EMAIL_LIST_ID must be set in the settings, something like us17")



def check_email(email):
    if not re.match(r".+@.+\..+", email):
        raise ValueError('String passed is not a valid email address')
    return


def get_subscriber_hash(member_email):
    '''
    This makes a email hash which is required by the Mailchimp API
    '''
    check_email(member_email)
    member_email = member_email.lower().encode()
    m = hashlib.md5(member_email)
    return m.hexdigest()


class Mailchimp(object):
    def __init__(self):
        super(Mailchimp, self).__init__()
        self.key = MAILCHIMP_API_KEY
        self.api_url  = 'https://{dc}.api.mailchimp.com/3.0'.format(
                                    dc=MAILCHIMP_DATA_CENTER
                                )
        self.list_id = MAILCHIMP_EMAIL_LIST_ID
        self.list_endpoint = '{api_url}/lists/{list_id}'.format(
                                    api_url = self.api_url,
                                    list_id=self.list_id
                        )


    def get_members_endpoint(self):
        endpoint = '{list_endpoint}/members'.format(
                                    list_endpoint=self.list_endpoint)
        return endpoint

    def add_email(self, email):
        check_email(email)
        endpoint = self.get_members_endpoint()
        data = { 
                "email_address": email, 
                "status": "subscribed"
                }
        r = requests.post(endpoint, 
                    auth=("", MAILCHIMP_API_KEY), 
                    data=json.dumps(data)
                    )
        return r.status_code, r.json()


    def check_valid_status(self, status):
        choices = ['subscribed','unsubscribed', 'cleaned', 'pending']
        if status not in choices:
            raise ValueError("Not a valid choice")
        return status

    def change_subscription_status(self, email, status):
        subscriber_hash     = get_subscriber_hash(email)
        members_endpoint       = self.get_members_endpoint()
        endpoint            = "{members_endpoint}/{sub_hash}".format(
                                members_endpoint=members_endpoint, 
                                sub_hash=subscriber_hash
                                )
        data                = {
                                "status": self.check_valid_status(status)
                            }
        r                   = requests.put(endpoint, 
                                auth=("", MAILCHIMP_API_KEY), 
                                data=json.dumps(data)
                                )
        return r.status_code, r.json()

    def check_subscription_status(self, email):
        subscriber_hash     = get_subscriber_hash(email)
        members_endpoint       = self.get_members_endpoint()
        endpoint            = "{members_endpoint}/{sub_hash}".format(
                                members_endpoint=members_endpoint, 
                                sub_hash=subscriber_hash
                                )
        r                   = requests.get(endpoint, 
                                auth=("", MAILCHIMP_API_KEY)
                                )
        return r.status_code, r.json()


    def unsubscribe(self, email):
        return self.change_subscription_status(email, status='unsubscribed') 


    def resubscribe(self, email):
        return = self.change_subscription_status(email, status='subscribed')


```


### 6. Ready to roll!
Now you can use the above to Subscribe, Unsubscribe, Resubscribe, and check the subscription status of any email for your users.

Be sure to check out the eCommerce course [here](https://www.codingforentrepreneurs.com/courses/ecommerce/mailchimp-integration/) for more details on how this all works.