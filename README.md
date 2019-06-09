 Linux-Server-Configuration 
This is the 3rd final project for "Full Stack Web Developer Nanodegree" on Udacity.

In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

IP ADDRESS: 3.121.71.187
HTTP ADDRESS: http://ec2-3-121-71-187.eu-central-1.compute.amazonaws.com

Google OAuth addresses: http://3.121.71.187.xip.io  or http://ec2-3-121-71-187.eu-central-1.compute.amazonaws.com

Tasks
Launch your Virtual Machine with your Udacity account
Follow the instructions provided to SSH into your server
Create a new user named grader
Give the grader the permission to sudo
Update all currently installed packages
Change the SSH port from 22 to 2200
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
Configure the local timezone to UTC
Install and configure Apache to serve a Python mod_wsgi application
Install and configure PostgreSQL:
Do not allow remote connections
Create a new user named catalog that has limited permissions to your catalog application database
Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!



Launch Virtual Machine
Instructions for SSH access to the instance
Download Private Key below

Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal. mv ~/Downloads/udacity_key.rsa ~/.ssh/

Open your terminal and type in chmod 600 ~/.ssh/udacity_key.rsa

In your terminal, type in ssh -i ~/.ssh/udacity_key.rsa root@3.121.71.187

 


Development Environment Information

Public IP Address

3.121.71.187


Private Key ( is not provided here. )



Create a new user named grader
sudo adduser grader
vim /etc/sudoers
touch /etc/sudoers.d/grader
vim /etc/sudoers.d/grader, type in grader ALL=(ALL:ALL) ALL, save and quit

Set ssh login using keys
generate keys on local machine usingssh-keygen ; then save the private key in ~/.ssh on local machine

deploy public key on developement enviroment

On you virtual machine:

$ su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ vim .ssh/authorized_keys
Copy the public key generated on your local machine to this file and save

$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
reload SSH using service ssh restart

now you can use ssh to login with the new user you created

ssh -i [privateKeyFilename] grader@3.121.71.187

ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@3.121.71.187

Update all currently installed packages
sudo apt-get update
sudo apt-get upgrade
Change the SSH port from 22 to 2200
Use sudo vim /etc/ssh/sshd_config and then change Port 22 to Port 2200 , save & quit.
Reload SSH using sudo service ssh restart
Configure the Uncomplicated Firewall (UFW)
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable 
Configure the local timezone to UTC
Configure the time zone sudo dpkg-reconfigure tzdata
It is already set to UTC.
Install and configure Apache to serve a Python mod_wsgi application
Install Apache sudo apt-get install apache2
Install mod_wsgi sudo apt-get install python-setuptools libapache2-mod-wsgi
sudo apt-get install libapache2-mod-wsgi python-dev
sudo apt-get install git

Enable mod_wsgi by
sudo a2enmod wsgi

Restart Apache sudo service apache2 restart

Set up the folder structure

cd /var/www
sudo mkdir catalog
sudo chown -R grader:grader catalog
cd catalog


Create a .wsgi file

sudo nano catalog.wsgi
and add the following thing into this file

    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'super_secret_key'
    
    

Install and configure PostgreSQL
Install PostgreSQL sudo apt-get install postgresql

Check if no remote connections are allowed sudo vim /etc/postgresql/9.5/main/pg_hba.conf

Login as user "postgres" sudo su - postgres

Get into postgreSQL shell psql

Create a new database named catalog and create a new user named catalog in postgreSQL shell

postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
Set a password for user catalog

postgres=# ALTER ROLE catalog WITH PASSWORD 'catalog';
Give user "catalog" permission to "catalog" application database

postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Quit postgreSQL postgres=# \q

Exit from user "postgres"

exit



Install git, clone and setup your Catalog App project.
Install Git using sudo apt-get install git
Use cd /var/www to move to the /var/www directory
Create the application directory sudo mkdir catalog
Move inside this directory using cd catalog
Clone the Catalog App to the virtual machine git clone https://github.com/arturr210br/itemcatalog.git
Rename app.py to __init__.py using sudo mv app.py __init__.py
Edit database_setup.py, website.py and functions_helper.py and change engine = create_engine('sqlite:///divisions.db') to engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
Install pip sudo apt-get install python-pip
Use pip to install dependencies wich are being used in the project
Install psycopg2 sudo apt-get -qqy install postgresql python-psycopg2
Create database schema sudo python database_setup.py

Now we need to install and start the virtual machine

 sudo pip install virtualenv
 sudo virtualenv venv
 source venv/bin/activate
sudo chmod -R 777 venv
You should see a (venv) appears before your username in the command line. 8. Now we need to install the Flask and other packages needed for this application

sudo apt-get install python-pip (If needed)
sudo pip install flask
sudo pip install httplib2
sudo pip install oauth2client
sudo pip install sqlalchemy
sudo pip install psycopg2 (sometimes it asks to install psycopg2-binary. You'll find this while executing your program)
sudo pip install requests
sudo pip install redirect
sudo pip install psslib


Use the below command to change the client_secrets.json line to /var/www/catalog/catalog/client_secrets.json

sudo nano __init__.py
and change the host to your Amazon Lightsail public IP address and port to 80 and change the last line of your program to this

    before app.run(host='0.0.0.0', port=5001)
    after  app.run()
Now we need to configure and enable the virtual host

sudo nano /etc/apache2/sites-available/catalog.conf

    <VirtualHost *:80>
        ServerName 3.121.71.187
        ServerAlias http://ec2-3-121-71-187.eu-central-1.compute.amazonaws.com
        ServerAdmin arurr210@bol.com.br
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

Enable the virtual host with the following command: sudo a2ensite catalog


sudo nano __init__.py
command to change all engine to engine = create_engine('postgresql://catalog:catalog@localhost/catalog change you engine in database_setup.py, populate_databae.py also.

Run the database_setup.py populate_databae.py and init.py by using

python database_setup.py
python populate_databae.py
python __init__.py


Restart the Apache Server by using below command

 sudo service apache2 restart
and enter your public IP address or host name into the browser. Your application should be online now!

To Enable and Disable default Apache2
After executing your programs it may give default apache page in website. To solve that issues follow below instructions

(Disable) sudo a2dissite 000-default.conf
(Enable) sudo a2ensite catalog
After performing both enable and disable operations you have to restart apache2


sudo service apache2 reload
sudo service apache2 restart
