---
title: Getting Started with HarperDB &amp; Express.js on Ubuntu x Linode
slug: haperdb-expressjs-ubunutu-linode

publish_timestamp: Dec. 19, 2022
url: https://www.codingforentrepreneurs.com/blog/haperdb-expressjs-ubunutu-linode/

---


In this post, we'll be setting up a Node.js application running Express.js and HarperDB on Ubuntu on Linode. You can use any Ubuntu LTS from version 18.04 and, but we'll be using Ubuntu 20.04 LTS for this guide. 

The goal of this is to introduce you to leveraging the flexible HarperDB database along with a minimal Express.js application. The HarperDB instance will be running on Linode we'll our Express.js web server will be running just on our local machine.

If you like this one, let me know in the comments so we can do more Node.js/Express/HarperDB in the future.

## Setup HarperDB
Parts of this section has been adapted from official HarperDB Repos at [HarperDB Add-Ons](https://github.com/HarperDB-Add-Ons).


### Create instance
1. Login or create a [linode account](https://linode.com/cfe)
2. Navigate to Create > Linode
3. Under Distribution Images, select `Ubuntu 20.04 LTS` 
4. Under Region, select `Dallas, TX` or whatever is closest to you
5. Linode Plan: I recommend at least a `Shared CPU - Linode 2GB` plan ($10/mo).
6. Linode Label: `harperdb-instance-1` (or any name you want)  
7. Add tags (optional)
8. Root Password: Pick a secure password. You can use Python to generate one:
```bash
python3 -c 'import secrets; print(secrets.token_urlsafe(32))'
```
9. SSH Keys (optional but recommended)
10. Click `Create Linode`

### SSH Into `harperdb-instance-1`
After a couple minutes your Ubuntu instance will be ready on Linode. Grab the IP address mine was `96.126.123.201`


```bash
ssh root@96.126.123.201
```
With Ubuntu the default user will be `root`. 

### Create `ubuntu` User

```bash
useradd --create-home --shell /bin/bash --groups sudo ubuntu
```
Generate another new password
```bash
python3 -c 'import secrets; print(secrets.token_urlsafe(32))'
```
Set the new user `ubuntu`'s password:

```
passwd ubuntu
```

### Update default configuration for the `ubuntu` user

#### Increase Number of Open Files for User
```bash
echo "ubuntu soft nofile 1000000" | tee -a /etc/security/limits.conf
echo "ubuntu hard nofile 1000000" | tee -a /etc/security/limits.conf
```

#### Ensure Swap is Enabled
```bash
swapon -s # Confirm swap file/partition exists
```

#### Increase Number of Allowed Network Connections

```bash
echo -e "net.ipv4.tcp_tw_reuse='1'\nnet.ipv4.ip_local_port_range='1024 65000'\nnet.ipv4.tcp_fin_timeout='15'" >> /etc/sysctl.conf

su - ubuntu
```
### Install Node Version Manager & Node.js

Install nvm
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
. /home/ubuntu/.nvm/nvm.sh
. /home/ubuntu/.bashrc
```

Use nvm to install Node.js version 14 (required by HarperDB)
```bash
nvm install 14.20.0
```

Update npm
```bash
npm install -g npm@latest
```


### Install HarperDB
With Node.js installed, we now have `npm` on our database server to install HarperDB. This will take a few minutes to install:

```bash
npm install -g harperdb@3.3.0 --verbose
```

Set your default HarperDB credentials:

```
export HARPER_SERVER_PORT=9925
export HARPER_CUSTOM_FUNCTIONS_PORT=9926
export HDB_ADMIN_USERNAME=HDB_ADMIN
export HDB_ADMIN_PASSWORD=$(python3 -c 'import secrets; print(secrets.token_urlsafe(32))')
```
Be sure to run `echo $HDB_ADMIN_PASSWORD` and store this password somewhere safe.

### Start HarperDB
```bash
harperdb install --TC_AGREEMENT yes --HDB_ROOT /home/ubuntu/hdb --SERVER_PORT $HARPER_SERVER_PORT --HDB_ADMIN_USERNAME "$HDB_ADMIN_USERNAME" --HDB_ADMIN_PASSWORD "$HDB_ADMIN_PASSWORD" --HTTPS_ON true --CUSTOM_FUNCTIONS true --CUSTOM_FUNCTIONS_PORT $HARPER_CUSTOM_FUNCTIONS_PORT
```
You should see something like this:
```bash
Starting HarperDB...

               ▒▒▒▓▓▓▓▓▓▓▓▓▓▓▒▒                                
           ▒▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒                     
       ▒▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▓▓▓▓▓▓▓▓▓▓▓▒▒                    
   ▒▒▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓▒                   
   ▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▒                  
    ▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▒▒                
    ▒▒▒▒▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒          
   ▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒    
  ▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒▒▒
 ▒▒▒▒▒▒▒▒▒▒▒▒▒▓▓▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒▒ 
▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒   
  ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒      
     ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒        
         ▒▒▒▓▓▓▓▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒                          
            ▒▒▒▓▓▓▓▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒▒                          
               ▒▒▒▒▓▓▓▓▓▓▒▒▒▒▒▒▒▒▒▒▒                           
                   ▒▒▒▓▓▒▒▒▒▒▒▒▒▒▒▒                            
                      ▒▒▒▒▒▒▒                                  

                    HarperDB, Inc. Denver, CO.

|------------- HarperDB 3.3.0 successfully started ------------|
```
### Add Crontab & Restart Instance


Add to Crontab for Reboot
```bash
crontab -l 2>/dev/null; echo "@reboot PATH=\"/home/ubuntu/.nvm/versions/node/v14.20.0/bin:$PATH\" && harperdb" | crontab -
```

Update & Restart
```bash
exit # to return to root user

sudo apt-get update && sudo apt-get upgrade -y # takes a few minutes
reboot now
```

### Connect Our HarperDB to HarperDB Studio

1. Get or create a free [Harper Studio account](https://studio.harperdb.io/?utm_source=cfe&utm_medium=article&utm_campaign=cfe)
2. Login
3. Click the `+` icon where it says _Create New HarperDB Cloud Instance_ + _Register User-Installed Instance_
4. Select `Register User-Installed Instance` using:
- `Instance Name`: `linode-1`
- `Username`: `HDB_ADMIN`
- `Password`: Did you save the result of `echo $HDB_ADMIN_PASSWORD` from before?
- `Host`: `<your linode ip>` (mine was `96.126.123.201`)
- `Port`: `9925` (the default port for HarperDB)
- `SSL`: `true`
  

## Setup Express.js
Now that we have our database setup, let's integrate it with HarperDB and the minimal Node.js framework Express.js.

### Create Project Folder

```bash
mkdir ~/express-harperdb
cd ~/express-harperdb
```

### Create Node.js Package

```bash
npm init
```
Accept all the defaults resulting in:

```bash
cat package.json
```

Returning something such as:
```json
{
  "name": "nodejs-harperdb",
  "type": "module",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
}
```

### Update Environment Variables
In the root of our project, create a `.env` file and add the following:

```
HARPER_SERVER_PORT=9925
HARPER_CUSTOM_FUNCTIONS_PORT=9926
HARPERDB_USER=HDB_ADMIN
HARPERDB_PW=pil62BL-I4jgdRyojJrHJ3c6vd3IEedvBpTWqGR0Aqs
HARPERDB_URL=https://50.116.28.246:9925
```

These correspond to the values set during the HarperDB installation.

- `HARPERDB_USER` is the `HDB_ADMIN_USERNAME` and configured to `HBD_ADMIN`

- `HARPERDB_PW` is the `HDB_ADMIN_PASSWORD` and configured to the result of `python -c 'import secrets; print(secrets.token_urlsafe(32))'`
- `HARPERDB_URL` combines the `HARPER_SERVER_PORT` and the IP Address of your Linode Instance from the previous step.


### Install Dependencies 

Now let's get our project setup with the following:

- *Express.js* is a minimal web framework for Node.js.
- *dotenv* loads our environment variables from a `.env` file. 
- *node-fetch*: allow us to make HTTP requests to our HarperDB instance. This is the Node.js version of `fetch()` from the browser. Works great with `async/await` and is nearly the same as you might expect.


Install with:

```bash
npm install express dotenv node-fetch
```

Then dev-only dependencies:
- *nodemon* automatically reloads our Express.js server when we make changes to the code and thus suitable for development.

Install with:

```bash
npm install nodemon --save-dev
```

### Root Server
In `app/index.js` add the following:

```javascript
import express from 'express';
const app = express();
const PORT = 8000;

app.get('/', (req, res) => res.json('Express Server'));

app.listen(PORT, async () => {
  console.log(`⚡️[server]: Server is running at http://localhost:${PORT}`);
});
```
Later in this guide, we'll replace the entire `app/index.js`. Right now we just want to ensure it's running.

Add a shortcut command to your `package.json` to include the following script:

```json
"scripts": {
  "dev": "nodemon app/index.js",
  ...
},
```
Now we can simply run `npm run dev` (or `yarn dev` if you have yarn installed) to start our server.


### Load Environment Variables

In `app/configEnv.js` add the following:

```javascript
import path from 'path';
import dotenv from 'dotenv';

// Load the current file's directory
const baseDir = path.basename(import.meta.url);

// Define the absolute path to the .env file in the root project directory
const envFilePath = path.resolve(path.dirname(baseDir), '.env');

// Load the Environment Variables from the .env file in the root project directory
dotenv.config({ path: envFilePath})
```

Update `app/index.js` to load the environment variables at the top of the file:

```javascript
import './configEnv.js';
```


### Create a Database Client
Now we're going to create a reusable database client so we can easily make requests to our HarperDB instance through our Express.js server and our Environment Variable configuration for authentication.

In `app/client.js` add the following:
```javascript
import fetch, {Headers} from 'node-fetch';

process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";

// Define Connection Details
const DB_URL = process.env.HARPERDB_URL;
const DB_PASS = process.env.HARPERDB_PW;
const DB_USER = process.env.HARPERDB_USER;

const getAuthToken = () => {
    // Get the HarperDB username and password from the environment variables
    if (!DB_URL || !DB_PASS || !DB_USER) {
        console.log('Error: .env variables are undefined or not setup correctly.');
        throw 'Internal server error';
      }
    // Convert the username and password to base64
    const auth = Buffer.from(`${DB_USER}:${DB_PASS}`).toString('base64')
    return auth;
}

const dbClient = async (data, method) => {
  // A database client method that takes an object of data and a method (GET, POST, PUT, DELETE) then sends it to our
  // HarperDB instance and returns the result.
  const authString = getAuthToken()
  const myHeaders = new Headers();
  myHeaders.append('Content-Type', 'application/json');
  myHeaders.append('Authorization', `Basic ${authString}`);
  const jsonBodyContent = JSON.stringify(data);
  const requestOptions = {
    method: method ? method : 'POST',
    headers: myHeaders,
    body: jsonBodyContent,
    redirect: 'follow',
    rejectUnauthorized: false,
  };

  const response = await fetch(DB_URL, requestOptions);
  const result = await response.json();
  return result;
};

export default dbClient
```

### Create a HarperDB Crud Model Class
Models are a great way to organize our database interactions. In this section, we'll create a simple example of what a model might look like as an entry point into leveraging HarperDB's very flexible CRUD functionality.

First, let's create a `crud.js` file in the `app` directory with a few defaults:

```javascript
// import the database client
import client from './client.js';

// This is a workaround for the self-signed certificate HarperDB uses
process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";

```

Still in `app/crud.js`, let's create our schema (`APP_SCHEMA`) for our HarperDB tables. The schema is essentially the namespace where our tables will live.

```javascript
// Define the schema and user we'll be using for this app
const APP_SCHEMA = process.env.APP_SCHEMA || 'dev';

export const configureAppSchema = async () => {
    const schemaData = {
        "operation": "create_schema",
        "schema": APP_SCHEMA
    }
    return await client(schemaData, "POST");
}
```

You should only have to add the schema once in HarperDB but we'll ensure the schema exists every time we start our Express.js server.

Now let's create our Database model class to handle various CRUD operations (`CREATE`, `RETRIEVE`, `UPDATE`, `DELETE`). In `app/crud.js` add the following:
```javascript

export class DogModel {
  constructor() {
    this.db = client // convenience for connecting to the database
    this.table = "dog" // our table name
    this.schema = APP_SCHEMA // same schema as before
  } 

  configure = async () => {
    // Configure the table
    // Should only need to be run 1 time just like the schema
    const tableData = {
        "operation": "create_table",
        "schema": APP_SCHEMA,
        "table": this.table,
        "hash_attribute": "id"
    }
    return await this.db(tableData, "POST");
  }

  entryAdd = async (records) => {
    // This is a nosql operation so we can add multiple records into your database at once will add/insert the data into the database _or_ it will update the current data that machines the hash_attribute (id in this case) as defined in the table configuration above
      const data = {
          "operation": "upsert",
          "schema": this.schema,
          "table": this.table,
          "records": records 
      }
        return await this.db(data, "POST");
    }

    entryCreateAttribute = async (attribute) => {
      // If you need to add additional fields/attributes to your data, this is what you'll use.
        const data = {
            "operation": "create_attribute",
            "schema": this.schema,
            "table": this.table,
            "attribute": attribute
        }
        return await this.db(data, "POST");
    }

    

    entryDetail = async (id) => {
      // Using a SQL operation, retrieve a single record in our database by it's id
        const sqlLookup = `SELECT * FROM ${this.schema}.${this.table} WHERE id = ${id}`
        const data = {
            "operation": "sql",
            "sql": sqlLookup
        }
        return await this.db(data, "POST");
    }
    entryList = async () => {
      // Using a SQL operation, list all records in our database
        const sqlLookup = `SELECT * FROM ${this.schema}.${this.table}`
        const data = {
            "operation": "sql",
            "sql": sqlLookup
        }
        return await this.db(data, "POST");
    }
    entryDelete = async (id) => {
      // Using a SQL operation, delete a record from our database
        // const sqlLookup = `DELETE FROM ${this.schema}.${this.table} WHERE id = ${id}`
        const data = {
             "operation": "delete",
              "table": "dog",
              "schema": "dev",
              "hash_values": [
                id
              ]
        }
        return await this.db(data, "POST");
    }
}

export default DogModel
```

Now that we have the various database operations we might want to execute, let's update our Express.js Server to use the model.

### HarperDB and Express.JS Endpoints

Updating `app/index.js` let's add the following:

```javaScript
// Load in environment variables first
import './configEnv.js'

// import the DogModel and schema Config
import DogModel, { configureAppSchema } from './crud.js';

// Import express.js
import express from 'express';

// Create an instance of express
const app = express();

// configure the default port to listen on
const PORT = process.env.PORT || 8000;

// Handled the index route
app.get('/', (req, res) => res.json('Express Server'));


// Add records
app.get("/add", async (req, res) => {
    const records = [
        {
            "id": "1",
            "name": "Fido",
            "color": "brown",
        },
        {
            "id": "2",
            "name": "Spot",
            "favoriteFood": "peanut butter"
        },
         {
            "id": "3",
            "name": "Rover",
         },
        {
            "id": "4",
            "name": "Spike",
            "color": "black",
            "favoriteFood": "cheese"
        }
    ]
    const response = await new DogModel().entryAdd(records)
    return res.json(response);
})

// adding a new attribute
app.get("/new-attribute", async (req, res) => {
    const attribute = req.query.attribute ? `${req.query.attribute}` : "favoriteFood" 
    const response = await new DogModel().entryCreateAttribute(attribute)
    return res.json(response);
})


// list lookup
app.get("/list", async (req, res) => {
    const response = await new DogModel().entryList()
    return res.json(response);
})

// detail lookup
app.get("/detail/:id", async (req, res) => {
    const response = await new DogModel().entryDetail(req.params.id)
    return res.json(response);
})


const configure = async () => {
    // run when server starts
    const appSchemaResponse =  await configureAppSchema()
    const dogModelResponse = await new DogModel().configure()
    return {schema: appSchemaResponse, model: dogModelResponse}
}



app.listen(PORT, async () => {
  console.log(`⚡️[server]: Server is running at http://localhost:${PORT}`);
  const configuration = await configure()
  console.log(configuration)
});
```

Now that we have our Express.js server setup, let's start it up
```bash
npm run start
```

How cool is that? HarperDB is simple to setup, your database is self managed on a server _and_ you can run SQL and NoSQL operations for all of your data.
