#Linux Server configuration Project

#About the project
A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications.

IP Address: 18.185.57.126
SSH Port: 2200
URL using DNS: 18.185.57.126 

#Steps Followed to Configure the server:

1. Update all packages
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade


2. Enable automatic security updates
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades

3. Change timezone to UTC and Fix language issues
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8

4. Create a new user grader and Give him sudo access
sudo adduser grader
sudo nano /etc/sudoers.d/grader 
Then add the following text grader ALL=(ALL) ALL

5. Setup SSH keys for grader
On local machine ssh-keygen 
choose path for storing public and private keys
copy the contents of the public key
On remote machine home as user grader
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys 
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys 
nano .ssh/authorized_keys 
Then paste the contents of the public key created on the local machine.

6. Change the SSH port from 22 to 2200 | Enforce key-based authentication | Disable login for root user
sudo nano /etc/ssh/sshd_config
Then change the following:
Find the Port line and edit it to 2200. Remove the '#' if it's put before the port line.
Find the PasswordAuthentication line and edit it to no.
Find the PermitRootLogin line and edit it to no.
Save file
run sudo service ssh restart

7. Configure the Uncomplicated Firewall (UFW)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable

8. Install Apache2 and mod-wsgi for python2 and Git
sudo apt-get install apache2 libapache2-mod-wsgi git

9. Install and configure PostgreSQL
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql


Then

CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit


Note: In your catalog project you should change database engine to
engine = create_engine('postgresql://catalog:password@localhost/catalog')


10. Connect with sftp to upload project zip file
sftp -P 2200 -i grader_private_key grader@18.185.57.126 
put item_catalog.zip

11. from the grader user unzip the project file:
if not downloaded download the unzip module.
sudo apt-get install unzip
unzip item_catalog.zip

12. move project to catalog folder, unzip it and create configuration file:
cd /var/www/
sudo mkdir item_catalog
cd item_catalog
cd ~
mv item_catalog -r /item_catalog
nano catalog.wsgi
Then add the following in catalog.wsgi file

#!/usr/bin/python
import sys
sys.stdout = sys.stderr

# Add this if you'll create a virtual environment, So you need to activate it
# -------
activate_this = '/var/www/item_catalog/env/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))
# -------

sys.path.insert(0,"/var/www/item_catalog")

from app import app as application
application.secret_key = 'secret_key'


13. Setup virtual environment and Install app dependencies

sudo apt-get install python-pip
sudo -H pip install virtualenv
virtualenv env
source env/bin/activate

pip install flask packaging oauth2client redis passlib flask-httpauth
pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests

14. Configure apache server
sudo nano /etc/apache2/sites-available/udacity-project.conf
Then add the following content:

# serve catalog app
``` html
<VirtualHost *:80>
  ServerName 18.185.57.126
  ServerAdmin <Email>
  DocumentRoot /var/www/item_catalog
  WSGIDaemonProcess catalog user=grader group=grader
  WSGIScriptAlias / /var/www/item_catalog/catalog.wsgi
  <Directory /var/www/item_catalog>
    WSGIProcessGroup catalog
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
  </Directory>
  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
#Resources:

* [Ask Ubuntu](https://askubuntu.com/)
* [Stack Overflow](https://stackoverflow.com/)
* [Flask mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Apache Server Configuration Files](https://httpd.apache.org/docs/current/configuring.html)
* [Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Set Up Apache Virtual Hosts on Ubuntu ](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
* [mod_wsgi documentation](https://modwsgi.readthedocs.io/en/develop/)
* [Automatic Security Updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates#Using_the_.22unattended-upgrades.22_package)
