# Linux Server Configuration

This project is part of the Full Stack Web Developer nanodegree from udacity. The project utilizes the [Congiguring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299).

## The purpose of this project
A deep understanding of exactly what web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, I will be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host my application need.

## Usage
Local IP address: `http://54.255.130.126`

## Prerequirements
This project requires Python 2.X (2.7.x is expected) and PostgreSQL 9.3 or latest version. [AWS account](https://aws.amazon.com/en/), [Apache](https://httpd.apache.org/), [ubuntu](https://www.ubuntu.com/) are required.

## Configuration steps
### Step 1. Get your server
Create your Ubuntu Linux server instance on Amazon Lightsail after signing up your Amazon account. After creating your instance, you get the public IP address of the instance.
### Step 2. Secure your server
Download [private key](https://lightsail.aws.amazon.com/ls/webapp/account/keys) (You can find the file Download dependency in your machine.)
On Amazon Lightsail Networking tab and firewall section, add 2 custom applications with port range 2200 and 123 using TCP protocol.
Open terminal and run `ssh -i ~/Downloads/LightsailDefaultPrivateKey-ap-southeast-1.pem ubuntu@54.255.130.126 -p 22`
SSH in, update all currently installed packages with the command `sudo apt-get update`.
Check current firewall status with `sudo ufw status`
Deny all incoming with `sudo ufw default deny incoming`
Allow all outgoing with `sudo ufw default allow outgoing`
Allow port 2200 with `sudo ufw 2200`
Allow HTTP(port 80) `sudo ufw allow 80/tcp`
And TCP www with `sudo ufw www`
Lastly NTP(port 123) with `sudo ufw allow ntp`
Enabled changes `sudo ufw enable`
Check firewall status `sudo ufw status`

Go and open file called sshd_config running `sudo nano /etc/ssh/sshd_config`
Change port 22 to 2200
Change PermitRootLogin from without-password to prohibit-password
Restart SSH Service `service sshd restart`

### Step 3. Give `grader` access
Create a new user account called `grader`
Run `sudo adduser grader`
Run `sudo nano /etc/sudoers.d` to make file and giver permmission for sudo
Inside the file, write
`# CLOUD_IMG: This file was created/modified by the Cloud Image build process
[user-name] ALL=(ALL) NOPASSWD:ALL`
Save and quit
Make sure if grader has sudo `sudo cat /etc/passwd`

#### Step 3-1. Generate key pair
Type locally `ssh-keygen`
Enter file location `~/.ssh/key*` You can name the directory ewhatever you want.
Enter passphrase you want twice

#### Step 3-2. Create public key to place on your remote server to allow SSH login
Login into grader account using `ssh -v grader@[public_IP_address] -p 2200`
Type password
Make a directory `mkdir .ssh`
Make another file to save all the public key `touch .ssh/authorized_keys`
Go inside and copy the public key `sudo nano .ssh/authorized_keys`
Set permissions `chmod 700 .ssh`
And for authorised keys `chmod 644 .ssh/authorized_keys`
Run `sudo service ssh restart`
Open new terminal and run `ssh grader@54.255.130.126 -p 2200`
Fill in your password in the popped up window

### Step 4. Prepare to deploy your project
### Step 4-1. Configuration Timezone to UTC
Check timezone `date`
Run `sudo dpkg-reconfigure tzdata`
On shell GUI select `None of the above` and select `UTC`
#### Resource: [Configuration timezone](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)

### Step 4-2. Install Apache
Install Apache `sudo apt-get install apache2`
Go `http://54.255.130.126` to find apache ubuntu default page
Install mod_wsgi to serve a Python mod_wsgi application using `sudo apt-get install libapache2-mod-wsgi`

### Step 4-3. Configure and enable Apache
Go and edit 000-default.conf fiel `sudo nano /etc/apache2/sites-available/catalog.conf`
Write the following lines:
`
<VirtualHost *:80>
                ServerName mywebsite.com
                ServerAdmin admin@mywebsite.com
                WSGIScriptAlias / /var/www/catalog/catalog.wsgi
                <Directory /var/www/catalog/Item-Catalog>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalog/Item-Catalog/project/static
                <Directory /var/www/catalog/Item-Catalog/project/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
`
#### Resource: [Deploy Flask app on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)

### Step 4-4. Create .wsgi file
Go and create `sudo nano /var/www/catalog/catalog.wsgi`
Inside the .wsgi file, write down
`
#! /usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/Item-Catalog/")
`
`
from project import app as application
application.config.from_object('config')
`
Check the structural tree with `tree`
Restart Apache2 with `sudo service apache2 restart`

#### Resource: [Deploy Flask app on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)

### Step 4-5. Install PostgreSQL
Login and check packages `sudo apt-get update`
Install PostgreSQL with `sudo aptget-install postgresql postgresql-contrib`
As a default setting, PostgreSQL is not allowed to connect remotely.
Check if remote connection is not allowed `sudo nano  /etc/postgresql/9.5/main/postgresql.conf`
In the file, find `listen_addresses=‘localhost'`

### Step 4-6. Create a new user
Run to login Postgres with postgres accoutnt `sudo -i -u postgres`
Create new user with `CREATE USER catalog WITH PASSWORD ‘your password’;`
Go to postgres shell `psql`
Check if user name ‘catalog’ is created `\du`
Give the permission to the user with `ALTER USER catalog CREATEDB;`
Check the user's permission `\du`
Create database with `CRETE DATABASE catalog WITH OWNER catalog;`
Check the list of databases `\l`
Restart postgresql `sudo service postgresql restart`
#### Resource: [How to secure PostgreSQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

### Step 4-7. Install Git
install git with `sudo apt-get install git`
Setup username `git config --global user.name "your-user-name"`
Setup password `git config --global user.name "your-password"`
Check all of the configuration items `git config --list`
Resource: [Setup Git](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04)

### Step 5. Deploy the project
### Step 5-1. Clone your project repository
Go and run `cd /var/www/catalog/`
Clone your project respositry `git clone (your-project-url)`
Current repository structure in my case

### Step 5-2. Run the application
Go to `/var/www/catalog/Item-Catalog/project`
Create the database schema `python finalProjectDatabase_setup.py`
Restart Apache `sudo service apache2 restart`
Open a browser and type to find your project `http://54.255.130.126`
You might find many bugs running the  application.
To find error messages, run commands `cat /var/log/apache2/error.log` or `cat /var/log/apache2/access.log`

### Step 5-3. Engine configuration
Go to finalProjectDatabase_setup.py file with `sudo nano /var/www/catalog/Item-Catalog/project/finalProjectDatabase_setup.py`
Find create_engine written for SQLite configuration and replace it `create_engine('postgresql://catalog:catalog@localhost/catalog')`
#### Resource: [Engine Configuration](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html#postgresql)

### Step 5-3. Client secrets configuration
Go to [Google Cloud Platform](https://console.cloud.google.com/apis/credentials)
And Find OAuth 2.0 client IDs to find Authorized JavaScript origins to add the origin URL `http://54.255.130.126`
Under your project files, find client_secrets.json and add `http://54.255.130.126`
Go to auth.py file `sudo nano /var/www/catalog/Item-Catalog/project/auth.py`
Change the client_secret path in the flow_from_clientsecrets to `oauth_flow = flow_from_clientsecrets('var/www/catalog/Item-Catalog/client_secrets.json'`

### Step 5-4. Install modules and packages
Before installing modules and packages, install python-pip with `sudo apt-get install python-pip`
Install virtualenv `sudo pip install virtualenv`
You might see error message such as `locale.Error: unsupported locale setting`
In that case, follow [here](https://stackoverflow.com/questions/36394101/pip-install-locale-error-unsupported-locale-setting)
Install Flask `sudo pip install Flask`
Install AQLAlchemy `sudo pip install SQLAlchemy`
Install httplib2 `sudo pip install httplib2`
Install Jinja2 `sudo pip install Jinja2`
Install oauth2client `sudo pip install oauth2client`
Install psycopg2 `sudo pip install psycopg2`

Your project might not work well in the current situation. The following bugs are what I met in the deploying process.

## Third-party resources
[finger](http://manpages.ubuntu.com/manpages/precise/man1/finger.1.html)

## References
[Configuration timezone](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
[Deploy Flask app on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
[Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)
[Setup Git](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04)
[Solve locale setting](https://stackoverflow.com/questions/36394101/pip-install-locale-error-unsupported-locale-setting)
[Google Cloud Platform](https://console.cloud.google.com/apis/credentials)
[Engine Configuration](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html#postgresql)
[How to secure PostgreSQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
