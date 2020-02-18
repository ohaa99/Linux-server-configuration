# Linux Server Configuration
This is the last project required for Udacity Full Stack Web Developer Nanodegree.
I deployed  my last project which  was a [Training Center Guide](https://github.com/ohaa99/catalog) .
I setup my project on an Ubuntu Linux server on an Amazon Lightsail instance. 
## Access information
* Public IP address: http://13.233.165.143/
* SSH port: 2200
### Get Linux instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home)
1. Create an AWS account
2. Click **Create instance** button on the home page
3. Select **Linux/Unix** platform [There are three versions . I chose ubuntu 16.04]
4. Select **OS Only** and **Ubuntu** as blueprint
5. Select an instance plan
6. Name your instance
7. Click **Create** button
### SSH into your Server
1. Download private key from the **SSH Keys** section in the **Account** section on Amazon Lightsail. The file name should be something like _LightsailDefaultPrivateKeyxxxxx.pem
2. Create a new file named **lightsail_key.rsa** under ~/.ssh folder on your local machine
3. Copy paste the **content** from downloaded private key file to **lightsail_key.rsa**
4. Set file permission as owner only : `$ chmod 600 ~/.ssh/lightsail_key.rsa`
5. SSH into the instance: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@[Your_Public_IP].
1. Run `sudo apt-get update` to update packages
2. Run `sudo apt-get upgrade` to install the latest versions of the packages.
3. Set for future updates: `sudo apt-get dist-upgrade`

###  Change the SSH port from 22 to 2200
1. Run `$ sudo nano /etc/ssh/sshd_config` to open up the configuration file
2. Change the port number from **22** to **2200** in this file
3. Save and exit the file
4. Restart SSH: `$ sudo service ssh restart`

### Configure the firewall
1. Check firewall status: `$ sudo ufw status`
2. Set default firewall to deny all incomings: `$ sudo ufw default deny incoming`
3. Set default firewall to allow all outgoings: `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 to allow SSH: `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 to allow www: `$ sudo ufw allow www`
6. Allow incoming UDP packets on port 123 to allow NTP: `$ sudo ufw allow 123/udp`
7. Close port 22: `$ sudo ufw deny 22`
8. Enable firewall: `$ sudo ufw enable`
9. Check out current firewall status: `$ sudo ufw status`
10. Update the firewall configuration on Amazon Lightsail website under **Networking**. add **ports** 80, 123, 2200.
11. Open up a new terminal and you can now ssh in via the new port 2200: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@YOUR_SERVER_PUBLIC_IP -p 2200`
### Create a new account and give it `sudo` access 
1. Create a new user **grader**:`$ sudo adduser grader`
2. `usermod -aG sudo ugrader`
### Set SSH login to the **grader** user using public-private keys
1. **On your local machine** , Create an SSH key pair for **grader** using the `ssh-keygen` tool . Save it in `~/.ssh` path
2. Deploy public key on development environment
    * On your local machine, read the generated public key
     `cat ~/.ssh/yourfile.pub`
    * On your virtual machine change into **grader** using `$ su grader` then create .ssh directory and authorized_keys file using the following commands.
   ```
      $ mkdir .ssh
      $ touch .ssh/authorized_keys
      $ nano .ssh/authorized_keys
   ```
    * Copy the public key from `~/.ssh/FILE-NAME.pub` to this _authorized_keys_ file on the remote and save
3. Run `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` on remote server to change file permission
4. Restart SSH: `$ sudo service ssh restart`
5. Now you are able to login in as grader: `$ ssh -i ~/.ssh/[Your_Keys_On_Local_Machine] -p 2200 grader@YOUR_SERVER_PUBLIC_IP`
6. You will be asked for grader's password. To unable it, open configuration file : `$ sudo nano /etc/ssh/sshd_config`
7. Change `PasswordAuthentication yes` to **no**
8. Restart SSH: `$ sudo service ssh restart`


### Configure the local timezone to UTC
1. Run `$ sudo dpkg-reconfigure tzdata`
2. Choose **None of the above** to set timezone to UTC

### Install and configure Apache
1. Install **Apache**: `$ sudo apt-get install apache2`
### Install and configure Python mod_wsgi
1. Install the **mod_wsgi** package: `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable **mod_wsgi**: `$ sudo a2enmod wsgi`
3. Restart **Apache**: `$ sudo service apache2 restart`

### Install PostgreSQL
1. Run `$ sudo apt-get install postgresql`
2. Make sure PostgreSQL does not allow remote connections
3. Open file: `$ sudo nano /etc/postgresql/10/main/pg_hba.conf`
4. Check to make sure it looks like this:
   ```
   # Database administrative login by Unix domain socket
   local   all             postgres                                peer

   # TYPE  DATABASE        USER            ADDRESS                 METHOD

   # "local" is for Unix domain socket connections only
   local   all             all                                     peer
   # IPv4 local connections:
   host    all             all             127.0.0.1/32            md5
   # IPv6 local connections:
   host    all             all             ::1/128                 md5
   ```
### Install git and clone catalog application from github
1. Run `$ sudo apt-get install git`
3. CD to www directory: `$ cd /var/www/`
4. Clone the catalog app: `$ sudo git clone GIT_PROJECT_UR `
5. Change the ownership: `$ sudo chown -R ubuntu:ubuntu yourproject/`
6. CD to `/var/www/yourproject`
7. Change file **application.py** to **__init__.py**: `$ mv application.py __init__.py`
8. Change line `app.run(host='0.0.0.0', port=8000)` to `app.run()` in `__init__.py` file

## Setup for deploying a Flask App on Ubuntu VPS
1. Install pip: `$ sudo apt-get install python-pip`
2. Install packages:
```
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
   ```

### Setup and enble a virtual host
1. Create file: `$ sudo touch /etc/apache2/sites-available/[yourproject].conf`
2. Add the following to the file:
```
   <VirtualHost *:80>
		ServerName XX.XX.XX.XX
		ServerAdmin admin@xx.xx.xx.xx
		WSGIScriptAlias / /var/www/yourproject/yourproject.wsgi
		<Directory /var/www/yourproject/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/yourproject/static
		<Directory /var/www/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```
3. Run `$ sudo a2ensite yourproject` to enable the virtual host
4. Restart **Apache**: `$ sudo service apache2 reload`

### Configure .wsgi file
1. Create file: `$ sudo touch /var/www/yourproject/yourproject.wsgi`
2. Add content below to this file and save:
```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/yourproject/")

   from __init__ import app as application
   application.secret_key = 'super_secret_key'
```
3. Restart **Apache**: `$ sudo service apache2 reload`

### Edit the database path
1. Replace engine variable in `__init__.py`, `database_setup.py` with `engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost')`

### Set up database schema
1. Run `$ sudo python database_setup.py`
2. Run `$ sudo python tvshows.py` to populate database
3. Restart **Apache**: `$ sudo service apache2 reload`
4. Now follow the link to [Your_Public_ip] **HERE** is http://13.233.165.143/ the application should be runing online
5. If internal errors occur: check the [Apache error file](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)
note : if you get **a internal server error** , run  `sudo tail /var/log/apache2/error.log
` to see the bugs and fix them.
## Third-Party Resources
1. [Amazon Lightsail Docs](https://lightsail.aws.amazon.com/ls/docs/en_us/all)
3. [Initial Server Setup with Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
4. [Apache Docs](https://httpd.apache.org/docs/2.2/configuring.html)
5. [How To Configure the Apache Web Server on an Ubuntu or Debian VPS](https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps)
6. [Flask by Example - Setting Up Postgres, SQLAlchemy, and Alembic](https://realpython.com/blog/python/flask-by-example-part-2-postgres-sqlalchemy-and-alembic/)

