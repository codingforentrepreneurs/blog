---
title: Jupyter Notebook Server with AWS EC2 and AWS VPC
slug: jupyter-notebook-server-aws-ec2-aws-vpc

publish_timestamp: April 16, 2019
url: https://www.codingforentrepreneurs.com/blog/jupyter-notebook-server-aws-ec2-aws-vpc/

---

This is a step-by-step guide to deploying a Jupyter Notebook Server to AWS EC2 and AWS VPC. We have a related video series [here](/projects/jupyter-x-aws).

# Create your VPC
You need a virtual private cloud (VPC) for your EC2 instances to actually work. I think it's best to work through this on your own. There isn't extra cost for VPCs so you can practice this portion as many times as you'd like.

VPCs enable you to have a LOT of EC2 instances even if we're only adding 1 in this guide.


### 1. Login to [AWS Console](https://console.aws.amazon.com)

### 2. Select the `us-east-1` region, __N. Virginia__, top right corner

### 3. Navigate to [VPC](https://console.aws.amazon.com/vpc/)

1. On the sidebar, click **Your VPCs**
2. Click **Create VPC**
3. In the **Create VPC** section, do the following:
- Name tag : JupyterVPC
- IPv4 CIDR block: `10.0.0.0/16`
The block `10.0.0.0/16` allows for 65,536 possible IP addresses for your VPC EC2 instances

4. Click "Create"

> For more on CIDR blocks, read [this]() and use [this](https://cidr.xyz) to select your CIDR range.


### 4. Create Subnet
For this series we just need 1 public subnet. A public subnet (vs a private one) will allow the Jupyter server to be reachable by the internet.

1. We're still in the [VPC](https://console.aws.amazon.com/vpc/) console
2. On the sidebar, click **Subnets**
3. Click **Create subnet**
4. In the **Create subnet** section, do the following:
- Name tag : JupyterPublicSubnet
- VPC: *select the vpc from above*
- Availability Zone: You can select *no preference* or choose an AZ from the dropdown
- IPv4 CIDR block: `10.0.1.0/24`

If you want multiple subnets, you'll need to use a CIDR block that has a smaller count. Counter to intuition, the last number needs to be higher in order to have a different block ie (`10.0.1.0/24` vs `10.0.0.0/16`). 

5. Click "Create"
6. Select the Subnet newly created subnet
7. Under "Actions" select
- Select "Modify auto-assign IP settings"
- Check `Auto-assign IPv4`
- Click "Save"

### 5. Create Internet Gateway

1. We're still in the [VPC](https://console.aws.amazon.com/vpc/) console
2. On the sidebar, click **Internet Gateways**
3. Click **Create internet gateway**
4. In the **Create internet gateway** section, do the following:
- Name tag : JupyterIGW
5. Click "Create"
6. Select newly created internet gateway. The current state should read `detached`
7. Under "actions", select "attach to VPC"
8. Select your `JupyterVPC` created above.
9. Click **Attach**


### 6. Update Routing Table 
Your routing table allows the internet gateway to be accessed.

1. We're still in the [VPC](https://console.aws.amazon.com/vpc/) console
2. On the sidebar, click **Route Tables**
3. Select your JupyterVPC routing table. The Name tag will be under the `VPC ID` column
4. After selected, look for the **Routes** tab under the list of route tables.
5. Select **Edit Routes**
6. Click **Add Route** and add the following:
- Destination: `0.0.0.0/0`
- Target: Select "Internet Gateway" and find your *JupyterIGW* from above
7. Click **Save Routes**




# Provision an EC2 Instance


### 1. Login to [AWS Console](https://console.aws.amazon.com)

### 2. Select the `us-east-1` region, __N. Virginia__, top right corner


### 3. Navigate to [EC2](https://console.aws.amazon.com/ec2/) and create an Ubuntu Instance

We'll use Ubuntu since setting it up for what we need is simple enough. If you did our project [hello linux](), you'll be able to do literally all of that with an Ubuntu EC2 as well.


1. On the sidebar, click **Instances**
2. Click **Launch Instance**

##### Step 1. Choose an Amazon Machine Image (AMI)
- Search and select Ubuntu Server 18.04 LTS
- Use 64-bit (x86)
- Click **Select**

##### Step 2. Choose an Instance Type
- If it's your first few times doing this, use `t2.micro`. Since it's a free tier.
- Click **Next: Configure Instance Details**

##### Step 3: Configure Instance Details
We'll use the following details:

Number of instances | 1
Network | Select your Jupyter VPC from above
Subnet | It should auto-populate if you created 1 subnet. If not, be sure to get the public subnet

- **Advanced Details** / **Bootstrap Scripts**
This will run the installations we need prior to going into this EC2 instance. As we configure the instance, un-collapse the **Advanced Details** tab and add the following in `User data` **as text**

```
#!/bin/bash

sudo apt-get update -y

sudo apt-get install build-essential libssl-dev libpq-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip -y

sudo apt-get install supervisor -y 

sudo apt-get install python3-pip python3-dev python3-venv -y

sudo apt-get install nano -y

sudo apt-get install git -y 

sudo apt-get install nginx curl -y

sudo apt-get install ufw -y

sudo ufw allow 'Nginx Full'

sudo ufw allow ssh

sudo python3 -m pip install jupyter

sudo service supervisor start

sudo apt autoremove -y
```
- Click **Next: Add Storage**

##### Step 4: Add Storage
Use the defaults. Notice that *Delete on Termination is checked* this means that everything related to this instance will be gone once you terminate it.

- Click **Next: Add Tags**


##### Step 5: Add Tags
Add tags if you want. I'll leave them empty

- Click **Next: Add Tags**



##### Step 6: Configure Security Group
We'll create a new security group. With a security group, we can add support for HTTP, HTTPS, and SSH. Since the Jupyter notebook server needs at least http, we'll add that too.

- **Assign a security group:** Check *Create a new security group*
- **Security group name:** JupyterSecurityGroup
- **Description:** Leave default
- Click *Add rule* and fill in the following:
    - Type: Http
- Click *Add rule* and fill in the following:
    - Type: Https

You should now have 3 rules for this security group.

- Click **Review and Launch**


##### Step 7: Review Instance Launch
Review the details.

- Click **Launch**

You should see a pop up for "key pairs"

- Select "Create a new key pair"
- In **Key pair name** add `JupyterKey`, this is how you'll be able to ssh into your vpc.
- Select "Download Key Pair"
- Select "Launch Instance"
- Select "View Instances"
- Under "Description" for your newly created instance, look for and copy your **IPv4 Public IP**


##### Step 8: Wait

Your EC2 is going to take a few minutes to provision and then install all of our bootstrap scripts.

Open up your browser to your **IPv4 Public IP** something like:

```
http://34.235.154.196
```

You should see a default nginx page which means it's working! 




# Provision the Jupyter Server

Above we created an `pem` private key so we can ssh into our EC2 instance. If you lost this or need a new one, the easies option is to launch a new EC2 instance. You won't need to create another VPC because that's already created.


### 1. SSH into EC2

SSH means "secure shell" which means you can type commands on a remote device. In our case, it's our EC2 instance. 

- Mac / Linux users: use **Terminal** 
- Windows users: use **PowerShell** or **PuTTy**

Move into a working directory. Make sure your private key (created in last section) is there too. A couple things to note:

`<you-ip>` is your **IPv4 Public IP** address. 

`<your private key>` is the private key you created above should be `JupyterKey.pem`

The commands are:
```
chmod 400 <your private key>.pem
ssh ubuntu@<your-ip> -i <your private key>.pem
```

My example:
```
cd path/to/my/dev/folder/
chmod 400 JupyterKey.pem
ssh ubuntu@34.235.154.196 -i JupyterKey.pem
```



### 2. Double check installs. 
In some cases you may have skipped the bootstrap scrips above. You can verify installs by just attempted to install again. Be sure to `ssh` into your EC2 instance first.

```
sudo apt-get update -y

sudo apt-get install build-essential libssl-dev libpq-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip -y

sudo apt-get install supervisor -y 

sudo apt-get install python3-pip python3-dev python3-venv -y

sudo apt-get install nano -y

sudo apt-get install git -y 

sudo apt-get install nginx curl -y

sudo apt-get install ufw -y

sudo ufw allow 'Nginx Full'

sudo ufw allow ssh

sudo python3 -m pip install jupyter

sudo service supervisor start

sudo apt autoremove -y
```



#### 3. Generate Jupyter Config 

`jupyter notebook --generate-config`

This command will create configuration file we need to deploy run a jupyter notebook server. If it fails, there's a good chance you didn't do the installations correctly.

The config file should be located under

`/home/ubuntu/.jupyter/jupyter_notebook_config.py`


**Get Your public ip address**
```
ipAdd=`curl http://169.254.169.254/latest/meta-data/public-ipv4`
echo "${ipAdd}"
```
_Why is this here? I added this for future reference so you can make more advanced bootstrap scripts_

**Create your default password**
Our configuration will *not* work as you may work locally. We want to password protect our jupyter environment. Running this command will allow you to add a password and it will return an encrypted string.

```
ipython -c "from notebook.auth import passwd; passwd()"
```
Enter the above command, enter your passwrod, and you'l get the following output:
```
'sha1:45e47cb75d9e:49dc0b09f4e671485b6113c1e2c5a13d7d37fa78'
```
Be sure to copy that output.



Update our configuration for live running.
```
sudo nano /home/ubuntu/.jupyter/jupyter_notebook_config.py
```

```
c = get_config()

# Kernel config
c.IPKernelApp.pylab = 'inline'  # if you want plotting support always in your notebook

# Notebook config

c.NotebookApp.allow_origin = 'http://107.21.189.212' # put your public IP Address here
c.NotebookApp.ip = '*'
c.NotebookApp.allow_remote_access = True
c.NotebookApp.open_browser = False
c.NotebookApp.password = u'sha1:45e47cb75d9e:49dc0b09f4e671485b6113c1e2c5a13d7d37fa78'
c.NotebookApp.port = 8888

# For https & letsencrypt later
# c.NotebookApp.certfile = u'/your/cert/path/cert.pem'
# c.NotebookApp.keyfile = u'/your/cert/path/privkey.pem'
```

`c.NotebookApp.allow_origin` this setting is very important. Make sure you put your same public IP address here otherwise you're not going to be to connect to the Jupyter kernel because of CORS errors.

> Official Jupyter Config [docs](https://jupyter-notebook.readthedocs.io/en/stable/config.html)





#### 4. Nginx Settings
Now we can configure nginx as to be routed to the jupyter server. You can 100% use Apache2 instead but nginx is very easy to configure amoung many other strengths.

**Note**: the `jupyter_app.conf` name is arbitrary.


```
sudo nano /etc/nginx/sites-available/jupyter_app.conf
```

```
server {
    server_name jupyter_notebook;
    listen 80;
    listen [::]:80;

    location / {
        include proxy_params;
        proxy_pass http://localhost:8888;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $http_host;
        proxy_http_version 1.1;
        proxy_redirect off;
        proxy_buffering off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
    }
}
```

If you intend to use a custom domain, replace `server_name jupyter_notebook;` with `server_name your_custom_domain.com;`

```
sudo ln -s /etc/nginx/sites-available/jupyter_app.conf /etc/nginx/sites-enabled/jupyter_app.conf

sudo rm /etc/nginx/sites-enabled/default
```

```
systemctl daemon-reload

sudo systemctl reload nginx
```





### 5. Setup Supervisor 
Create supervisor program to ensure jupyter runs in the background and restarts if the EC2 instance is restarted.


**Note**: the `my_jupyter.conf` name is arbitrary.


```
sudo nano /etc/supervisor/conf.d/my_jupyter.conf
```

```
[program:my_jupyter]
user=ubuntu
directory=/home/ubuntu
command=jupyter notebook
autostart=true
autorestart=true
stdout_logfile=/var/log/my_jupyter/stdout.log
stderr_logfile=/var/log/my_jupyter/stderr.log
```

```
sudo mkdir /var/log/my_jupyter
```

```
sudo supervisorctl reread
sudo supervisorctl update
```



### 6. Navigate to your ip
Boom! You now have a jupyter notebook server.

### 7. Assign an Elastic IP
Now we want to be able to stop our EC2 instance and restart it at anytime. We're going to do this by using an Elastic IP

1. Go into [EC2](https://console.aws.amazon.com/ec2/) 
2. On the sidebar, under "Network and security" select "Elastic IP"
3. Click "Allocate new address"
- Under "IPv4 address pool" choose "Amazon Pool"
- Click "Allocate"
- Click "close"
4. Select New Elastic IP, click "Actions > Associate Address" and:
- Use the "Instance" resource
- Select your instance
- Select your instance private ip
- Check "Allow Elastic IP to be reassociated if already attached"
- Click "Associate"

5. You now have an IP address that you can reuse. Update the above configuration for your new IP Address.




### 8. Thank you and next steps:

1. Auto shutdown timer. What if you accidentally forget to shutdown your instance? You'll be billed for all that time. That's not a big deal when you're using the `t2.micro` but if you move to a more powerful EC2 instance, then it's a really good idea to have an auto shutdown timer.
2. Add HTTPs with LetsEncrypt. Above we left some configuration notes to do this exactly
3. Run this step on a GPU-enabled EC2 instance so you can do machine learning
4. Try to use JupyterHub so you can have many of your own users.
5. Create an EC2 AMI for re-using your Jupyter server.