# Linux-Server-Configuration

### Project Description
This project consists in taking a baseline installation of a Linux server and prepare it to host your web applications. we will secure the server from a number of attack vectors, install and configure a database server, and deploy one of the existing web applications onto it.

### Server/App Info:

- IP address: 35.180.215.126

- Accessible SSH port: 2200

- Application URL: http://35.180.215.126

- Username and password for Udacity reviewer: grader student

### Configuration steps

#### Launching an AWS Lightsail Instance and connect to it via SSH
  - Launching an AWS Lightsail instance
  - The instance's security group provides a SSH port 22 by default
  - The public IP is 35.180.215.126
  - Download the private key LightsailDefaultKeyPair.pem from AWS and rename it as Lightsail_key.rsa
#### User, SSH and Security Configurations
  - Log into the remote VM as root user (ubuntu) through ssh: ```$ ssh -i Lightsail_key.rsa ubuntu@35.180.215.126```
  - Create a new user grader: ```$ sudo adduser grader.```
  - Grant udacity the permission to sudo, by adding a new file under the sudoers directory: ```$ sudo nano /etc/sudoers.d/grader.``` In the file put in:``` grader ALL=(ALL:ALL) ALL```, then save and quit.
  - Generate a new key pair by entering the following command at the terminal of your local machine.
    `$ ssh-keygen choosing the grader_key name.`
  Print the public key ``` $ cat grader_key.pub.```
  Select the public key and copy it.
  - Create a new directory called .ssnable
  h ```$ mkdir .ssh on your virtual machine.```
  - Paste the public key grader_key.pub to authorized_keys, and change the permissions:
  ```$ sudo chmod 700 /home/grader/.ssh.```
  ```$ sudo chmod 644 /home/grader/.ssh/authorized_keys.```
  - Change the owner from ubuntu to grader: ```$ sudo chown -R grader:grader /home/grader/.ssh```
  - Enforce key-based authentication, change SSH port to 2200 and disable remote login of root user:
  ```$ sudo nano /etc/ssh/sshd_config```
  - Change PasswordAuthentication to no.
  - Change Port to 2200.
  - Change PermitRootLogin to no
  ```$ sudo service ssh restart.```
  - In AWS Lightsail Networking, add 2200 as the inbound custom TCP Rule port.


#### Configure the local timezone to UTC
  - Open time configuration and set it to UTC: ```$ sudo dpkg-reconfigure tzdata.```
  - Install ntp daemon ntpd for a better synchronization of the server's time over the network connection:``` $ sudo apt-get install ntp.```

#### Update all currently installed packages
  - $ sudo apt-get update.
  - $ sudo apt-get upgrade.

#### Configure the Uncomplicated Firewall (UFW)
  Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
  $ sudo ufw default deny incoming.
  $ sudo ufw default allow outgoing.
  $ sudo ufw allow 2200/tcp.
  $ sudo ufw allow 80/tcp.
  $ sudo ufw allow 123/udp.
  $ sudo ufw enable.
  ```
  Add 3 rules above as Security Group inbound rules of AWS Lightsail instance

#### Install Apache, mod_wsgi and Git
 ```$ sudo apt-get install apache2.```
    Install mod_wsgi with the following command: ```$ sudo apt-get install libapache2-mod-wsgi python-dev.```
    Enable mod_wsgi:``` $ sudo a2enmod wsgi.
    $ sudo service apache2 start.
    $ sudo apt-get install git.```

####  Configure Apache to serve a Python mod_wsgi application. Clone the item-catalog app from Github
 ```$ cd /var/www 
    $ sudo mkdir catalog $ 
    $ sudo chown -R grader:grader catalog
    $ cd catalog
 ```
 Change user to grader and git clone the repository: 
    ```git clone https://github.com/cmartinezBoada/Build-an-item-catalog-application.git
    ```
 To make .git directory is not publicly accessible via a browser, create a .htaccess file in the .git folder and put the following in this file: RedirectMatch 404 /\.git
 
 #### Install pip , virtualenv (in /var/www/catalog)
 ```$ sudo apt-get install python-pip
    $ sudo install virtualenv
    $ sudo virtualenv venv
    $ source venv/bin/activate
    $ sudo chmod -R 777 venv
  ```
    
#### Install Flask, sqalchemy, requests, oath2client and other dependencies:
 ```$ sudo pip install Flask
    $ sudo pip install sqlalchemy
    $ sudo pip install requests
    $ sudo pip install --upgrade oath2client
 ```
 Install Python's PostgreSQL adapter psycopg2: $ sudo apt-get install python-psycopg2

#### Configure and Enable a New Virtual Host: $ sudo nano /etc/apache2/sites-available/catalog.conf
Add the following content:
```<VirtualHost *:80>
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
```


Enable the new virtual host:```$ sudo a2ensite catalog```
Create and configure the .wsgi File
```$ cd /var/www/catalog/
   $ sudo nano catalog.wsgi
   ```
Add the following content:

```import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/catalog/")

   from catalog import app as application
   application.secret_key = 'secret'
```
You need to add the major modifications include:
Rename app.py to __init__.py

#### Create a catalog.wsgi file in /var/www/catalog, then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```

#### Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`

#### Install and configure PostgreSQL
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

#### Restart Apache
  - `sudo service apache2 restart`

#### Visit site at [http://35.180.215.126](http://35.180.215.126)

### Resources: 

<https://stackoverflow.com>
