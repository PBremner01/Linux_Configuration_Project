 Udacity Linux Configuration Project
	Phil Bremner							04/17/2018

Project Overview
	Take a baseline installation of the AWS Lightsail Ubuntu server and configure it to host a Python – PostgresSQL web application.

		URL:   http://34.212.27.187.xip.io
		
1.	Install a new Ubuntu Linus Server from AWS Lightsail
2.	Update all installed packages
      - sudo get-apt update
      - sudo get-apt upgrade
3.	Change SSH port from 22 to 2200
     - sudo nano /etc/ssh/sshd_config    ;  change port parameter from 22 to 2200
     - sudo ufw allow 2200
     - sudo ufw  deny 22
     - sudo ufw delete deny 22
     - sudo ufw deny 22/tcp
     - On AWS, for this instance, allow port 2200
4.	Configure the UFW
     - sudo ufw allow 2200
     - sudo ufw allow 2200/tcp
     - sudo ufw allow 80
     - sudo ufw allow 123
     - sudo ufw allow www
     - sudo ufw enable
Create a new user account named grader
     - sudo adduser grader
     - sudo cat /etc/sudouer
Generating Key Pairs - 
a) generate the Key Pair locally	
	- ssh -keygen 
               b) to place the public key on the server;    as student from home;   
                                    mk dir .ssh
		       Touch .ssh/authorized_keys
			       Nano .ssh/authorized_keys  ; cut and paste the key in
                             - set file permissions
                                      chmod 700 ssh
                                      chmod 600 ssh/authorized_keys
                             - Use the usermod command to add the user to the sudo group.
                                               usermod -aG sudo grader
5.	Eliminate root user login access
      - sudo nano /etc/ssh/sshe and set PermitRootLogin prohibit-password to no
6.	Configure the local timezone to UTC
      - sudo dpkg-reconfigre tzdata
7.	Install and configure Apache to serve a Python mod_wsgi application
      - sudo get-apt install apache2
      - sudo get-apt install libache2-mod_wsgi
      - sudo nano /etc/apach2/sites-enabled/catalog.conif

        <VirtualHost *:80>
            ServerName 34.212.27.187
            ServerAlias ec2-34-212-27-187.compute-1.amazonaws.com
            ServerAdmin Phil@SAPHuddle.com
            WSGIScriptAlias / /var/www/html/catalog.wsgi
            <Directory /var/www/html/catalog/>
                Order allow,deny
                Allow from all
            </Directory>
            Alias /static /var/www/html/catalog/static
            <Directory /var/www/html/catalog/static/>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
       </VirtualHost>


 Install Postgresql
     - sudo ap-get install petgresql
      - check the remote connections
          sudo nano /etc/postgresql/9.5/main/pg_hba.conf
          
              # Database administrative login by Unix domain socket
              local   all             postgres                                peer

              # TYPE  DATABASE        USER            ADDRESS                 METHOD

              # "local" is for Unix domain socket connections only
              local   all             all                                     peer
              # IPv4 local connections:
              host    all             all             127.0.0.1/32            md5
              # IPv6 local connections:
              host    all             all             ::1/128                 md5

          - sudo service postgresql restart

8.	Create the .wsgi file in /var/www/html
    -sudo nano catalog.wsgi

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/html/catalog")
from init import app as application
application.secret_key = xxxxxxxxxxxxxxxxx'
application.config['SQLALCHEMY_DATABASE_URI'] = (
    'postgresql://'
    'catalog:password@localhost/catalog')

9.	Create a new database named catalog and create a new user named catalog with password catalog
    - log in as the postgres user; sudo us – postgres
    - create database catalog
    - create user catalog
    - alter role catalog WITH PASSWORD ‘catalog’
    - quit postgresql     ;    \q
10.	Install git
    - sudo apt-get install git
Clone and Deploy the Item Catalog Project
    - cd /var/www/html
    - sudo mkdir catalog
    - cd catalog
    - git clone https://github.com/PBremner01/Catalog.git
     - rename the application.py file to init.py
11.	 in the init.py file; change:
CLIENT_ID = json.loads(
#    open('client_secrets.json', 'r').read())['web']['client_id']
 open('/var/www/html/catalog/client_secrets.json', 'r').read())['web']['client_id']

and

#engine = create_engine('sqlite:///catalogs.db')
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')

and

        # Upgrade the authorization code into a credentials object
        oauth_flow = flow_from_clientsecrets('/var/www/html/catalog/client_secrets.json', scope='')

and

enter your app.secret_key

12.	In the catalogload2.py file change:
#engine = create_engine('sqlite:///catalogs.db')
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
13.	In the cdatabase_setup.py change:
#engine = create_engine('sqlite:///catalogs.db')
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
14.	 Recreate the clients_secret file in the Google Developers 
    - https://console.cloud.google.com/apis/credentials
    - set the Authorized JavaScript origin to: http://34.212.27.187.xip.io
    - set the Authorized redirect URIs to : http://34.212.27.187.xip.io and
                                            http://34.212.27.187.xip.io/auth2callback
    - download the client secrets file
    - copy the client_secrets.json to /var/www/html/catalog
15.	Restart the Apache with sudo service apache2 restart

16.	Go to http:// http://34.212.27.187.xip.io/  to test ….


Many thanks to :
	https://github.com/jasonally/linux-server-config/blob/master/README.md
https://github.com/bcko/Ud-FS-LinuxServerConfig-LightSail/blob/master/README.md



