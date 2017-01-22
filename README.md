# FSND-p7-Linux-Server-Configuration
FSND project

This is a project for the Udacity Project: Linux Server Configuration

A baseline installation of a Ubuntu Linux disto on virtual machine (VM) that is hosting a Flask web application. Included in this project are topics such as how to accee, securing, configuration, updates, and installing and configuration of a web and database server.  And hosting it Amazon AWS.

# Requirement

Download and install the following according to your operating system
[Git](http://git-scm.com/downloads)

The Flask app of the project [here](https://github.com/trevordo/FSND-p3-catalog) was cloned remotely to the VM and can be accessed live [here](http://35.160.247.81/)

# Setup of the server environment

The following is a summary for implementing a flask app on a VM.

## Step 1: Access the Amazon AWS VM 
Source: [Udacity](https://www.udacity.com/account#!/development_environment)

1. Download the Pivate Key
2. (use your perferred terminal) Move the Private Key the ~/.ssh folder (the ~ is your environment's home directory) where Downloads is the folder location of the Key.
```sh
$ mv ~/Downloads/udacity_key.rsa ~/.ssh/
```
(use your perferred terminal) change the file permission
```sh
$ chmod 600 ~/.ssh/udacity_key.rsa
```
(use your perferred terminal) Connecting to the server for the first time 
```sh
$ ssh -i ~/.ssh/udacity_key.rsa root@35.160.247.81
```

## Create a new user, modify sudoers, add Key pairs
Source: [Udacity](https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4331066009/concepts/48010894680923#) and [DigitialOcean 1](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) and [DigitalOcean 2](https://www.digitalocean.com/community/tutorials/how-to-tune-your-ssh-daemon-configuration-on-a-linux-vps)

After connection to server is successful
1. Create a new user
```sh
$ sudo adduser grader
```
2. Enter a password
3. (Optional)Additional entry of user information
4. Modify the /etec/sudoers 
```sh
$ sudo visudo
```
5. Under "# User privilege specification' and after 'root ALL=(ALL:ALL) ALL' add 'grader ALL=(ALL:ALL) ALL'
6. Generate Public Private Key pairs and a LOCAL machine
```sh
$ ssh-keygen 
Enter file in which to save the key... Press enter to accept default directory and name 'id_rsa'
Enter passphrase
```
While still logged as root change user and install the public key on the server 
```sh
$ sudo su - grader
$ sudo mkdir .ssh
$ sudo touch .ssh/authorized_keys
```
Copy contents of Public Key (.pub) into authorized_keys file on server and save
```sh
$ sudo nano .ssh/authorized_keys
$ sudo chmod 700 .ssh
$ sudo chmod 644 .ssh/authorized_keys
```
Diable password logins and change port to 220
```sh
$ sudo nano /etc/ssh/sshd_config
```
10. Change to Port 2200 from 20
11. Find PasswordAuthentication and change 'yes' to 'no' and save
12. Add 'AllowUsers grader' to the end of the file
13. Restart ssh service
```sh
$ sudo service ssh restart
```
14. REMOVE 'root ALL=(ALL:ALL) ALL' with visudo 
15. Logging as user grader
```sh
$ ssh grader@35.160.247.81 -p2200 -i ~/.ssh/id_rsa
Enter passphrase
```

## Configure the Uncomplicated Firewall (UFW) and add rules for incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) 
Source: [Ubuntu](https://help.ubuntu.com/community/UFW)

Turn UFW on with the default set of rules:
```sh
$ sudo ufw enable
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
```

## Update all currently installed packages and configure timezone
Source: Udacity and [Timezone on Ubuntu.com](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)

1. Log in as user grader
2. Update the list of packages information and upgrade packages
```sh
$ sudo apt-get update
$ sudo sudo apt-get upgrade
```
Change the time zone by reconfiguring settings
```sh
$ sudo dpkg-reconfigure tzdata
Follow prompts and select UTC for the last prompt
```
Time synchronization using NTP
```sh
$ sudo apt-get install ntp
$ sudo nano /etc/ntp.conf
```
Replace server list with list from [pool.ntp.org](http://www.pool.ntp.org/en/) save and exit nano

# Installation of Flask Application

## Install and configure Apache
Source: [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)

Install Apache
```sh
$ sudo apt-get install apache2
```
Opening a web browser and inserting your IP-address will present the Apache success page
To serve Python apps install mod_wsgi and python-setuptools and restart server
```sh
$ sudo apt-get install python-setuptools libapache2-mod-wsgi
$ sudo service apache2 restart
```
enable mod_wsgi
```sh
$ sudo a2enmod wsgi
```

### Deploy a Flask Application on an Ubuntu VPS 
Source: [DigitialOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

Install additional packages for Apache to server Flask applications
```sh
$ sudo apt-get install libapache2-mod-wsgi python-dev
```
Create a basic Flask app by creating folder structure and __init__.py file
```sh
$ cd /var/www
$ sudo mkdir catalog
$ cd catalog and $ sudo mkdir catalog
$ cd catalog and $ sudo mkdir static templates
$ sudo nano __init__.py
```
Add the following to __init__.py
```sh
from flask import Flask  
app = Flask(__name__)  
@app.route("/")  
def hello():  
    return "Veni vidi vici!!"  
if __name__ == "__main__":  
    app.run() 
```
Install PIP, Flask and setup virtual environment
```sh
$ sudo apt-get install python-pip
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ sudo chmod -R 777 venv
$ source venv/bin/activate
$ sudo pip install Flask
$ sudo pip install psycopg2
$ sudo pip install sqlalchemy
$ sudo pip install httplib2
$ sudo pip install requests
$ sudo sudo pip install flask-seasurf
$ sudo pip install --upgrade oauth2client
$ sudo pip install sqlalchemy
```
(Optional) check downloaded packages. Some packages may not install in virtual environment, create links if necessary
Source: [Udacity](https://discussions.udacity.com/t/no-module-named-sqlalchemy-white-running-project-in-linux/203592/19)
```sh
Deactivate virtual environment
$ deactivate
List packages and check for installation
$ ls venv/lib/python2.7/site-packages/
create links if not in directory (example sqlalchemy)
ln -s /usr/lib/python2.7/dist-packages/sqlalchemy $VIRTUAL_ENV/lib/python2.7/site-packages
```
Configuring the new virtual host files
```sh
$ sudo nano /etc/apache2/sites-available/catalog.conf
```
Add the following to the new file
```sh
<VirtualHost *:80>
    ServerName PUBLIC-IP-ADDRESS
    ServerAdmin admin@PUBLIC-IP-ADDRESS
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Enable the flask site and disable the default site
```sh
$ sudo a2ensite catalog
$ sudo a2dissite 000-default
```
Create the wsgi file
```sh
cd /var/www/catalog and $ sudo vim catalog.wsgi
```
Add the following to the new file
```sh
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```
Restart Apache
```sh
$ sudo service apache2 restart
```

## Install and configure PostgreSQL:
Source: [DigitialOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04) and [PostgreSQL.org](https://www.postgresql.org/docs/9.1/static/app-createuser.html)

Install PostgreSQL
```sh
$ sudo apt-get install postgresql postgresql-contrib
```
Do not allow remote connections, the default is no but can be changed by running
```sh
$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf
```
Create a new user named catalog that has limited permissions to your catalog application database
```sh
create a linux catalog user
$ sudo adduser catalog
Enter password
Change user to postgres (superuser for PostgreSQL)
$ sudo su - postgres
Load PostgreSQL
$ psql
Create user catalog and create database
=> createuser catalog;
=> CREATE ROLE catalog PASSWORD "password same as linux catalog user" CREATEDB LOGIN;
=> CREATE DATABASE catalog WITH OWNER catalog;
=> \c catalog
Configuring the catalog database
=> REVOKE ALL ON SCHEMA public FROM public;
=> GRANT ALL ON SCHEMA public TO catalog;
View current users and roles
=> \du
Exit PostgreSQL
=> \q
```

## Install git, clone and set up your Catalog App project from your GitHub.

Install git and configure
```sh
$ sudo apt-get install git
$ git config --global user.name 'enter your name'
$ git config --global user.email 'enter your email'
```
Clone Catalog app (repo at top)
```sh
$ git clone http://GIT-REPO-ADDRESS-FOR-CLONE.git
```
Move all files downloaded to /var/www/catalog/catalog/ and delete remanents of folder
Deactivate access to github repo
```sh
$ cd /var/www/catalog/ 
$ sudo nano .htaccess
```
Add the following
```sh
RedirectMatch 404 /\.git
```
Update the app where the instance of "engine = create_engine(sql db location)" to
```sh
engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
```
Rename finalproject.py to __init__.py
```sh
$ mv finalproject.py __init__.py
```
Re-enter psql and run seminar_populate.py

## Setting up oauth for google and facebook 
Source: [Apache.org](http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html)

Go to the google and facebook developer consoles and update the redirects and origin address
```sh
Use the follow address for redirects, where xx is the IP-address
 http://ec2-XX-XX-XXX-XXX.us-west-2.compute.amazonaws.com/
```
update the Apache configuration file
```sh
$ sudo vim /etc/apache2/sites-available/catalog.conf
```
Add the following under "ServerAdmin"
```sh
ServerAlias http://ec2-XX-XX-XXX-XXX.us-west-2.compute.amazonaws.com/
```
Restart Apache
```sh
$ sudo service apache2 reload
```

## Additional toubleshooting

To change/add a user password in psql
```sh
=> \password
```
if receiving oauth error 'TypeError: <database.object at address> is not JSON serializable' fix dependency issue
```sh
$ sudo pip install werkzeug==0.8.3
$ sudo pip install flask==0.9
$ sudo pip install Flask-Login==0.1.3
```
