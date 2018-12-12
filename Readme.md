# Linux-Server-Configuration

### Project Description
This project consists in taking a baseline installation of a Linux server and prepare it to host your web applications. we will secure the server from a number of attack vectors, install and configure a database server, and deploy one of the existing web applications onto it.

### Server/App Info:

- IP address: 35.180.215.126

- Accessible SSH port: 2200

- Application URL: http://35.180.215.126

- Username and password for Udacity reviewer: grader student

### Configuration steps

1. Launching an AWS Lightsail Instance and connect to it via SSH
  - Launching an AWS Lightsail instance
  - The instance's security group provides a SSH port 22 by default
  - The public IP is 35.180.215.126
  - Download the private key LightsailDefaultKeyPair.pem from AWS and rename it as Lightsail_key.rsa
2. User, SSH and Security Configurations
  - Log into the remote VM as root user (ubuntu) through ssh: $ ssh -i Lightsail_key.rsa ubuntu@35.180.215.126
  - Create a new user grader: $ sudo adduser grader.
  - Grant udacity the permission to sudo, by adding a new file under the sudoers directory: $ sudo nano /etc/sudoers.d/grader. In the file put in: grader ALL=(ALL:ALL) ALL, then save and quit.
  - Generate a new key pair by entering the following command at the terminal of your local machine.
  1. $ ssh-keygen choosing the grader_key name.
  2. Print the public key $ cat grader_key.pub.
  3. Select the public key and copy it.
  4. Create a new directory called .ssh $ mkdir .ssh on your virtual machine.
  - Paste the public key grader_key.pub to authorized_keys, and change the permissions:
  1. $ sudo chmod 700 /home/grader/.ssh.
  2. $ sudo chmod 644 /home/grader/.ssh/authorized_keys.
  3. Change the owner from ubuntu to grader: $ sudo chown -R grader:grader /home/grader/.ssh
  - Enforce key-based authentication, change SSH port to 2200 and disable remote login of root user:
  1. $ sudo nano /etc/ssh/sshd_config
  2. Change PasswordAuthentication to no.
  3. Change Port to 2200.
  4. Change PermitRootLogin to no
  5. $ sudo service ssh restart.
  6. In AWS Lightsail Networking, add 2200 as the inbound custom TCP Rule port.


3. Configure the local timezone to UTC
  - Open time configuration and set it to UTC: $ sudo dpkg-reconfigure tzdata.
  - Install ntp daemon ntpd for a better synchronization of the server's time over the network connection: $ sudo apt-get install ntp.

4. Update all currently installed packages
  - $ sudo apt-get update.
  - $ sudo apt-get upgrade.

5. Configure the Uncomplicated Firewall (UFW)
  Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

  - $ sudo ufw default deny incoming.
  - $ sudo ufw default allow outgoing.
  - $ sudo ufw allow 2200/tcp.
  - $ sudo ufw allow 80/tcp.
  - $ sudo ufw allow 123/udp.
  - $ sudo ufw enable.
  Add 3 rules above as Security Group inbound rules of AWS Lightsail instance

6. Install Apache, mod_wsgi and Git
  - $ sudo apt-get install apache2.
  - Install mod_wsgi with the following command: $ sudo apt-get install libapache2-mod-wsgi python-dev.
  - Enable mod_wsgi: $ sudo a2enmod wsgi.
  - $ sudo service apache2 start.
  - $ sudo apt-get install git.

7.  Configure Apache to serve a Python mod_wsgi application
  - Clone the item-catalog app from Github
    1. $ cd /var/www 
    2. $ sudo mkdir catalog $ 
    3. $ sudo chown -R grader:grader catalog
    2. $ cd catalog
    3. $ change user to grader and git clone the repository: git clone https://github.com/cmartinezBoada/Build-an-item-catalog-application.git
  - To make .git directory is not publicly accessible via a browser, create a .htaccess file in the .git folder and put the following in this file: RedirectMatch 404 /\.git
  - Install pip , virtualenv (in /var/www/catalog)
    1. $ sudo apt-get install python-pip
    2. $ sudo pip install virtualenv
    3. $ sudo virtualenv venv
    4. $ source venv/bin/activate
    5. $ sudo chmod -R 777 venv
  - Install Flask, sqalchemy, requests, oath2client and other dependencies:
    1. $ sudo pip install Flask
    2. $ sudo pip install sqlalchemy
    3. $ sudo pip install requests
    4. $ sudo pip install --upgrade oath2client
  - Install Python's PostgreSQL adapter psycopg2: $ sudo apt-get install python-psycopg2
  - Configure and Enable a New Virtual Host: $ sudo nano /etc/apache2/sites-available/catalog.conf
    Add the following content:
<VirtualHost *:80>
 ServerName 35.180.215.126
 ServerAdmin grader@35.180.215.126
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


Enable the new virtual host:

$ sudo a2ensite catalog
Create and configure the .wsgi File

$ cd /var/www/catalog/
$ sudo nano catalog.wsgi
Add the following content:

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secret'
The original NHL Teams app that were developed on local machine needs some tweaks in order to be deployed on AWS. The major modifications include:
Rename app.py to __init__.py
Update the absolute path of client_secrets.json in __init__.py
Add app.secret_key for the Flask app in __init__.py
Add the code to create a dummy user in db_seed.py

8. Install Apache
  - `sudo apt-get install apache2`

9. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`


10. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  - `cd /catalog`
  - Clone your project from github `git clone https://github.com/rrjoson/udacity-item-catalog.git catalog`
  - Create a catalog.wsgi file, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```
  - Rename application.py to __init__.py `mv application.py __init__.py`

11. Install virtual environment
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate the virutal environment `source venv/bin/activate`
  - Change permissions `sudo chmod -R 777 venv`

12. Install Flask and other dependencies
  - Install pip with `sudo apt-get install python-pip`
  - Install Flask `pip install Flask`
  - Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

13. Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`

14. Configure and enable a new virtual host
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Paste this code:
  ```
  <VirtualHost *:80>
      ServerName 35.167.27.204
      ServerAlias ec2-35-167-27-204.us-west-2.compute.amazonaws.com
      ServerAdmin admin@35.167.27.204
      WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
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
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
  - Enable the virtual host `sudo a2ensite catalog`

15. Install and configure PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
  - Change create engine line in your `__init__.py` and `database_setup.py` to:
  `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
  - `python /var/www/catalog/catalog/database_setup.py`
  - Make sure no remote connections to the database are allowed. Check if the contents of this file `sudo nano /etc/postgresql/9.3/main/pg_hba.conf` looks like this:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```

16. Restart Apache
  - `sudo service apache2 restart`

17. Visit site at [http://35.167.27.204](http://35.167.27.204)

**Special Thanks to *[iliketomatoes](https://github.com/iliketomatoes)* for a very helpful README**
