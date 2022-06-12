# openHAB Cloud

openHAB Cloud is a companion cloud service and backend for the openHAB open-source home automation software.
The openHAB Cloud backend provides secure remote access and enables openHAB users to remotely monitor,
control and steer their homes through the internet, collect device statistics of their openHABs, receive
notifications on their mobile devices or collect and visualize data etc.
The main core features of openHAB Cloud are an user-management frontend, secure remote access, remote proxy-access, device registry & management, messaging services and data management & persistence.
The openHAB Cloud also serves as core backend integration point for cloud-based features (e.g. IFTTT) and
provides an OAuth2 application enablement.


## Functional Architecture ##



![FunctionalArchitecture](/docs/FunctionalArchitecture_openHAB-cloud_v1.0.png)



## Getting started


openHAB Cloud is mainly based on the following frameworks and technologies:


* [Node.js](https://nodejs.org/en/) - Server-side Javascript-framework
* [Express.js](http://expressjs.com/) - Web application framework for Node.js
* [Nginx](https://www.nginx.com/resources/wiki/) - Web server & reverse proxy
* [MongoDB](https://www.mongodb.com/) - NoSQL database
* [redis](http://redis.io) - Session Manager & data structure server
* [Socket.IO](http://socket.io/) - Bi-directional communication between web clients and servers


### <a name="quickStart"></a> Quick start ###

Installing openHAB Cloud on Linux (CentOS 7).

First, we need to make sure that the list of packages and dependencies
from all repositories are up-to-date:

```
yum update -y
```

We need to install easy Yum packages for Curl, Nginx and Python:

```
yum install gcc gcc-c++ make python curl -y
```
MongoDB is no longer in Yum by default, it's provided by MongoDB's manual repo. Create a new repo file:
```
vi /etc/yum.repos.d/mongdb-org-5.0.repo
```
and add the following:

```
[mongodb-org-5.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/5.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-5.0.asc
```

With the repo intact, we can install MongoDB and the command line shell.

```
yum install mongodb-org mongodb-mongosh -y
```

Redis is also not in Yum by default, it's provided by Remi's Repo. Enable that, and install Redis:

```
yum install epel-release yum-utils -y
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm -y
yum-config-manager --enable remi
yum install redis -y
```

Edit the redis configuration file, if desired, and set it to listen on only a specific IP address:

```
vi /etc/redis.conf
```
Look for the section about IP address binding and set to your tastes. 
and add your desired IP address on the line for the bind parameter:

```
bind 127.0.0.1 <your IP here> -::1
```

Next install Node.js and NPM. A good way in 2022 is using NVM, installed thusly:

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```
Close and reopen the terminal, then add NVM to your path:

```
source ~/.bashrc
```

At time of writing, Node 14.19.3 is most current LTS in the 14.x branch, so the command for NVM would be:
 
```
nvm install 14.19.3
```
Now you need install git and clone the openHAB Cloud repository to your
preferred directory (here: CentOS 7) with the following commands:

```
yum install git -y
cd /home/YOUR-DIR/
git clone [https://github.com/openhab/openhab-cloud.git](https://github.com/ElectraPort/openhab-cloud.git)
```

Clone the openhab-cloud repository and after the completed checkout you should have the directory in your chosen folder:

```
ls -al
total 32
drwxr-xr-x  5 centos centos 4096 Jun  4 17:06 .
drwxr-xr-x  3 root   root   4096 Jun  4 12:34 ..
-rw-r--r--  1 centos centos  220 Apr  9  2014 .bash_logout
-rw-r--r--  1 centos centos 3637 Apr  9  2014 .bashrc
drwx------  2 centos centos 4096 Jun  4 16:30 .cache
drwxr-xr-x 13 root   root   4096 Jun  4 17:06 openhab-cloud
-rw-r--r--  1 centos centos  675 Apr  9  2014 .profile
drwx------  2 centos centos 4096 Jun  4 12:34 .ssh
```

Now we need to change into the openhab-cloud directory and check if node is installed and properly set in the path:

```
cd openhab-cloud
node --version
```

If you see the node version, you are fine to continue (Note: openHAB Cloud is based on Node.js version 14).

To run openHAB Cloud you need to use NPM to install the required software bundles/stacks:

```
npm install
```

and all the module dependencies from **package.json** should be resolved and needed packages installed.

As of June 2022, this generates several errors/warnings that require resolution. In order to resolve them, several packages need to be manually installed/curated.

```
npm audit fix
npm install bcryptjs bcrypt ioredis@^4.14.0 mongoose sinon@^7.0.0 qunitjs
npm install underscore --location=global
npm install
npm audit fix
npm audit
```
npm WARN deprecated qunitjs@2.4.1: 2.4.1 is the last version where QUnit will be published as 'qunitjs'. To receive future updates, you will need to change the package name to 'qunit'.

At this point the NPM install returns:

```
npm WARN bcrypt-cache@2.0.1 requires a peer of bcrypt@^1.0.0 but none is installed. You must install peer dependencies yourself.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@2.3.2 (node_modules/fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.3.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.13 (node_modules/qunitjs/node_modules/fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.13: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})

added 2 packages from 4 contributors, removed 2 packages, updated 1 package and audited 1312 packages in 7.145s

found 19 vulnerabilities (4 low, 2 moderate, 10 high, 3 critical)

```

npm audit fix tells us that these 3 packages will introduce breaking changes. We'll come back to those later.

openHAB Cloud uses redis for session management.
To learn more about redis please read [here](http://redis.io).

Start and enable both database servers:

```
systemctl start redis
systemctl start mongod
systemctl enable redis
systemctl enable mongod
```

To check if redis is installed correctly and running we run this command:

```
redis-cli ping
```
Redis will answer with PONG if all is fine.

Now, we can create our database users. Rather than leave blank passwords/open access, it's recommended to set a username and password to secure access.
For Redis, some final tuning:

Reopen /etc/redis.conf and set appendonly to yes where you find it.

```
sudo vi /etc/redis.conf
```

To learn more about optimizing redis please read [here](https://redis.io/topics/persistence)

One more optimization the creators of redis recommend is tuning the VM's memory usage. They suggest changing /etc/sysctl.conf, but this isn't recommended.
The best place to do it is in /lib/sysctl.d/50-default.conf which is where vendor mods are supposed to be.

```
sudo vi /lib/sysctl/50-default.conf
```

and at the bottom, add two lines similar to this:

```
# Added to improve Redis performance 6/9/2022
vm.overcommit_memory = 1
```

Create the redis first user/service account:

```
[centos@<ip> ~]$ redis-cli
127.0.0.1:6379> config set requirepass <yourpass>
OK
127.0.0.1:6379> AUTH 1234567
OK
127.0.0.1:6379> exit
[centos@<ip>4 ~]$ redis-cli AUTH <yourpass>
OK
```

For MongoDB, create the first user/service account:

```
[centos@<your-ip-here> ~]$ mongosh
Current Mongosh Log ID: 62a2b482ea9f739a86ec0297
Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.5.0
Using MongoDB:          5.0.9
Using Mongosh:          1.5.0

test> use admin
switched to db admin
admin>
```

And then at the admin prompt, paste in the modified string that works for your deployment:  

```
db.createUser(
   {
     user: "<your username>",
     pwd: passwordPrompt(),   // Or  "<cleartext password>"
     roles:
       [
         { role: "readWrite", db: "config" },
         "clusterAdmin"
       ],
     authenticationRestrictions: [ {
        clientSource: ["<your IP here>"],
        serverAddress: ["<your IP also here>"]
     } ]
   }
)
```
If you did passwordPrompt() above, then enter the user's new password at the prompt and be sure to make a note of it.

Your output should resemble:

```
{ ok: 1 }
```
Edit /etc/mongod.conf and change from 127.0.0.1 to 0.0.0.0.

In the next step you have to rename the system configuration file:
```
mv config-production.json config.json
```

We can now adjust the config parameters according to our setup:
(Note: MongoDB username and password fields should be deleted, if there is no authentication activated).

```
vi config.json
```

Set the parameters to suit your tastes.

Now you can run openHAB Cloud by the following command:
```
sudo node app.js
```
Errors you might receive at this point:
- 
Point your webbrowser to:
```
http://localhost:3000
```

You should be ready with your openHAB Cloud installation!


#### <a name="setupNginx"></a>Setting up Nginx ####
Optionally you can also setup Nginx as webserver:

For this we have to configure nginx as webserver and copy the openHAB Cloud nginx config, overriding the default config:
```
sudo cp /home/ubuntu/openhabcloud/etc/nginx_openhabcloud.conf /etc/nginx/sites-available/default
```

Change the following lines to match your installation.
Point `server_name` to your IP/DNS.

To obtain your iftttTestToken,

Register a test user at your openhab cloud instance.
Then enter those user credentials at the “Demo account login” section of the IFTTT API tab - Authentication section
Run a connection test by clicking on the “Begin Test” button located at the “Connection Test” section of the API tab
Click “Allow” when prompted to authorize IFTTT accessing your openhab-cloud
Once test complete, Copy the “AccessToken” from the BODY of the RESPONSE of “Request Access Token” section. This becomes your iftttTestToken for the above config.jason update
```
cd /etc/nginx/sites-enabled
sudo vi default
```

```
server {
#listen *:443;
listen *:80;
#ssl on;
# ssl_certificate /etc/nginx/ssl/YOUR-KEY.crt;
# ssl_certificate_key /etc/nginx/ssl/YOUR-KEY.key;
#ssl_certificate /etc/nginx/ssl/YOUR-CER.crt;
#ssl_certificate_key /etc/nginx/ssl/YOUR-KEY.key;

server_name YOUR-DNS-NAME;

#if ( $scheme = "http" ) {
#    rewrite ^/(.*)$    https://$host/$1 permanent;
#}

charset utf-8;

access_log /var/log/nginx/openhabcloud.org-access.log;
error_log /var/log/nginx/openhabcloud-error.log;
client_max_body_size 300m;

location /css {
    alias  /home/ubuntu/openhabcloud/public/css;
    }
location /js {
    alias /home/ubuntu/openhabcloud/public/js;
    }
location /img {
    alias /home/ubuntu/openhabcloud/public/img;
    }
location /bootstrap {
    alias /home/ubuntu/openhabcloud/public/bootstrap;
    }
location /font-icons {
    alias /home/ubuntu/openhabcloud/public/font-icons;
    }
location /fonts {
    alias /home/ubuntu/openhabcloud/public/fonts;
    }
location /js-plugin {
    alias /home/ubuntu/openhabcloud/public/js-plugin;
    }
location /staff/js-plugin {
    alias /home/ubuntu/openhabcloud/public/js-plugin;
    }
location /downloads {
    alias /home/ubuntu/openhabcloud/public/downloads;
    }
location / {
    proxy_pass http://localhost:3000;
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Real-IP $remote_addr ;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for ;
        proxy_set_header X-Forwarded-Proto https;
}

#error_page 404 /404.html;

# redirect server error pages to the static page /50x.html
#
error_page 500 502 503 504 /50x.html;
location = /50x.html {
root html;
}
}
```

You need to restart nginx:

```
sudo service nginx restart
```

## Docker compose

See [docker-compose README.md](deployment/docker-compose/README.md) for instructions on how to run openhab-cloud using docker-compose.
 
## Installing openHAB Cloud on Amazon Web Services (AWS) ##


###  Getting started with EC2 #

To install openHAB Cloud in the AWS cloud, you first need to create an account at Amazon Web Services.
Sign up for an AWS account [here](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html).

When you have created an account, you should be able to navigate to the the AWS Management Console
which is a simple web interface for managing your virtual server instances.


Make sure that your account has access to EC2 and click on Services -> EC2 link to open the EC2 dashboard.  


![AWS_1](/docs/AWS_1.png)




The EC2 dashboard will look something like the image below:




![AWS_2.png](/docs/AWS_2.png)

We now need to setup a virtual server to install and run the openHAB Cloud application on the node.
Amazon calls the virtual servers instances. The EC2 instance is similar to a regular unmanaged web-server.

#### How to create and launch the openHAB Cloud instance on AWS:


* As mentioned before select EC2 (Amazon’s Elastic Compute Cloud) from the list of services:

* In the menu bar, on the right corner you will find an item labelled “Region”:
Click to select your nearest region or chose a preferred region where you want to locate your virtual server.
We will use the [AWS Free Tier](https://aws.amazon.com/free/?nc1=h_ls), which includes services with a free tier available for 12 months following your AWS sign-up date. AWS cost varies between regions (be aware that is only free for the first year).


![AWS_4.png](/docs/AWS_4.png)

* After choosing your region click the blue “Launch Instance” button:






* Select an Amazon Machine Image (AMI) as base for openHAB Cloud node:
The AMI is a template that contains the virtual server software configuration (operating system, application server, and applications) required to launch your instance.
You can select an Amazon AMI, from the user community or you can select one of your own AMIs.

We are going to use a free tier and therefor select the Ubuntu Server, 64-bit by clicking on the blue "Select" button:

![AWS_6.png](/docs/AWS_6.png)


You will see under the Ubuntu logo that there is a free tier eligible, what we want to use.
In our guide, we are using the following AMI:
Ubuntu Server 14.04 LTS (HVM), SSD Volume Type - AMI ID: ami-fce3c696

![AWS_7.png](/docs/AWS_7.png)

Leave the default selection with t2.micro where the green label says "Free tier eligible" and click on
"Next: Configure Instance Details" to open the instance details page.

![AWS_7.png](/docs/AWS_7.png)

Just go ahead since we will use the default values on the "Configure Instance Details" page.
Click on the button "Next: Add Storage".

On the following page, you see the storage device settings for your instance.
Just continue by clicking "Next: Tag Instance", since the default storage will be fine for us.

We don't need to Tag our Instance and we continue by clicking on "Next: Configure Security Group"

In the Configure Security Group page, leave the selected option for "Create a new security group" and
add rules for protocols and ports by clicking "Add Rule" like in the image below:

![AWS_8.png](/docs/AWS_8.png)

We will add SSH to access and admin the instance (virtual server) by your console.
The next two rules a pretty straight forward and depends how you want to run openHAB Cloud on your node.
We recommend to use only HTTPS for Security reason. In this example, you also see that we added HTTP.

![AWS_8.png](/docs/AWS_8.png)

After setting up the Security Group, go ahead by clicking the blue "Review and Launch" button.

You will see a summary of your Instance Launch, which will look like this image:

![AWS_9.png](/docs/AWS_9.png)

Just hit the blue "Launch" button and you will be prompted to select or create a key pair
to connect securely to your instance.

Select the "Create a new key pair" option from the dropdown menu and
enter a name for your key pair (public and private key) and download your .pem file.
Warning: Don't loose this file, you will not be able to download it again!

![AWS_10.png](/docs/AWS_10.png)

The key file must not be publicly viewable for SSH to work.
Open a terminal window and use this command to restrict access rights to the .pem file:

```
chmod 400 YOUR-PEM-FILENAME.pem
```

Finally, under Instance you can see your instance starting up and running.


![AWS_11.png](/docs/AWS_11.png)

There you will find all the needed info to ssh into your node.
The important info is your Public DNS / Public IP address.


![AWS_13.png](/docs/AWS_13.png)

You can ssh to your instance with the user "ubuntu", see [here](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/managing-users.html) for more info on
user accounts on linux instances. In terminal window run this command with your instance infos:

```
ssh -l ubuntu -i YOUR-PEM-FILENAME.pem YOUR-AWS-EC2-PUBLIC-DNS
```

Now you should be ready with your instance and see the welcome information of ubuntu.



As next step go to back to the [Quickstart](#quickStart) and follow the installation steps.
You should jump to the [Setting up Nginx](#setupNginx) section and follow the instructions.


You are ready to start the openHAB Cloud service.

You can run openHAB Cloud as background service:
```
sudo nohup node app.js &
```

You can also use the systemd startup script under:
```
/etc/openhabcloud.service
```

You should now point your webbrowser to:
```
http://YOUR-AWS-EC2-IP
```
or
```
http://YOUR-AWS-EC2-PUBLIC-DNS
```

You should be ready with your openHAB Cloud installation!

# Release-Notes
## 1.0.5
* When upgrading from older versions, please run the `./scripts/deleteDuplicateUserDevices.js`
  script, start openhab-cloud once (and shut it down again) and then execute the following
  statement in your MongoDB collection:
  ```
  use <YOUR_DB>
  db.userdevices.reIndex()
  ```
  This is necessary to ensure a unique index on the collection.
