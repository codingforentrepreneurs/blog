---
title: Python Tutorial - Google Geocoding API
slug: python-tutorial-google-geocoding-api

publish_timestamp: Aug. 20, 2019
url: https://www.codingforentrepreneurs.com/blog/python-tutorial-google-geocoding-api/

---


In this short tutorial, we're going to show you how to do API calls to the Google Geocoding API. The purpose is to be able to quickly and easily, grab data from an address, zipcode, or just a city with an API that is a critical piece of Google Maps.

The more we use third party APIs, the better our services can be. The Geocoding API can clean up our address data with very little effort.

#### Installs
We're going to use [python requests](https://pypi.org/project/requests/) to make doing our API calls much easier. 
```
pip install requests
```

#### Google Geocoding API

Keep in mind that Google's Geocoding API has a number of free requests but after a certain point, they start to charge you. 

1. Sign up on [Google Cloud Compute (GCP)](https://cloud.google.com)
2. Login to [the console](https://console.developers.google.com).
3. Go to the Geocoding API [here](https://console.cloud.google.com/marketplace/details/google/geocoding-backend.googleapis.com) click "Enable". If that link fails, do a search for *Geocoding API*, open it, and click "Enable"

> You might have to create a project and do a few other configuration to get to the point where you can activate the Geocoding API

4. Get your Google Cloud API Credentials:
    - Do a search for "Credentials" within GCP
    - Click "Create Credentials" > "API Key"
    - Copy the key value, mine was `AIzaSyBSXMpu6lqd8kViIpy1GNWQ1symTXdMRzw` this is your Google Cloud API key. 
   
> **Google Cloud Compute API Keys** have unrestricted privileges so it's highly recommend that you restrict the API key to the Geocoding API as well as to your local IP address.


#### The Code
Below we're going to extract the latitude and longitude from any given address, zipcode, or city. It's really simple and easy.


```python
GOOGLE_API_KEY = 'your_api_key_from_above' 

def extract_lat_long_via_address(address_or_zipcode):
    lat, lng = None, None
    api_key = GOOGLE_API_KEY
    base_url = "https://maps.googleapis.com/maps/api/geocode/json"
    endpoint = f"{base_url}?address={address_or_zipcode}&key={api_key}"
    # see how our endpoint includes our API key? Yes this is yet another reason to restrict the key
    r = requests.get(endpoint)
    if r.status_code not in range(200, 299):
        return None, None
    try:
        '''
        This try block incase any of our inputs are invalid. This is done instead
        of actually writing out handlers for all kinds of responses.
        '''
        results = r.json()['results'][0]
        lat = results['geometry']['location']['lat']
        lng = results['geometry']['location']['lng']
    except:
        pass
    return lat, lng
```

> Interested in seeing this implemented in Pandas? Check out [this post](https://www.codingforentrepreneurs.com/blog/api-calls-pandas-dataframe-pd-apply)


Do you have a challenge that needs to be solved? Please submit your ideas on [/suggest](/suggest).
