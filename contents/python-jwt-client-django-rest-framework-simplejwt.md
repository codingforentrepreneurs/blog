---
title: A Python JWT Client for Django Rest Framework simplejwt
slug: python-jwt-client-django-rest-framework-simplejwt

publish_timestamp: Feb. 21, 2022
url: //www.desalsa.io:8000/blog/python-jwt-client-django-rest-framework-simplejwt/

---

In this blog post, we&#x27;ll look at an example Python client for authentication via JSON Web Tokens (JWT). The code below is meant to be used as a snippet you can modify as you need. If you have questions or comments, please write them below.

If you want full context on how to use this code, consider watching our [Django Rest Framework tutorial series](https://www.codingforentrepreneurs.com/projects/django-rest-framework-2022/).

This snippet was made for using the `django-rest-framework-simplejwt` Authentication backend that we implement at the end of the above tutorial series. If you want to learn more about that package, consider reading their [docs](https://django-rest-framework-simplejwt.readthedocs.io/).

A few key assumptions:

- You have Django Rest Framework setup on a Django Project (or you can [clone this one](https://github.com/codingforentrepreneurs/Django-Rest-Framework-Tutorial))
- You know how to use basic [Token Authentication](https://www.codingforentrepreneurs.com/projects/django-rest-framework-2022/token-authentication) with Django Rest Framework and the [Python Requests](https://docs.python-requests.org/en/latest/) package.
- Storing something like `creds.json` (as we do below) 100% has **security holes** and should be used only as you learn.

The endpoints I will be using are as follows:

- `https://localhost:8000/api/token/` mapped to the `TokenObtainPairView`
- `https://localhost:8000/api/token/refresh/` mapped to the `TokenRefreshView`
- `https://localhost:8000/api/token/verify/` mapped to the `TokenVerifyView`
- `https://localhost:8000/api/products/` mapped to my Django Rest Framework generic ListAPIView that with the pagination class LimitPageOffsetPagination.

Below is what the respective `urls.py` looks like:

```python
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenVerifyView,
)

from products.views import product_list_create_view

urlpatterns = [
    ...
    path(&#x27;api/token/&#x27;, TokenObtainPairView.as_view(), name=&#x27;token_obtain_pair&#x27;),
    path(&#x27;api/token/refresh/&#x27;, TokenRefreshView.as_view(), name=&#x27;token_refresh&#x27;),
    path(&#x27;api/token/verify/&#x27;, TokenVerifyView.as_view(), name=&#x27;token_verify&#x27;),
    path(&#x27;api/products/&#x27;, product_list_create_view, name=&#x27;product-list&#x27;),
]
```
&gt; For simplicity of this post, this `urls.py` *does not exactly match* how the urls are written in the [Django Rest Framework](https://www.codingforentrepreneurs.com/projects/django-rest-framework-2022/) tutorial or [github repo](https://github.com/codingforentrepreneurs/Django-Rest-Framework-Tutorial). This post does, however, match how they are *configured* (ie the same url paths point to the same views).


Here&#x27;s the client:

```python
from dataclasses import dataclass
import requests
from getpass import getpass
import pathlib 
import json


@dataclass
class JWTClient:
    &quot;&quot;&quot;
    Use a dataclass decorator
    to simply the class construction
    &quot;&quot;&quot;
    access:str = None
    refresh:str = None
    # ensure this matches your simplejwt config
    header_type: str = &quot;Bearer&quot;
    # this assumesy ou have DRF running on localhost:8000
    base_endpoint = &quot;http://localhost:8000/api&quot;
    # this file path is insecure
    cred_path: pathlib.Path = pathlib.Path(&quot;creds.json&quot;)

    def __post_init__(self):
        if self.cred_path.exists(): 
            &quot;&quot;&quot;
            You have stored creds,
            let&#x27;s verify them
            and refresh them.
            If that fails,
            restart login process.
            &quot;&quot;&quot;
            try:
                data = json.loads(self.cred_path.read_text())
            except Exception:
                print(&quot;Assuming creds has been tampered with&quot;)
                data = None
            if data is None:
                &quot;&quot;&quot; 
                Clear stored creds and
                Run login process
                &quot;&quot;&quot;
                self.clear_tokens()
                self.perform_auth()
            else:
                &quot;&quot;&quot;
                `creds.json` was not tampered with
                Verify token -&gt; 
                if necessary, Refresh token -&gt;
                if necessary, Run login process
                &quot;&quot;&quot;
                self.access = data.get(&#x27;access&#x27;)
                self.refresh = data.get(&#x27;refresh&#x27;)
                token_verified = self.verify_token()
                if not token_verified:
                    &quot;&quot;&quot;
                    This can mean the token has expired
                    or is invalid. Either way, attempt
                    a refresh.
                    &quot;&quot;&quot;
                    refreshed = self.perform_refresh()
                    if not refreshed:
                        &quot;&quot;&quot;
                        This means the token refresh
                        also failed. Run login process
                        &quot;&quot;&quot;
                        print(&quot;invalid data, login again.&quot;)
                        self.clear_tokens()
                        self.perform_auth()
        else:
            &quot;&quot;&quot;
            Run login process
            &quot;&quot;&quot;
            self.perform_auth()
        
    def get_headers(self, header_type=None):
        &quot;&quot;&quot;
        Default headers for HTTP requests
        including the JWT token
        &quot;&quot;&quot;
        _type = header_type or self.header_type
        token = self.access
        if not token:
            return {}
        return {
                &quot;Authorization&quot;: f&quot;{_type} {token}&quot;
        }

    def perform_auth(self):
        &quot;&quot;&quot;
        Simple way to perform authentication
        Without exposing password(s) during the
        collection process.
        &quot;&quot;&quot;
        endpoint = f&quot;{self.base_endpoint}/token/&quot; 
        username = input(&quot;What is your username?\n&quot;)
        password = getpass(&quot;What is your password?\n&quot;)
        r = requests.post(endpoint, json={&#x27;username&#x27;: username, &#x27;password&#x27;: password}) 
        if r.status_code != 200:
            raise Exception(f&quot;Access not granted: {r.text}&quot;)
        print(&#x27;access granted&#x27;)
        self.write_creds(r.json())

    def write_creds(self, data:dict):
        &quot;&quot;&quot;
        Store credentials as a local file
        and update instance with correct
        data.
        &quot;&quot;&quot;
        if self.cred_path is not None:
            self.access = data.get(&#x27;access&#x27;)
            self.refresh = data.get(&#x27;refresh&#x27;)
            if self.access and self.refresh:
                self.cred_path.write_text(json.dumps(data))
    
    def verify_token(self):
        &quot;&quot;&quot;
        Simple method for verifying your
        token data. This method only verifies
        your `access` token. A 200 HTTP status
        means success, anything else means failure.
        &quot;&quot;&quot;
        data = {
            &quot;token&quot;: f&quot;{self.access}&quot;
        }
        endpoint = f&quot;{self.base_endpoint}/token/verify/&quot; 
        r = requests.post(endpoint, json=data)
        return r.status_code == 200
    
    def clear_tokens(self):
        &quot;&quot;&quot;
        Remove any/all JWT token data
        from instance as well as stored
        creds file.
        &quot;&quot;&quot;
        self.access = None
        self.refresh = None
        if self.cred_path.exists():
            self.cred_path.unlink()
    
    def perform_refresh(self):
        &quot;&quot;&quot;
        Refresh the access token by using the correct
        auth headers and the refresh token.
        &quot;&quot;&quot;
        print(&quot;Refreshing token.&quot;)
        headers = self.get_headers()
        data = {
            &quot;refresh&quot;: f&quot;{self.refresh}&quot;
        }
        endpoint = f&quot;{self.base_endpoint}/token/refresh/&quot; 
        r = requests.post(endpoint, json=data, headers=headers)
        if r.status_code != 200:
            self.clear_tokens()
            return False
        refresh_data = r.json()
        if not &#x27;access&#x27; in refresh_data:
            self.clear_tokens()
            return False
        stored_data = {
            &#x27;access&#x27;: refresh_data.get(&#x27;access&#x27;),
            &#x27;refresh&#x27;: self.refresh
        }
        self.write_creds(stored_data)
        return True

    def list(self, endpoint=None, limit=3):
        &quot;&quot;&quot;
        Here is an actual api call to a DRF
        View that requires our simplejwt Authentication
        Working correctly.
        &quot;&quot;&quot;
        headers = self.get_headers()
        if endpoint is None or self.base_endpoint not in str(endpoint):
            endpoint = f&quot;{self.base_endpoint}/products/?limit={limit}&quot; 
        r = requests.get(endpoint, headers=headers) 
        if r.status_code != 200:
            raise Exception(f&quot;Request not complete {r.text}&quot;)
        data = r.json()
        return data


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    Here&#x27;s Simple example of how to use our client above.
    &quot;&quot;&quot;
    
    # this will either prompt a login process
    # or just run with current stored data
    client = JWTClient() 

    # simple instance method to perform an HTTP
    # request to our /api/products/ endpoint
    lookup_1_data = client.list(limit=5)
    # We used pagination at our endpoint so we have:
    results = lookup_1_data.get(&#x27;results&#x27;)
    next_url = lookup_1_data.get(&#x27;next&#x27;)
    print(&quot;First lookup result length&quot;, len(results))
    if next_url:
        lookup_2_data = client.list(endpoint=next_url)
        results += lookup_2_data.get(&#x27;results&#x27;)
        print(&quot;Second lookup result length&quot;, len(results))
```

Do you have ways to improve? Please comment below and use [Markdown format](https://www.codingforentrepreneurs.com/blog/markdown-cheatsheet/) when pasting code.