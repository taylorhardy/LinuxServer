# LinuxServer

public Ip: 52.37.215.13
SSH PORT: 2200
Full project URL: http://ec2-52-37-215-13.us-west-2.compute.amazonaws.com/
Tasks given and method for completion:

Launch your Virtual Machine with your Udacity account

Log into your udacity account
Visit https://www.udacity.com/account#!/development_environment and press Create Development Environment.
Follow the instructions provided to SSH into your server

Download private key
Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal. mv ~/Downloads/udacity_key.rsa ~/.ssh/
Open your terminal and type in chmod 600 ~/.ssh/udacity_key.rsa
ssh -i ~/.ssh/udacity_key.rsa root@52.37.215.13


Create a new user named grader

sudo adduser grader
install finger to check user has been added apt-get install finger
finger grader

Give the grader the permission to sudo
create a grader file in sudoers.d and put in the following: 
grader ALL=(ALL) PASSWD:ALL

Find updates:sudo apt-get update
Install updates with sudo apt-get upgrade. 

Change the SSH port from 22 to 2200 

nano /etc/ssh/sshd_config change port 22 to port 2200
Change PermitRootLogin without-password to PermitRootLogin no to disallow root login
Change PasswordAuthentication from no to yes to allow grader to be able to login
add AllowUsers grader inside file to allow grade to login through SSH
save file(nano: ctrl+x, Y, Enter)
restart ssh servicesudo service ssh reload

Create SSH keys and copy to server manually:

On your local machine use ssh-keygen in terminal
save youkeygen file in your ssh directory /Users/username/.ssh/ example full file path that could be used: /Users/username/.ssh/udacity_key
login into grader account using password set during user creation ssh grader@52.37.215.13 -p 2200
Make .ssh directory with mkdir .ssh
make file to store keytouch .ssh/authorized_keys (dont forget to make authorized_keys plural)
On your local machine read contents of the public key cat .ssh/udacity_key.pub
Copy the key and paste in the file you just created in grader sudo nano .ssh/authorized_keys paste contents(ctr+v)
save file(nano: ctrl+x, Y, Enter)
Set permissions for files: chmod 700 .ssh chmod 644 .ssh/authorized_keys
Change PasswordAuthentication from yes back to no. nano /etc/ssh/sshd_config so it will only accept the key
save file(nano: ctrl+x, Y, Enter)
login with key pair: ssh grader@52.37.215.13 -p 2200 -i ~/.ssh/udacity_key


Configure the Uncomplicated Firewall (UFW)

Check UFW status to make sure its inactivesudo ufw status
Deny all incoming by default sudo ufw default deny incoming
Allow outgoing by default sudo ufw default allow outgoing
Allow SSH on port 2200 sudo ufw allow 2200/tcp
Allow HTTP on port 80 sudo ufw allow 80/tcp
Allow NTP on port 123 sudo ufw allow 123/udp
Turn on firewallsudo ufw enable (Be absolutely sure you allowed port 2200 or you will not be able to use ssh)

Configure the local timezone to UTC

run sudo dpkg-reconfigure tzdata from prompt: select none of the above. Then select UTC.

Install and configure Apache to serve a Python mod_wsgi application

sudo apt-get install apache2 Check the default apache message appears at you public IP address given during setup.
install mod_wsgi: sudo apt-get install libapache2-mod-wsgi
configure Apache to handle requests using the WSGI module sudo nano /etc/apache2/sites-enabled/000-default.conf
to be setup for when your catalog project is added add WSGIScriptAlias / /var/www/catalog/catalog.wsgi before </ VirtualHost> closing line
save file(nano: ctrl+x, Y, Enter)
Restart Apache sudo apache2ctl restart


Install git

sudo apt-get install git

install python dev and verify WSGI is enabled

Install python-dev packagesudo apt-get install python-dev
Verify wsgi is enabled sudo a2enmod wsgi


Create flask app taken from digitalocean


Configure And Enable New Virtual Host

Create host config file sudo nano /etc/apache2/sites-available/catalog.conf
paste the following:
<VirtualHost *:80>
  ServerName 52.37.215.13
  ServerAdmin admin@52.37.215.13
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
save file(nano: ctrl+x, Y, Enter)
Enable sudo a2ensite catalog


Create the wsgi file

cd /var/www/catalog
sudo nano catalog.wsgi
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'SuperSecretKey'
save file(nano: ctrl+x, Y, Enter)

sudo service apache2 restart

Clone Github Repo

sudo git clone https://github.com/taylorhardy/catalog.git

make .git inaccessible

from cd /var/www/catalog/ create .htaccess file sudo nano .htaccess
paste in RedirectMatch 404 /\.git
save file(nano: ctrl+x, Y, Enter)


install dependencies:
sudo apt-get install python-pip
source venv/bin/activate
pip install httplib2
pip install requests
sudo pip install --upgrade oauth2client
sudo pip install sqlalchemy
pip install Flask-SQLAlchemy
sudo pip install python-psycopg2


Install and configure PostgreSQL:

Install postgressudo apt-get install postgresql
install additional modelssudo apt-get install postgresql-contrib
by default no remote connections are not allowed
config database_setup.py sudo nano database_setup.py
python engine = create_engine('postgresql://catalog:db-password@localhost/catalog')
repeat for application.py(main.py)
copy your main app.py file into the init.py file mv app.py __init__.py
Add catalog user sudo adduser catalog
login as postgres super usersudo su - postgres
enter postgrespsql
Create user catalogCREATE USER catalog WITH PASSWORD 'db-password';
Change role of user catalog to creatDBALTER USER catalog CREATEDB;
List all users and roles to verify\du
Create new DB "catalog" with own of catalogCREATE DATABASE catalog WITH OWNER catalog;
Connect to database\c catalog
Revoke all rights REVOKE ALL ON SCHEMA public FROM public;
Give accessto only catalog roleGRANT ALL ON SCHEMA public TO catalog;
Quit postgres\q
logout from postgres super userexit
Setup your database schema python database_setup.py


retstart apache sudo service apache2 restart
