---
title: Getting Started with HarperDB &amp; Express.js on Ubuntu x Linode
slug: haperdb-expressjs-ubunutu-linode

publish_timestamp: Dec. 6, 2022
url: https://www.codingforentrepreneurs.com/blog/haperdb-expressjs-ubunutu-linode/

---


<div class='alert-warning p-5 text-center font-bold text-lg text-black'>Coming Soon!</div>

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
harperdb install --TC_AGREEMENT yes --HDB_ROOT /home/ubuntu/hdb --SERVER_PORT $HARPER_SERVER_PORT --HDB_ADMIN_USERNAME $HDB_ADMIN_USERNAME --HDB_ADMIN_PASSWORD '$HDB_ADMIN_PASSWORD' --HTTPS_ON true --CUSTOM_FUNCTIONS true --CUSTOM_FUNCTIONS_PORT $HARPER_CUSTOM_FUNCTIONS_PORT
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

apt-get update && apt-get upgrade -y # takes a few minutes
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
Coming soon
