# Linux Configuration Server


Project Overview
In this project, I will be using a Linux virtual machine configurated to support the Item Catalog website. I will then deploy my application and will be learning about using the Apache HTTP Server to develop and maintain an open-source HTTP server for modern operating systems including UNIX and Windows.

Why this Project?
A deep understanding of exactly what web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer.

Learn how to access, secure, and perform the initial configuration of a bare-bones Linux server. Learn how to install and configure a web and database server and actually host a web application.

Deploying web applications to a publicly accessible server is the first step in getting users
Properly securing application ensures my application remains stable and that my user’s data is safe

Project can be viewed here: ec2-52-35-64-122.us-west-2.compute.amazonaws.com

1. Login to working environment
Install packages:
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install finger

2. Connect using ssh

Add New User
1. add new user
sudo adduser grader
pw: grader
confirm grader is created:
sudo cat /etc/passwd
sudo ls /etc/sudoers.d

giving sudo access to user
  change 90-cloud-init-users file to grader
sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader
  change ubuntu to grader
2nd way authentication user (key-based authorization)
generate key pair on local machine, not server. 
open new shell (never share private key)
  ssh-keygen
  create keygen in folder name as you like. I called mine linuxCourse
  linuxCourse.pub will be placed on server to enable Key based authorization
  
Installing Public Key (placed in remote server so ssh can use to login)
In grader environment
  sudo su - grader
  mkdir .ssh
  touch .ssh/authorized_keys
  
In new shell
  cat .ssh/linuxCourse.pub
    copy text
 
In grader Environment
  nano .ssh/authorized_keys
    paste text
Set the ssh file permissions:
  chmod 700 .ssh
  chmod 644 .ssh/authorized_keys
  
Configure the local timezone to UTC
Run sudo dpkg-reconfigure tzdata and then choose UTC

firewall

change port to 2200

disable root login

Install Apache
sudo apt-get install apache2

Install mod_wsgi
Run sudo apt-get install libapache2-mod-wsgi python-dev
Enable mod_wsgi with sudo a2enmod wsgi
Start the web server with sudo service apache2 start

Clone the Catalog app from Github
Install git using: sudo apt-get install git
cd /var/www
sudo mkdir catalog
Change owner of the newly created catalog folder sudo chown -R grader:grader catalog
cd /catalog
Clone your project from github git clone https://github.com/nguyenwinle/itemcatalog.git catalog

Create a catalog.wsgi file, then add this inside:
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'

sudo apt-get install python-pip
Install Flask pip install Flask
Install other project dependencies sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils

Install virtual environment
Install the virtual environment sudo pip install virtualenv
Create a new virtual environment with sudo virtualenv venv
Activate the virutal environment source venv/bin/activate
Change permissions sudo chmod -R 777 venv


Update path of client_secrets.json file
nano __init__.py
Change client_secrets.json path to /var/www/catalog/catalog/client_secrets.json
Configure and enable a new virtual host
Run this: sudo nano /etc/apache2/sites-available/catalog.conf
Paste this code:
<VirtualHost *:80>
    ServerName 52.35.64.122
    ServerAlias ec2-52-35-64-122.us-west-2.compute.amazonaws.com
    ServerAdmin admin@52.35.64.122
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
    ErrorLog /var/log/apache2/error.log
    LogLevel warn
    CustomLog /var/log/apache2/access.log combined
</VirtualHost>

Enable the virtual host sudo a2ensite catalog
Install and configure PostgreSQL
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
CREATE USER catalog WITH PASSWORD 'password';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
Change create engine line in your __init__.py and database_setup.py to: engine = create_engine('postgresql://catalog:password@localhost/catalog')
python database_setup.py
python __init__.py


