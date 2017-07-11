# Linux Configuration Server
Project Overview

In this project, I used Amazon Web Services to support my Item Catalog application. I was able to deploy my application onto Amazon's server and learned how to use the Apache HTTP Server to develop and maintain an open-source HTTP server for modern operating systems including UNIX and Windows.

Project can be viewed [here](http://ec2-52-43-28-190.us-west-2.compute.amazonaws.com).

## Here are the following steps I took to deploy my project:

### 1. Log into working environment (for example, Ubuntu)
Install the following packages using sudo:
- sudo apt-get update
- sudo apt-get upgrade
- sudo apt-get install finger

### 2. Connect using SSH
- Create new development environment.
- Download private keys and write down your public IP address.
- Move the private key file into the folder ~/.ssh:
- mv ~/Downloads/udacity_key.rsa ~/.ssh/
- Set file rights (only owner can write and read.):
- chmod 600 ~/.ssh/udacity_key.rsa
SSH into the instance:
- ssh -i ~/.ssh/udacity_key.rsa ubuntu@52.43.28.190

### 3. Add New User named Grader
- sudo adduser grader
- pw: grader

Confirm grader is created:
- sudo cat /etc/passwd
- sudo ls /etc/sudoers.d

### 4. Giving Sudo Access to User
Change 90-cloud-init-users file to grader. 
- sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
- sudo nano /etc/sudoers.d/grader
  Change ubuntu to grader

### 5. 2nd Way Authentication User (Key-Based Authorization)
Generate key pair on local machine, not server. 
Open new shell in local machine (never share private key).
Create keygen in .ssh folder. I called mine "keygen":
- ssh-keygen
- /c/Users/nguyen le/.ssh/keygen
keygen.pub will be placed on server to enable Key based authorization

### 6. Installing Public Key (placed in remote server so ssh can use to login)
In grader environment
- sudo su - grader
- mkdir .ssh
- touch .ssh/authorized_keys
  
### 7. In new shell
- cat .ssh/keygen.pub
- copy text
 
### 8. In Grader Environment
- sudo nano .ssh/authorized_keys
- paste text

Set the SSH file permissions:
- chmod 700 .ssh
- chmod 600 .ssh/authorized_keys
  
### 9. In local machine
- ssh -i keygen grader@52.43.28.190

### 10. Configure the local timezone to UTC
- sudo dpkg-reconfigure tzdata
- Then choose UTC

### 11. Set Firewall
In AWS instance, add port tcp 2200 and port udp 123 in your networks
Change port to 2200:
- sudo nano /etc/ssh/sshd_config
- sudo service ssh restart

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):
- sudo ufw allow ssh
- sudo ufw allow 2200/tcp
- sudo ufw allow 80/tcp
- sudo ufw allow 123/udp
- sudo ufw enable

### 12. Disable root login
- sudo nano /etc/ssh/sshd_config
Change PermitRootLogin without-password line to PermitRootLogin no

Restart ssh 
- sudo service ssh restart

Now you are only able to login using:
- ssh -i keygen -p 2200 ubuntu@52.43.28.190

### 13. Install Following Packages:
Install Apache:
- sudo apt-get install apache2

Install mod_wsgi:
- sudo apt-get install libapache2-mod-wsgi python-dev

Enable mod_wsgi with:
- sudo a2enmod wsgi

Start the web server with:
- sudo service apache2 start

### 14. Clone the Catalog app from Github
Install git using: 
- sudo apt-get install git
- cd /var/www
- sudo mkdir catalog

Change owner of the newly created catalog folder:
- sudo chown -R grader:grader catalog
- cd catalog

Clone your project from github git clone:
- https://github.com/nguyenwinle/itemcatalog.git catalog

rename project.py to __init__.py
- mv project.py __init__.py

Create a catalog.wsgi file:
- sudo nano catalog.wsgi

Add the following into wsgi file:

    activate_this = '/var/www/catalog/catalog/database_setup.py'
    execfile(activate_this, dict(__file__=activate_this))
    
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'super_secret_key'

### 15. Install the Following packages:
- sudo apt-get install python-pip
- pip install Flask

Install other project dependencies:
- sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils

### 16. Install virtual environment
- sudo pip install virtualenv

Create a new virtual environment with:
- sudo virtualenv venv

Activate the virutal environment: 
- source venv/bin/activate

Change permissions: 
- sudo chmod -R 777 venv

### 17. Update path of client_secrets.json file
- nano __init__.py

Change client_secrets.json path:
- /var/www/catalog/catalog/client_secrets.json

Change fb_client_secrets.json path:
- /var/www/catalog/catalog/fb_client_secrets.json

### 18. Configure and Enable a New Virtual Host
- sudo nano /etc/apache2/sites-available/catalog.conf

Paste this code:

    <VirtualHost *:80>
        ServerName 52.43.28.190
        ServerAlias ec2-52-43-28-190.us-west-2.compute.amazonaws.com
        WSGIDaemonProcess catalog       
        python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
        WSGIProcessGroup catalog
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

Enable the virtual host:
- sudo a2ensite catalog
### 20. Install and configure PostgreSQL:
- sudo apt-get install libpq-dev python-dev
- sudo apt-get install postgresql postgresql-contrib

Log in as Postgres and use psql:
- sudo su - postgres
- psql

Create user and database in postgres:
- CREATE USER catalog WITH PASSWORD 'password';
- ALTER USER catalog CREATEDB;
- CREATE DATABASE catalog WITH OWNER catalog;
- \c catalog
- REVOKE ALL ON SCHEMA public FROM public;
- GRANT ALL ON SCHEMA public TO catalog;
- \q
- exit

Change create engine line in your __init__.py and database_setup.py to: 
- engine = create_engine('postgresql://catalog:password@localhost/catalog')
- python __init__.py (use ctrl + q to quit) 
- python database_setup.py

Install following packages: 
- pip install sqlalchemy, psycopg2, jinja2, flask, oauth2client, requests

### 21. Make sure no remote connections to the database are allowed
Check if the contents of this file:
- sudo nano /etc/postgresql/9.3/main/pg_hba.conf looks like this:


    local   all             postgres                                peer
    local   all             all                                     peer
    host    all             all             127.0.0.1/32            md5
    host    all             all             ::1/128                 md5

Restart Apache2:
- sudo service apache2 restart

View project [here](http://ec2-52-43-28-190.us-west-2.compute.amazonaws.com)

