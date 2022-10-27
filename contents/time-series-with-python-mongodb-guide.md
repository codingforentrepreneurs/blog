---
title: Time Series with Python &amp; MongoDB Guide
slug: time-series-with-python-mongodb-guide

publish_timestamp: Sept. 19, 2022
url: https://www.codingforentrepreneurs.com/blog/time-series-with-python-mongodb-guide/

---


In this blog post, we're going to uncover how to use Time Series data with Python and MongoDB. If you want to learn from a detailed video series, please consider enrolling in [this course](https://www.codingforentrepreneurs.com/courses/time-series-with-python-mongodb/). 

Time series data is incredibly compelling and can help us make better decisions throughout our projects and our organizations.

Let's get started!

### Prerequisites
- Python 3.8+ Installed
- Docker & Docker Compose installed or simply [Docker Desktop](https://www.docker.com/products/docker-desktop/).
- Terminal or PowerShell experience



## Step 1: Create Project Directory

1. Open Terminal/PowerShell
2. Create a directory for all projects:

```bash
mkdir -p ~/Dev
```
3. Create your project directory
```bash
mkdir -p ~/Dev/ts-pymongo
```
4. Setup VS Code Workspace (optional)
```bash
echo "" > ~/Dev/ts-pymongo/ts-pymongo.workspace
```
In `~/Dev/ts-pymongo/ts-pymongo.workspace` add:
```json
{
  "folders": [
    {
      "path": "."
    }
  ],
  "settings": {}
}
```
Side note: you can always perform this step within VS Code as well.



## Step 2: Docker Compose Configuration
In this step, we're going to use a local instance of MongoDB by leveraging Docker. The only reason to do it this way is to *learn* how to leverage MongoDB in your projects. Once you learn, using a managed MongoDB database is *highly* recommended so be sure to check out [MongoDB on Linode](https://www.linode.com/products/mongodb/) when you're ready for production.

Assuming you have [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and at least one of the following commands work:

- `docker compose version`
- `docker-compose version`

If either of the above commands do *not* work, consider using a managed instance on Linode right now.


*Docker Compose configuration*

In `~/Dev/ts-pymongo/`, we'll add the following:

- `src/.env`
- `docker-compose.yaml`

First, `~/Dev/ts-pymongo/docker-compose.yaml`:
```yaml
version: '3.9'

services:
  mongo:
    image: mongo
    restart: always
    ports: 
      - 27017:27017
    env_file: ./src/.env
```
Simple enough right? Let's add our environment variables for this to work:

```bash
mkdir -p ~/Dev/ts-pymongo/src
echo "" > ~/Dev/ts-pymongo/src/.env
```

In  `~/Dev/ts-pymongo/src/.env`, add:

```bash
MONGO_INITDB_ROOT_USERNAME="root"
MONGO_INITDB_ROOT_PASSWORD="HX3vApmHj5or0NIBp1cZTUi10Vr7Hq1HMIGC4birYZI"
```

I create passwords like this with: `python -c "import secrets;print(secrets.token_urlsafe(32))"`

*Docker & MongoDB Environment Variables*
When spinning up MongoDB in Docker or Docker compose we need to set the following enviroNment variables:
- `MONGO_INITDB_ROOT_USERNAME`
- `MONGO_INITDB_ROOT_PASSWORD` 

This is defined in the Official MongoDB Docker Image on [DockerHub](https://hub.docker.com/_/mongo/).

*Docker Compose Commands*

There are 3 important commands we need to know for this project:

- `docker compose up -d`: runs the database in the background
- `docker compose down`: turns off the database; keeps data
- `docker compose down -v`: removes the database; deletes data

It's true there's a *lot* more to Docker and Docker Compose than this but we'll save those details for another time.



## Step 3: Create your Python Virtual Environment
Isolate your python projects by leveraging virtual environments. There are many ways to accomplish this but we'll use the built-in python package [venv](https://docs.python.org/3/tutorial/venv.html).

I recommend you use the Python distribution directly from [python.org](https://www.python.org/downloads/). I do not recommend you use Anaconda, mini-conda, or any other Python distribution. Unofficial distributions, like Anaconda, can cause third-party dependency issues that are hard to diagnose which is why we're going to leave it out for now. 

### tldr

```bash
mkdir -p ~/Dev/ts-pymongo/
cd ~/Dev/ts-pymongo/
python3.10 -m venv venv
```
If on Windows, use `C:\Python310\python.exe` instead of `python3.10`.

For a more detailed virtual environment creation, check the sections below. If the *tldr* above works for you, skip to Step 3.



### macOS/Linux Virtual Environment Creation
```bash
cd ~/Dev/ts-pymongo/
python3.10 -m venv venv
```
Notice that we used `venv` twice? The first `-m venv` is calling the python module. The second `venv` is what we're naming our virtual environment and `venv` is a conventional name.

*Activate it*
```bash
source venv/bin/activate
```

*Update pip*
```bash
$(venv) python -m pip install pip --upgrade
```
Using `python -m pip` is the recommended method of handling pip installations. Using `pip` without `python -m` might cause system-wide dependency issues.

*Deactivate and Reactivate*
```bash
$(venv) deactivate
source venv/bin/activate
```

*Activation-less Commands*
If you prefer to not activate your virtual environment, that's okay. You'll just need to leverage `venv/bin/` and related items.

So you can use the virtual environment python with:

```bash
venv/bin/python --version
```

Or, for example, if you're using the Python package  `uvicorn`, you can call:
```bash
venv/bin/uvicorn
```

You can also use paths like:
```bash
~/Dev/ts-pymongo/venv/bin/uvicorn
```

or
```bash
/Users/cfe/Dev/ts-pymongo/venv/bin/uvicorn
```
Replace `cfe` as your username of course.

Now you're ready for the next step.

### Windows Virtual Environment Creation
```bash
cd ~/Dev/ts-pymongo/
C:\Python310\python.exe -m venv venv
```
`C:\Python310\python.exe` assumes this is the location where you installed `Python.3.10`. If you did not install it here, you'll need to locate the exact place you did install it before you create your virtual environment. 

One way to do that is to open up PowerShell (after you installed Python) and run:

```bash
python
```
This will either cause an error or it will enter the Python shell.

```python
import os
import sys
print(os.path.dirname(sys.executable))
```
This will yield something like:
```
C:\Python38
```
Whatever path `os.path.dirname(sys.executable)` yields you'll have to add `python.exe` to it in order to use python.

For example, above yielded `C:\Python38` and not `C:\Python310`. In that case you would use:

```bash
C:\Python38\python.exe -m venv venv
```
To create your virtual environment.

Notice that we used `venv` twice? The first `-m venv` is calling the python module. The second `venv` is what we're naming our virtual environment and `venv` is a conventional name.

*Activate it*
```bash
./venv/Scripts/activate
```
This will yield something like:

```bash
(venv) PS C:\Users\cfe\Dev\ts-pymongo>
```
We'll use `$(venv)` to denote an activated virtual environment going forward.

*Update pip*
```bash
$(venv) python -m pip install pip --upgrade
```
Using `python -m pip` is the recommended method of handling pip installations. Using `pip` without `python -m` might cause system-wide dependency issues.

*Deactivate and Reactivate*
```bash
$(venv) deactivate
./venv/Scripts/activate
```

*Activation-less Commands*
If you prefer to not activate your virtual environment, that's okay. You'll just need to leverage `venv/bin/` and related items.

So you can use the virtual environment python with:

```bash
venv/Scripts/python --version
```
Or, for example, if you're using the Python package `uvicorn`, you can call:

```bash
venv/Scripts/uvicorn
```
You can also use paths like:
```bash
~/Dev/ts-pymongo/venv/Scripts/uvicorn
```
or
```powershell
C:\Users\cfe\Dev\ts-pymongo\venv\Scripts\uvicorn
```
Replace `cfe` with your username of course. Also, as you may know, PowerShell can use either forward slash `/` or backslash `\`.

Now you're ready for the next step.

## Step 4. Connect Python to MongoDB
At this point, we have the following available to us:

- A Python Virtual Environment
- A Running MongoDB instance
  
Now it's time to connect Python to MongoDB

### 1. Install Python Requirements

**Activate our virtual environment**

*macOS/Linux*
```bash
source venv/bin/activate
```

*Windows*
```powershell
.\venv\Scripts\activate
```

**Create requirements.txt**
```bash
echo "" > src/requirements.txt
echo "pymongo" >> src/requirements.txt
echo "python-decouple" >> src/requirements.txt
echo "pandas" >> src/requirements.txt
echo "matplotlib" >> src/requirements.txt
```
- `pymongo` ([Docs](https://pymongo.readthedocs.io/en/stable/)) is the primary package we'll use to connect Python with MongoDB. 
- `python-decouple`  ([Docs](https://github.com/henriquebastos/python-decouple/)) is a neat way to load `.env` secrets/configuration into our Python project.
- `pandas`  ([Docs](https://pandas.pydata.org/docs/)) is powerful way to work with datasets. We'll use it for exporting our time series plots.
- `matplotlib`. ([Docs](https://matplotlib.org/)) Pandas requires `matplotlib` to create charts and graphs.

**Install requirements**
```bash
$(venv) python -m pip install pip --upgrade
$(venv) python -m pip install -r src/requirements.txt
```

**Create `db_client.py`**
In `src/db_client.py` add:

```python
from functools import lru_cache

import decouple
from pymongo import MongoClient

@lru_cache
def get_db_client(host='localhost'):
    mongodb_un = decouple.config("MONGO_INITDB_ROOT_USERNAME")
    mongodb_pw = decouple.config("MONGO_INITDB_ROOT_PASSWORD")
    mongodb_host = decouple.config("MONGO_HOST", default='localhost')
    db_url =  f"mongodb://{mongodb_un}:{mongodb_pw}@{mongodb_host}:27017"
    return MongoClient(db_url)
```
Let's break this down:
- `lru_cache` makes calling `get_db_client` a little more efficient during the same session. In other words, `get_db_client` will only create 1 instance of `MongoClient` which is exactly what we want if we turn this into a full web application.
- `decouple.config` allows us to use our configuration from `.env`
- `get_db_client` creates a Python MongoDB Client that connects to our Docker-based MongoDB.
- `MongoClient` takes in the database url which ends up looking like `MongoClient("mongodb://username:password@host:port")`. MongoDB typically uses port `27017` which is why we have that port listed here. If you used a managed mongodb service, chances are good you will have to update the `host` by setting `MONGO_HOST` in `.env`.



## Step 5. CRUD with PyMongo

Here's the process to leveraging CRUD in PyMongo:

1. Connect to your MongoDB Client
2. Select a Database
3. Select a Collection
4. Use CRUD operations

Navigate in `src` and run the following:

```bash
cd src
python
```

**1. Connect to your MongoDB Client**
```python
import db_client
client = db_client.get_db_client()
```

**2. Select a Database**
```python
db = client.business
```

**3. Select a Collection**
```python
collection = db["ratings"]
```
Let's verify what's in this collection:

```python
list(collection.find())
```
If you just started this one, you should see `[]` as your response. 

Before we jump into the CRUD operations, let's break down what just happened.

- `client = db.get_db_client()` initializes a connection to MongoDB
- `db = client.business` declares a database to use
- `collection = db["ratings"]` declares a collection to use

If you're coming from SQL, you might be wondering:

- Where is the command to create the database?
- What is a collection? Is it a table?
- Where is the command to create the collection?
- When do we declare the fields we want to enforce in the collection?

A simple answer to these questions are: that's not how MongoDB works. Databases are groups of collections and collections of groups of **documents**.

These documents have incredible flexibility and can look a bit like this:

- `{"product": "sparkling water", "price": 1.99}`
- `{"location": "Austin, Texas", "ranch_living": True}`


Both of these documents resemble dictionaries in Python and objects in JavaScript; and that's exactly the point.

With documents, the field names (ie keys) do not matter to storing the document. As the developer, you *can* enforce rules for field names but it's not required.

I think this is pretty neat and leaves the option for adding new data (including nested dictionaries/objects) whenever you need to. 


**4. Add Data as a Document to a Collection**

This is beyond simple. Here's how it's done:

```python
client = db.get_db_client()
db = client.business
collection = db["ratings"]
data_document = {"name": "Torchy's Tacos", "location": "Austin, Texas", "rating": 4.5}
collection.insert_one(data_document)
```

Another way:

```python
db = client.business
rating_data = {"name": "Gourdough's Big. Fat. Donuts.", "location": "Austin, Texas", "rating": 5.0}db["ratings"].insert_one(rating_data)
```

**5. List Data from Collection**
Now that we added two documents, we can list them out in the following ways:

```python
db = client.business
list(db["ratings"].find())
```
or
```python
db = client.business
collection = db["ratings"]
list(collection.find())
```
the `.find()` method on a collection also allows for querying the data such as:

```python
for obj in collection.find({"name": "Torchy's Tacos"}):
    print(obj)
```

There is *so* much more to querying with MongoDB and PyMongo that needs it's own blog post. 

In the above `find()` methods, you should see data that is returned that looks something like:

```python
{'_id': ObjectId('62f2785383c78ae7fdf5c037'), 'name': "Torchy's Tacos", 'location': 'Austin, Texas', 'rating': 4.5}
```

**6. Get a Single Document from a Collection**

Looking back on our data, we see that `_id` is set with an `ObjectId()` class. This `_id` is unique to the document and it's something we can use for lookups (especially to edit or delete items).


First let's get a document using the `find_one` method on our collection:

```python
document_result = collection.find_one({"name": "Torchy's Tacos"})
document_result
```
> Notice that `find_one` in PyMongo maps to the `findOne` method in MongoDB ([docs](https://www.mongodb.com/docs/manual/reference/method/db.collection.findOne/)). Python uses snake_case so be sure to try snake_case if you ever find a method that doesn't seem to work.

In this case, our `document_result` could yield `None` if our query yields no results. It's important to note that querying in MongoDB can get really complex so we'll leave that for another time. Let's get back to getting the value of the `_id`.


```python
document_result = collection.find_one({"name": "Torchy's Tacos"})
object_id = None
if document_result is not None:
    object_id = document_result['_id']
print(object_id)
```
With the data our collection has, this should yield something like:

```
62f2785383c78ae7fdf5c037
```

Now that we have an Object Id for a document, let's look it up:
```python
from bson.objectid import ObjectId

if isinstance(object_id, str):
    object_id = ObjectId(object_id)
result = collection.find_one({"_id": object_id})
print(result)
```
The `isinstance` block helps us enfource that our `object_id` is of the class `ObjectId` much like what we have seen in our `document_result` above. If you use `object_id` as a string, your lookup will yield `None`:

```python
no_result = collection.find_one({"_id": str(object_id)})
print(no_result)
assert no_result == None
```

**7. Update a Single Document from a Collection**

First, let's start with our `object_id` since it's unique for any given document. 
```python
object_id = ObjectId(object_id)
```

Now, let's get new data:

```python
new_data = {"cuisine": "Mexican", "only_location": False, "total_visitor_count": 120_000}
```

To update, we're going to combine a query (in our case the `object_id`) as well as the data we want to change (using `$set`):

```python
query_filter = {"_id": object_id}
update_data = {"$set": new_data}
collection.update_one(query_filter, update_data)
```
Notice that we nested our `new_data` instead of another dictionary with the key `$set`. This will literally set the values within our document based on whatever is in `new_data`. You can use dot notation for setting data but that's outside the scope of what we're doing here.

Now we're going to increment a field in our data.

Above we set `total_visitor_count` to `120_000` (stored as `120000`). This is the same thing as replacing the original value with our new value.

But what if we wanted to do some math on this field? Let's see how it's done with the `$inc` operator:

*Add 500 visitors**
```python
increment_data = {"$inc": {"total_visitor_count": 500}}
collection.update_one(query_filter, update_data)
```
Or

*Subtract 293 by adding -293 visitors** 
```python
decrement_data = {"$inc": {"total_visitor_count": -293}}
collection.update_one(query_filter, update_data)
```

So both `$set` and `$inc` are incredibly useful for updating the data stored within a document. `$inc` is especially useful when you're dealing with storing integers or floats.  Read more about `$inc` [here](https://www.mongodb.com/docs/manual/reference/operator/update/inc/) and more about `$set` [here](https://www.mongodb.com/docs/manual/reference/operator/update/set/) as well as all other filed update operators [here](https://www.mongodb.com/docs/manual/reference/operator/update-field/).



**7. Delete a Single Document from a Collection**

Let's create, update, and delete on this one:

*Setup*
```python
client = db.get_db_client()
db = client.business
collection = db["ratings"]
```

*Create*
```python
data = {'name': "CFE Tacos", 'location': 'Austin, Texas', 'rating': 5}
result = collection.insert_one(data)
object_id = result.inserted_id
object_id
```

*Update*
```python
data = {'name': "Just-in-Time Tacos"}
collection.update_one({"_id": object_id}, {"$set": data})
new_visitors_data = {'visitors': 300_000}
collection.update_one({"_id": object_id}, {"$inc": new_visitors_data})
new_visitors_data_again = {'visitors': 320_120}
collection.update_one({"_id": object_id}, {"$inc": new_visitors_data_again})
```

*List Matching Data*
```python
list(collection.find({"name": "Just-in-Time Tacos"}))
```

*Retrieve New Data*
```python
stored_result = collection.find_one({"_id": object_id})
print(stored_result)
```
Yields: `{'_id': ObjectId('62f28b6783c78ae7fdf5c03c'), 'name': 'Just-in-Time Tacos', 'location': 'Austin, Texas', 'rating': 5, 'visitors': 620120}`

*Delete Listing*
```python
delete_result = collection.delete_one({"_id": object_id})
print(delete_result.deleted_count, delete_result.acknowledged, delete_result.raw_result)
```


Now that we understand some of the basics of MongoDB, let's start using Time Series data.



## Step 6. Generate Time Series Collection & Data

Let's create a new collection that's designed for time series ([docs](https://www.mongodb.com/docs/manual/core/timeseries-collections/)):

We're going to add a new time series by using the `create_collection` method like this:

```python
name = "my_collection_name"
db.create_collection(
        name,
        timeseries= {
            "timeField": "timestamp",
            "metaField": "metadata",
            "granularity": "seconds"
        }
    )
```

Let's add it to our `src` folder.

In `src/collection_create.py` add::
```python
from pymongo import errors

import db_client # created above

def create_ts(name='rating_over_time'):
    """
    Create a new time series collection
    """
    client = db_client.get_db_client()
    db = client.business
    try:
        db.create_collection(
               name,
                timeseries= {
                    "timeField": "timestamp",
                    "metaField": "metadata",
                    "granularity": "seconds"
                }
            )
    except errors.CollectionInvalid as e:
        print(f"{e}. Continuing")

def drop(name='rating_over_time'):
    """
    Drop any given collection by name
    """
    client = db_client.get_db_client()
    db = client.business
    try:
        db.drop_collection(name)
    except errors.CollectionInvalid as e:
        print(f"Collection error:\n {e}")
        raise Exception("Cannot continue")
```
We now have convenient `create_ts` and `drop` methods that we'll use in the `generate_data.py` module create next.

Let's breakdown what's happening in the `create_ts` function:

- `db.create_collection` is a command that allows us to create a collection. In this case, we are included the required parameters to create a time series collection.
- `rating_over_time` is simply the name of the collection.
- `timeseries={}` is the parameter we must add for time series data.
  - `timeField` is the field name we'll set with a python `datetime` object
  - `metaField` is we'll use this for unique identifies for this document that rarely (if ever) change. Examples we'll use are restaurant name and cuisine type.
  - `granularity` options are `"seconds"`, `"minutes"`, or `"hours"`.  This is how the data will be stored (along with the timestamp). Ideally ou "choose the closest match to the time span between consecutive incoming measurements."
  - `errors.CollectionInvalid` is the exception that will be raised if `rating_over_time` already exists.

Given this criteria, we'll have following formatted data document inserted into our collection at any given time. The key to this is not change the field we have below. A few of them we assigned directly to parameters in `timeseries={}` above.

```python
data = {
  "metadata": {
    "name": "Some String",
    "cuisine": "Another String"
  },
  "rating": 4.5,
  "timestamp": datetime.datetime.now()
}
```


**Generate Random Data**
Now that we can create a time-series collection, let's build a module that will generate random data for us. This data will be generated randomly (really pseudo-random via the [random](https://docs.python.org/3/library/random.html) package) by the functionality below.


In `src/generate_data.py` let's add the following:

```python
import datetime
import random
import sys

import collection_create
import db_client



name_choices =  ["Big", "Goat", "Chicken", "Tasty", "Salty", "Fire", "Forest", "Moon", "State", "Texas", "Bear", "California"]
cuisine_choices = ["Pizza", "Bar Food", "Fast Food", "Pasta","Tacos", "Sushi", "Vegetarian", "Steak", "Burgers"]


def get_random_name():
    _name_start = random.choice(name_choices)
    _name_end = random.choice(name_choices)
    return f"{_name_start} {_name_end}"

def get_random_cuisine():
    selected_cuisine = random.choice(cuisine_choices)
    return selected_cuisine
  
def get_random_rating(skew_low=True):
    part_a = list([random.randint(1, 3) for i in range(10)])
    part_b = list([random.randint(3, 4) for i in range(10)])
    if not skew_low:
      part_c = list([random.randint(4, 5) for i in range(25)])
    else:
      part_c = list([random.randint(4, 5) for i in range(5)])
    _ratings =  part_a + part_b +  part_c
    return random.choice(_ratings)

def get_random_timestamp():
    now = datetime.datetime.now()
    delta = now - datetime.timedelta(days=random.randint(0, 5_000), minutes=random.randint(0, 60), seconds=random.randint(0, 60))
    return delta

def get_or_generate_collection(name="rating_over_time"):
    client = db_client.get_db_client()
    db = client.business
    collection_create.create(name=name)
    collection = db[name]
    return collection

def run(collection, iterations=50, skew_results=True):
    completed = 0
    for n in range(0, iterations):
        timestamp = get_random_timestamp()
        name = get_random_name()
        cuisine = get_random_cuisine()
        rating = get_random_rating(skew_low=True)
        if skew_results:
          if cuisine.lower() == "mexican":
              rating = random.choice([4, 5])
          elif cuisine.lower() == "bar food":
              rating = random.choice([1, 2])
          elif cuisine.lower() == "sushi":
              rating = get_random_rating(skew_low=False)              
        data = {
          "metadata": {
            "name": name,
            "cuisine": cuisine
          },
          "rating": rating,
          "timestamp": timestamp
        }
        result = collection.insert_one(data)
        if result.acknowledged:
            completed += 1
        if n > 0 and n % 1000 == 0:
            print(f"Finished {n} of {iterations} items.")
    print(f"Added {completed} items.")


if __name__ == "__main__":
    iterations = 50
    name="rating_over_time"
    try:
        iterations = int(sys.argv[1])
    except:
        pass
    try:
        name= sys.argv[2]
    except:
        pass
    collection = get_or_generate_collection(name="rating_over_time")
    run(collection, iterations=iterations)
```

Now let's generate some data. First we'll start off with `100` items:
```bash
python generate_data.py 100
```
This should be fast and show the following output:
```bash
collection rating_over_time already exists. Continuing...
Added 100 items.
```

Now, `1_000` (1,000)
```bash
python generate_data.py 1_000
```

And finally `100_000` (100,000):
```bash
python generate_data.py 100_000
```
This might take awhile but it's a good idea to have a diverse dataset in our collection. It's not actually diverse because the data is simulated but it will, hopefully, show us some interesting patterns in the data.


Now you might be wondering about this block:

```python
if skew_results:
  if cuisine.lower() == "mexican":
      rating = random.choice([4, 5])
  elif cuisine.lower() == "bar food":
      rating = random.choice([1, 2])
  elif cuisine.lower() == "sushi":
      rating = get_random_rating(skew_low=False)
```
Using pseudo-random data generators like we did in this example will, after enough data, likely skew all of our data points to be roughly the same. Having this block allows us to skew the results a little more in the way we might like. Feel free to play around with this as you'd like going forward. The next step will show you how to review the time series results to see why this skew might be necessary.


## Step 7. Time Series Aggregation Pipeline

Now we're going to aggregate our time series data. This entire blog post is really about this step. 

Let's have a look.

**Aggregation Pipeline Basics**

First and foremost, we need to see how we can actually aggregate data using PyMongo and MongoDB.

I recommend using the Python Shell and/or Jupyter notebooks for this step.


```bash
python shell
>>> 
```

```python
import db_client

client = db_client.get_db_client()
db = client.business
collection = db["rating_over_time"]
print(collection.find_one())
```
This should yield some like:

```python
{'timestamp': datetime.datetime(2008, 12, 1, 14, 2, 48, 107000), 'metadata': {'cuisine': 'Burgers', 'name': 'Fire State'}, '_id': ObjectId('62f2b7eb6672d1f1e97148a4'), 'rating': 3}
```

We can refer to each field as follows:
- `timestamp`
- `metadata`
  - `metadata.cuisine`: this is mapped to the `cuisine` field within the `metadata` dictionary/object
  - `metadata.name`: this is mapped to the `name` field field within the `metadata` dictionary/object
- `_id`: Document Object Id
- `rating`: Integer value of this particular time series entry

It's important to note each field because one of the first steps in aggregation is to group these items in some way.

I want to get the average rating for a restaurant. With this data, I am assuming that `metadata.name` and `metadata.cuisine` are unique together. In other words, a restaurant with the same name but different cuisine will be treated as a different business all together.


Here's how we can aggregate this data:

```python
results = list(collection.aggregate([
  {"$group": {
   "_id": {"name": "$metadata.name", "cuisine": "$metadata.cuisine"},
    "count": {"$sum": 1},
    "currentAvg": {"$avg": "$rating"},
  }}
]))
```

```python
>>> print(results[0])
{'_id': {'name': 'Fire Tasty', 'cuisine': 'Steak'}, 'count': 182, 'currentAvg': 2.9615384615384617}
>>> print(len(results))
1296
```
So we see a few things here that are important.

- `collection.aggregate` initializes a pipeline and takes a list `[]` as an argument. This list allows us to perform various operations on the data based on it's position in the list (more on this later).
- `{"$group": {}}` denotes how we're going to group this data including group-related operations.
- `_id` we must declare an `_id` for this group as this `_id` is how MongoDB will roll this data up. 
- `{"name": "$metadata.name", "cuisine": "$metadata.cuisine"}`:
    -  In this case, we referencing two of the original fields from the collection (`metadata.name` and `metadata.cuisine`). To reference data from the collection we must use the format `$fieldName.withDot.Notation`; the dollar sign `$` must proceed the field name. If you do *not* have a dollar sign, MongoDB will automatically set the field to whatever string you write.
    -  The new fields `name` and `cuisine` are arbitrary. You can name them `business` and `food_type` if you like. The results will be the same just with different field names
    -  These two fields combine to make a unique combination for this aggregation. This unique combo will be responsible for how the data is aggregated and how operations occur on the data. In practice, the restaurant would probably have 2 other unique ids that we would consider adding to the `metadata` dictionary: `location_id` and `business_id`. I did not included these other fields for simplicity.
- `"count": {"$sum": 1}`. The field `count` here is an arbitrary name once again but it makes sense for what data we're looking to exact. I want to know the total sum of records that match the `_id` we generated for this group. In other words, how many documents do we have that contain the same `metadata.name` and `metadata.cuisine`? The `{"$sum": 1}` operator handles this for us.
- `"currentAvg": {"$avg": "$rating"}`. Again, the `currentAvg` name is arbitrary. The important part is `{"$avg": "$rating"}`. In this case, we're using the `$avg` operator but on the field `$rating`. MongoDB will calculate the average rating for this entire group based the new group `_id` as well as the `average` field. 


What if we wanted to round the `currentAvg` to 2 decimal places?

Here's what our aggregation would look like:

```python
results = list(collection.aggregate([
  {"$group": {
   "_id": {"name": "$metadata.name", "cuisine": "$metadata.cuisine"},
    "count": {"$sum": 1},
    "currentAvg": {"$avg": "$rating"},
  }},
  {"$addFields": {"roundedAverage": {"$round": [ "$currentAvg", 2]}}}
]))
```

The changes are as follows:
- `{"$addFields": {}}` is a new item in the `collection.aggregate` list. The `roundedAverage` field will be added *after* the first step is complete (ie the `$group` step). This is important so we can use the results of the first step.
- `{"roundedAverage": {}}`: `roundedAverage` is the new field that we're adding to our aggregation. This addition will be handled on a smaller set of data since step 1 already occurred.
- `{"$round": [ "$currentAvg", 2]}` The `$round` operator takes a list as argument `[]` that includes two main pieces: the field, and the decimal to round to. In our case we choose the `$currentAvg` field form the previous step in the aggregation until 2 decimal places.


To clean the data up one more time, we'll remove the field `currentAverage` since it contains too many decimal places.

```python
results = list(collection.aggregate([
  {"$group": {
   "_id": {"name": "$metadata.name", "cuisine": "$metadata.cuisine"},
    "count": {"$sum": 1},
    "currentAvg": {"$avg": "$rating"},
  }},
  {"$addFields": {"roundedAverage": {"$round": [ "$currentAvg", 2]}}},
  { "$replaceWith": {
    "$unsetField": {
       "field": "currentAvg",
       "input": "$$ROOT"
   } } },
]))
```

As we see:

```python
  { "$replaceWith": {
    "$unsetField": {
       "field": "currentAvg",
       "input": "$$ROOT"
   } } },
   ```
Is the method for removing any field from the pervious step(s). In this case, `currentAvg` was the field that was removed.

If review any given result we should see something like:

```python
{'_id': {'name': 'Fire Tasty', 'cuisine': 'Steak'}, 'count': 211, 'roundedAverage': 2.95}
```

Now this is pretty cool. But this is standard MongoDB aggregation. It's not time series aggregation. Let's have a look on how to do that.


**The Time Series Aggregation Pipeline**

Time Series data helps us understand changes over time. We could do days, months, weeks, years, hours, minutes, and even seconds. Anything smaller than seconds is a bit too far outside the scope of this blog post.

I'm going to use month and year for the time series portion our aggregation. In simple terms, grouping the data by month and year, performing averages, and eventually plotting those averages.


Now let's remember a key thing from the last aggregation we did: `$group` and `_id`. We have to generate an compelling `_id` that includes our time series data. But how to do that? Do we want our data group with anything other than time series data? Perhaps cuisine type? Perhaps business name? Perhaps all of that?

The first step we need to do is enrich our dataset with a group-friendly date string because the `timestamp` field is not a date field. Let's change that.

To enrich an aggregation *prior* to grouping the data, we can use the `$project` operator. Like this:

```python
results = list(collection.aggregate([
  {"$project": {
    "date": { 
        "$dateToString": { "format": "%Y-%m", "date": "$timestamp" } 
    },
  }}
]))
print(results[:1])
```
This will yield something like:

```python
[{'_id': ObjectId('62f2b8bc6672d1f1e9742914'), 'date': '2022-08'}]
```
As we see `$project` actually modified the incoming data. Let's modify it so it contains the relevant details we need: `cuisine` and `rating`. 


```python
results = list(collection.aggregate([
  {"$project": {
    "date": { 
        "$dateToString": { "format": "%Y-%m", "date": "$timestamp" } 
    },
    "cuisine": "$metadata.cuisine",
    "rating": "$rating",
  }}
]))
print(results[:1])
```

This will yield something like:
```python
[{'_id': ObjectId('62f2b8036672d1f1e971a222'), 'date': '2008-11', 'cuisine': 'Steak', 'rating': 3}]
```

At this point we learned a few things:
- Using `$project` can help enrich our data and elminate data we don't need
- `"$dateToString": { "format": "%Y-%m", "date": "$timestamp" }` shows us how to convert our timestamp field `$timestamp` into the format date format `%Y-%m` which yields `'2008-11'`
- Both `cuisine` and `rating` reference data from the original collection

Now let's create a group:


```python
results = list(collection.aggregate([
  {"$project": {
    "date": { 
        "$dateToString": { "format": "%Y-%m", "date": "$timestamp" } 
    },
    "cuisine": "$metadata.cuisine",
    "rating": "$rating",
  }},
  { 
      "$group": {
        "_id": {
            "cuisine": "$cuisine",
            "date": "$date",
        },
        "average": { "$avg": "$rating" },
      }
     }
]))
print(results[:1])
```

The result we get is:

```python
[{'_id': {'cuisine': 'Fast Food', 'date': '2021-12'}, 'average': 3.1379310344827585}]
```

This is so close to how I want it. I'm going to add 2 new fields to make plotting our results easier. 


```python
results = list(collection.aggregate([
  {"$project": {
    "date": { 
        "$dateToString": { "format": "%Y-%m", "date": "$timestamp" } 
    },
    "cuisine": "$metadata.cuisine",
    "rating": "$rating",
  }},
  { 
      "$group": {
        "_id": {
            "cuisine": "$cuisine",
            "date": "$date",
        },
        "average": { "$avg": "$rating" },
      }
     },
     {"$addFields": {"cuisine": "$_id.cuisine" }},
     {"$addFields": {"date": "$_id.date" }}
]))
print(results[:1])
```
Now I added `cuisine` and `date` fields to make it easier to plot our data with Python Pandas.


An alternative method to adding these fields would be to do:

```python
results = list(collection.aggregate([
  {"$project": {
    "date": { 
        "$dateToString": { "format": "%Y-%m", "date": "$timestamp" } 
    },
    "cuisine": "$metadata.cuisine",
    "rating": "$rating",
  }},
  { 
      "$group": {
        "_id": {
            "cuisine": "$cuisine",
            "date": "$date",
        },
        "average": { "$avg": "$rating" },
        "cuisine": { "$first": "$cuisine" }, 
        "date": { "$first": "$date" },
      }
     },
]))
print(results[:1])
```
Both `{ "$first": "$cuisine" }` and `{ "$first": "$date" }` will get the first instance in the list of items that feed into this group. We could actually grab any instance simply because the data is being grouped by these same fields. The previous method is preferred in my opinion.


**Plotting PyMongo & MongoDB Time Series with Python Pandas**

Earlier in this post, we installed [pandas](https://pandas.pydata.org/docs/). The only reason to install this was to simply turn our time series aggregation pipeline into a plotted chart.

It's pretty simple.


First, we need to use pandas:

```python
import pandas as pd
```
Next, we'll use our results from our aggregation pipeline above and turn them into a pandas DataFrame:

```python
df = pd.DataFrame(results)
```
Now we'll only select the data we need:

```python
df = df[['date', 'cuisine', 'average']]
```

Next, we'll convert the date string (from the results data) into a datetime instance for pandas. It's not a datetime instance simply because it's aggregated data.
```python
df['date'] = pd.to_datetime(df['date'])
```
Then we'll reset the dataframe index to be based on the date column (for time series analysis in Pandas)
```python
df.set_index('date', inplace=True)
```
Finally, we're going to narrow our results down by dates as well as grouping the data via the cuisine type and exacting each cuisine's average for any given date.

```python
start_date = '2018-01-01'
end_date = '2022-12-31'
time_series_data =  df.loc[start_date:end_date].groupby('cuisine')['average']
```
You can play around with the `start_date` and `end_date` as you see fit. You can also omit the entire `.loc[start_date:end_date]` all together if you just want all the data from the pipeline. I recommend leveraging the `pymongo` pipeline more than pandas since `MongoDB` will most likely be much more efficient than pandas for this data. Either way, it's good to experiment with both!

Finally, let's plot!

```python
time_series_data.plot(legend=True)
```
If you don't see a chart, let's just go ahead and save our plot:

```python
plot_series = time_series_data.plot(legend=True)
plot_figure = plot_series[0].get_figure()
plot_figure.savefig("output.png")
```




**Time Series Pipeline to Matplotlib Chart via Pandas**
Now let's create a module we can use for creating these charts.

Create `src/chart.py`:

```python
import datetime
import pathlib

import pandas as pd

import db_client


def output_chart(start_date = '2018-01-01', end_date = '2022-12-31'):
    client = db_client.get_db_client()
    db = client.business
    collection = db['rating_over_time']
    dataset = list(collection.aggregate([
      {"$project": {
        "date": { 
            "$dateToString": { "format": "%Y-%m", "date": "$timestamp" } 
        },
        "cuisine": "$metadata.cuisine",
        "rating": "$rating",
      }},
      { 
      "$group": {
        "_id": {
            "cuisine": "$cuisine",
            "date": "$date",
        },
        "average": { "$avg": "$rating" },
      }
      },
      {"$addFields": {"cuisine": "$_id.cuisine" }},
      {"$addFields": {"date": "$_id.date" }}
    ]))


    df = pd.DataFrame(dataset)
    df['date'] = pd.to_datetime(df['date'])
    df = df[['date', 'cuisine', 'average']]
    df.set_index('date', inplace=True)

    base_dir = pathlib.Path(__file__).parent.parent
    output_dir = base_dir / 'plots' / 'cuisines'
    output_dir.mkdir(exist_ok=True, parents=True)
    now = datetime.datetime.now()

    group_data =  df.loc[start_date:end_date].groupby('cuisine')['average']
    # generate a chart with a legend and is 10" wide x 5" tall
    chart_a = group_data.plot(legend=True, figsize=(10, 5))
    fig = chart_a[0].get_figure()
    fig.savefig(str( output_dir / f'{int(now.timestamp())}-{start_date}-{end_date}.png'))


if __name__ == "__main__":
    start_date = '2018-01-01'
    end_date = '2022-12-31'
    output_chart(start_date=start_date, end_date=end_date)
```


Now let's navigate into our root project folder.

```bash
cd ..
```
Let's generate a time series chart:

```bash
python src/chart.py
```
And another
```bash
python src/chart.py
```
And another
```bash
python src/chart.py
```
In your project you should see a new folder `plot/cuisines` with some charts in it.




## Wrap up
Nice work if you got this far! Time Series Data in MongoDB is incredibly compelling. The nice thing about a *lot* of the code above is we can use it within a web application (like FastAPI or Flask) with very little changes.

MongoDB has a lot of promising features for a database that makes modern time series development such a joy.

Until next time,

Justin
