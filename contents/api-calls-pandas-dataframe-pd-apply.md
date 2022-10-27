---
title: API Calls within a Pandas Dataframe API using Pandas Apply
slug: api-calls-pandas-dataframe-pd-apply

publish_timestamp: Aug. 6, 2019
url: https://www.codingforentrepreneurs.com/blog/api-calls-pandas-dataframe-pd-apply/

---

Whenever you're doing data analysis in Python your most likely using [Pandas](https://pandas.pydata.org/) since it's widely used and, most importantly, one of the best tools available for data analytics. Even better, pandas is 100% open-source.

Pandas is only as good as the data it is given. Good data in, good data out. Garbage in, garbage out. That's the ethos of many data scientists around the globe.

Since we have this amazing tool in Pandas, it's time we starting using another concept that you've probably heard of: APIs.

Many APIs allow applications to automatically talk to one another, exchange data, and do so in a highly scalable and automated way.

This post is going to show you a basic example of how you can use the [pandas apply](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.apply.html) method to enrich a dataframe with API data. 


A few times you might want to use an external API

- Calculate shipping costs with Fedex/UPS/etc between your warehouse and your customer
- Extract Latitude and Longitude from a street address or zipcode
- Trigger an automation in Zapier
- Email a customer an order status update

Really, the list is endless. We're going to be focusing on integrating 1 API.

> Running through this post is **10x** better if you use a [jupyter notebook](https://www.codingforentrepreneurs.com/blog/jupyter-notebook-server-aws-ec2-aws-vpc), you can find this post's notebook [here](https://github.com/codingforentrepreneurs/Notebooks/blob/master/src/API%20Calls%20within%20a%20Pandas%20Dataframe%20API%20using%20Pandas%20Apply.ipynb).

### 1. The Google Geocoding API

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

### 2. Install Requirements
```
pip install pandas requests
```

### 3. Let's code


```python
import datetime
import pandas as pd
import requests
```


```python
# our dataset
data = {"addresses": ['Newport Beach, California', 'New York City', 'London, England', 10001, 'Sydney, Au']}
```


```python
# Calling DataFrame constructor on addresses list 
df = pd.DataFrame(data) 
df 
```

We only have 1 data point here. But what we're about to do can work for nearly any number of columns in a similar way.

Let's take a look at how the [pandas apply](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.apply.html) works with a simple example.


```python
# create a throw-away dataframe
df_throwaway = df.copy()

def apply_this_function(passed_row):
    passed_row['new_col'] = True
    passed_row['added'] = datetime.datetime.now()
    return passed_row

df_throwaway.apply(apply_this_function, axis=1) # axis=1 is important to use the row itself
```

As we see, the [pandas apply](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.apply.html) function works really well to add additional columns to our current columns. But we can go one step further, we can actually create new columns and row values based on what's within the row itself.


```python
# create another throw-away dataframe
df_example_2 = df.copy()

def apply_this_other_function(row):
    column_name = 'addresses'
    address_value = row[column_name]
    if isinstance(address_value, int):
        row[column_name] = address_value * 2
    return row

df_example_2.apply(apply_this_other_function, axis=1) # axis=1 is important to use the row itself
```

Now we see two main things about the `.apply` method, in each row we can:

1. Add new columns based on other column's values (within that row)
2. We can change the any value of any column within a single row)

In other words, `.apply` enables us to change rows very dynamically. Now it's time to call our Geocoding API.


```python
# create a working example. I like using a copy of the source data in case we make mistakes
rest_api_df = df.copy()
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
    
def enrich_with_geocoding_api(row):
    column_name = 'addresses'
    address_value = row[column_name]
    address_lat, address_lng = extract_lat_long_via_address(address_value)
    row['lat'] = address_lat
    row['lng'] = address_lng
    return row

rest_api_df.apply(enrich_with_geocoding_api, axis=1) # axis=1 is important to use the row itself
```

That's it. Pretty simple right? It's also very useful when you get into doing a lot of different API calls within your pandas workflows.