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
1. Create your Ubuntu Linux server instance on Amazon Lightsail after signing up your Amazon account. After creating your instance, you get the public IP address of the instance.
### Step 2. Secure your server
1. Download [private key](https://lightsail.aws.amazon.com/ls/webapp/account/keys) (You can find the file Download dependency in your machine.)
2. On Amazon Lightsail Networking tab and firewall section, add 2 custom applications with port range 2200 and 123 using TCP protocol.
3. Open terminal and run `ssh -i ~/Downloads/LightsailDefaultPrivateKey-ap-southeast-1.pem ubuntu@54.255.130.126 -p 22`
4. SSH in, update all currently installed packages with the command `sudo apt-get update`.
5. Check current firewall status with `sudo ufw status`
6. Deny all incoming with `sudo ufw default deny incoming`
7. Allow all outgoing with `sudo ufw default allow outgoing`
8. Allow port 2200 with `sudo ufw 2200`
9. Allow HTTP(port 80) `sudo ufw allow 80/tcp`
10. And TCP www with `sudo ufw www`
11. Lastly NTP(port 123) with `sudo ufw allow ntp`
12. Enabled changes `sudo ufw enable`
13. Check firewall status `sudo ufw status`

Go and open file called sshd_config running `sudo nano /etc/ssh/sshd_config`
Change port 22 to 2200
Change PermitRootLogin from without-password to prohibit-password
Restart SSH Service `service sshd restart`

### Step 3. Give `grader` access
1. Create a new user account called `grader`
2. Run `sudo adduser grader`
3. Run `sudo nano /etc/sudoers.d` to make file and giver permmission for sudo
4. Inside the file, write
`# CLOUD_IMG: This file was created/modified by the Cloud Image build process
[user-name] ALL=(ALL) NOPASSWD:ALL`
5. Save and quit
6. Make sure if grader has sudo `sudo cat /etc/passwd`

#### Step 3-1. Generate key pair
1. Type locally `ssh-keygen`
2. Enter file location `~/.ssh/key*` You can name the directory ewhatever you want.
3. Enter passphrase you want twice

#### Step 3-2. Create public key to place on your remote server to allow SSH login
1. Login into grader account using `ssh -v grader@[public_IP_address] -p 2200`
2. Type password
3. Make a directory `mkdir .ssh`
4. Make another file to save all the public key `touch .ssh/authorized_keys`
5. Go inside and copy the public key `sudo nano .ssh/authorized_keys`
6. Set permissions `chmod 700 .ssh`
7. And for authorised keys `chmod 644 .ssh/authorized_keys`
8. Run `sudo service ssh restart`
9. Open new terminal and run `ssh grader@54.255.130.126 -p 2200`
10. Fill in your password in the popped up window

### Step 4. Prepare to deploy your project
### Step 4-1. Configuration Timezone to UTC
1. Check timezone `date`
2. Run `sudo dpkg-reconfigure tzdata`
3. On shell GUI select `None of the above` and select `UTC`
#### Resource: [Configuration timezone](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)

### Step 4-2. Install Apache
1. Install Apache `sudo apt-get install apache2`
2. Go `http://54.255.130.126` to find apache ubuntu default page
3. Install mod_wsgi to serve a Python mod_wsgi application using `sudo apt-get install libapache2-mod-wsgi`

### Step 4-3. Configure and enable Apache
1. Go and edit 000-default.conf fiel `sudo nano /etc/apache2/sites-available/catalog.conf`
2. Write the following lines:
```
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
```
#### Resource: [Deploy Flask app on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)

### Step 4-4. Create .wsgi file
1. Go and create `sudo nano /var/www/catalog/catalog.wsgi`
2. Inside the .wsgi file, write down
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
3. Check the structural tree with `tree`
4. Restart Apache2 with `sudo service apache2 restart`

#### Resource: [Deploy Flask app on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)

### Step 4-5. Install PostgreSQL
1. Login and check packages `sudo apt-get update`
2. Install PostgreSQL with `sudo aptget-install postgresql postgresql-contrib`
3. As a default setting, PostgreSQL is not allowed to connect remotely.
4. Check if remote connection is not allowed `sudo nano  /etc/postgresql/9.5/main/postgresql.conf`
5. In the file, find `listen_addresses=‘localhost'`

### Step 4-6. Create a new user
1. Run to login Postgres with postgres accoutnt `sudo -i -u postgres`
2. Create new user with `CREATE USER catalog WITH PASSWORD ‘your password’;`
3. Go to postgres shell `psql`
4. Check if user name ‘catalog’ is created `\du`
5. Give the permission to the user with `ALTER USER catalog CREATEDB;`
6. Check the user's permission `\du`
7. Create database with `CRETE DATABASE catalog WITH OWNER catalog;`
8. Check the list of databases `\l`
9. Restart postgresql `sudo service postgresql restart`
#### Resource: [How to secure PostgreSQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

### Step 4-7. Install Git
1. install git with `sudo apt-get install git`
2. Setup username `git config --global user.name "your-user-name"`
3. Setup password `git config --global user.name "your-password"`
4. Check all of the configuration items `git config --list`
Resource: [Setup Git](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04)

### Step 5. Deploy the project
### Step 5-1. Clone your project repository
1. Go and run `cd /var/www/catalog/`
2. Clone your project respositry `git clone (your-project-url)`

### Step 5-2. Run the application
1. Go to `/var/www/catalog/Item-Catalog/project`
2. Create the database schema `python finalProjectDatabase_setup.py`
3. Restart Apache `sudo service apache2 restart`
4. Open a browser and type to find your project `http://54.255.130.126`
5. You might find many bugs running the  application.
To find error messages, run commands `cat /var/log/apache2/error.log` or `cat /var/log/apache2/access.log`

### Step 5-3. Engine configuration
1. Go to finalProjectDatabase_setup.py file with `sudo nano /var/www/catalog/Item-Catalog/project/finalProjectDatabase_setup.py`
2. Find create_engine written for SQLite configuration and replace it `create_engine('postgresql://catalog:catalog@localhost/catalog')`
#### Resource: [Engine Configuration](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html#postgresql)

### Step 5-4. Client secrets configuration
1. Go to [Google Cloud Platform](https://console.cloud.google.com/apis/credentials)
2. And Find OAuth 2.0 client IDs to find Authorized JavaScript origins to add the origin URL `http://54.255.130.126`
3. Under your project files, find client_secrets.json and add `http://54.255.130.126`
4. Go to auth.py file `sudo nano /var/www/catalog/Item-Catalog/project/auth.py`
5. Change the client_secret path in the flow_from_clientsecrets to `oauth_flow = flow_from_clientsecrets('var/www/catalog/Item-Catalog/client_secrets.json'`

### Step 5-5. Install modules and packages
1. Before installing modules and packages, install python-pip with `sudo apt-get  install python-pip`
2. Install virtualenv `sudo pip install virtualenv`
3. You might see error message such as `locale.Error: unsupported locale setting`
4. In that case, follow [here](https://stackoverflow.com/questions/36394101/pip-install-locale-error-unsupported-locale-setting)
5. Install Flask `sudo pip install Flask`
6. Install AQLAlchemy `sudo pip install SQLAlchemy`
7. Install httplib2 `sudo pip install httplib2`
8. Install Jinja2 `sudo pip install Jinja2`
9. Install oauth2client `sudo pip install oauth2client`
10. Install psycopg2 `sudo pip install psycopg2`

## Third-party resources
[finger](http://manpages.ubuntu.com/manpages/precise/man1/finger.1.html)

## References
* [Configuration timezone](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
* [Deploy Flask app on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)
* [Setup Git](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04)
* [Solve locale setting](https://stackoverflow.com/questions/36394101/pip-install-locale-error-unsupported-locale-setting)
* [Google Cloud Platform](https://console.cloud.google.com/apis/credentials)
* [Engine Configuration](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html#postgresql)
* [How to secure PostgreSQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
