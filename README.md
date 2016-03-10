# LinuxServer

The public IP address of the linux server is 52.37.215.13

The page can be reached by visiting http://ec2-52-37-215-13.us-west-2.compute.amazonaws.com/

The software installed follows: Finger, GIT, Apache2, Flask, Github-Flask, virtualenv, sqlalchemy, httplib2, psycopg2, PostgreSQL, 
python-dev, libapache2-mod-wsgi

In order to meet the criteria I had to change the sshd_config to change the listing port to 2200 for ssh, and removed password 
authenticiation and root login. 

Also added catalog.conf to display the wsgi file for the web site. 

Changed the database url to create_engine('postgresql://catalog:db-password@localhost/catalog') in order to connect to postgres from it 
previously using SQLite. Adjusted this setting on database_setup.py, application.py, __init__.py, and addData.py. Because of this i had
to install psycogp2. 
