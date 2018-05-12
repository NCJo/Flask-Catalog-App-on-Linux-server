# Flask-Catalog-App-on-Linux-Server

#### This fullstack web application was developed using:
* Flask
* Javascript
* Python2
* Bootstrap
* HTML/CSS


#### Note to reviewer
1. clone or download "private key"
```
git clone https://github.com/NCJo/Flask-Catalog-App-on-Linux-server.git
```
2. public IP address: 18.236.184.190
3. SSH post: 2200
4. Click [this link](http://18.236.184.190/) to get to the main page!

5. Type the following into the terminal
```
ssh -i <path_to_key> grader@18.236.184.190 -p 2200
```

#### The configuration summary
1. Change port from 22 to 2200
2. Configuring Firewall to allow SSH, http, and ntp
3. Change local timezone to UTC
4. Install Apache
5. Install PostgreSQL
6. Configure database
7. add Grader user and grant the user sudo permission
8. Installed Python2 and all its dependencies

#### Useful resources!
1. [How to figure SSH key-based authentication on a linux server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
2. [How to deploy a flask application on an Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
3. [How to set up amazon lightsail](https://cloudacademy.com/blog/how-to-set-up-your-first-amazon-lightsail/)
4. [How to configure SSH key-based Authentication on a Linux Server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)

#### Steps
##### Log in to the remote ubuntu with terminal

```
ssh -i ~/.ssh/<amazon-default-key-that-you downloaded-from-account> ubuntu@<ip address>> -p 22
```
##### Create new user <newuser>
* ```sudo adduser newuser```
    * add password for later use
* use finger to verify new user created
* ```sudo apt-get install finger```
* ```finger newuser```
##### Give <newuser> permission to sudo
* `sudo visudo`
* add this line to the file `newuser ALL=(ALL:ALL) ALL` below "#User privilege specification"
* save and exit
* Add newuser with `sudo nano /etc/sudoers.d/newuser` and add line `newuser ALL=(ALL:ALL) ALL`
* Add root with `sudo nano /etc/sudoers.d/root` and add line `root ALL=(ALL:ALL) ALL`
##### Update the current installed packages
* find update: `sudo apt-get update`
* install new updates: `sudo apt-get upgrade`
##### How to change SSH port from 22 to 2200
* `nano sudo /etc/ssh/sshd_config` change line port 22 to port 2200
* to disallow root login: `PermitRootLogin no`
* IMPORTANT: make `PasswordAuthentication yes`
    * it's for newuser to be able to set up SSH on the next section
* IMPORTANT: restart ssh service `sudo service ssh reload`
* make sure to allow port 2200 in amazon lightsail website
##### Configure SSH keys
* on local machine, create SSH keys: `ssh-keygen`
    * add password when generate keygen
* login to newuser using the password: `ssh -v newuser@<ip> -p 22000`
* VERY IMPORTANT STEPS: as mess up the ownership of folders and file create in these step will lead to newuser cannot using ssh to log in
    * make ssh directory`mkdir .ssh`
    * file to store pub key `touch .ssh/authorized_keys`
    * on local machine copy your content inside your pub key
    * using `nano .ssh/authorized_keys` and paste the copied content inside of the file
    * IMPORTANT: set permission: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
    * `nano /etc/ssh/sshd_config` and change `PasswordAuthentication` to no
    * may need to reboot server
    * login with ssh keypair: `ssh newuser@<ip> -p 2200 -i ~/.ssh/newuser.key`
##### Configure Firewall
* use `sudo ufw status` to check that firewall is inactive
* `sudo ufw default deny incoming`
* `sudo ufw default allow outgoing`
* allow ssh: `sudo ufw allow ssh`
* allow ssh on port 2200 `sudo ufw allow 2200/tcp`
* allow http on port 80 `sudo ufw allow 80/tcp`
* allow ntp on port 123 `sudo ufw allow 123/udp`
* turn on firewall: `sudo ufw enable`
##### Change local TZ to UTC
* `sudo dpkg-reconfigure tzdata` then select UTC
##### Install Apache to serve Python mod_wsgi
* check the browser for 'it works! from your IP after `sudo apt-get install apache2`
* install `sudo apt-get install libapache2-mod-wsgi`
* configure Apache to handle requests using WSGI: `sudo nano /etc/apache2/sites-enabled/000-default.conf`
* add `WSGIScriptAlias / /var/www/html/myapp.wsgi` between `<VirtualHost>` and `</VirtualHost>` at the last line
* save and exit
* IMPORTANT: restart Apache `sudo apache2ctl restart`
##### Deploy Flask
* `cd /var/www` -> `sudo mkdir catalog`
* `cd catalog`
* `sudo mkdir catalog` -> `cd catalog`
* IMPORTANT put all your Flask contents in `/var/www/catalog/catalog/`
##### Python dependencies installation
* `sudo apt-get install python-pip`
* OPTIONAL: `sudo pip install virtualenv`
    * `sudo virtualenv venv`
    * `sudo chmod -R 777 venv`
    * `source venv/bin/activate`
    * `sudo pip instal Flask`
    * Use `deactivate` to get out of virtualenv
##### Configure and enable new virtual host
* create host config file: `sudo nano /etc/apache2/sites-available/catalog.conf`
* paste this following:
```
<VirtualHost *:80>
  ServerName 34.201.114.178
  ServerAdmin admin@34.201.114.178
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

* enable this file: `sudo a2ensite catalog`
> Create the wsgi file
* `cd /var/www/catalog`
* `sudo nano catalog.wsgi`
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
# this point to where the main.py is
sys.path.insert(0,"/var/www/catalog/catalog")

# replace 'catalog' with whatever your main.py is
from catalog import app as application
application.secret_key = 'Add your secret key'
```
* `sudo service apache2 restart`
###### Clone Git repo
* `sudo git clone https://github.com/NCJo/fullstack_catalog_webapps.git`
* IMPORTANT: get to hidden file `shopt -s dotglob`
* move git folder to /var/www/catalog/catalog/
    *   sudo mv /var/www/catalog/<git-folder>/* /var/www/catalog
    *   remove clone directory `sudo rm -r <git-folder>`
##### Make .git unaccessible to public
* cd /var/www/catalog/ create .htaccess file `sudo nano .htaccess`
* paste the following: `RedirectMatch 404 /\.git`
##### Install Flask dependencies
* OPTIONAL WITH ENV: `source venv/bin/activate`
* IMPORTANT
* `sudo pip install httplib2`
* `sudo pip install requests`
* `sudo pip install --upgrade oauth2client`
* `sudo pip install sqlalchemy`
* `sudo pip install Flask-SQLAlchemy`
* `sudo pip install python-psycopg2`
* install other packages
##### Install PostgreSQL
* `sudo apt-get install postgresql`
* install additional models `sudo apt-get install postgresql-contrib`
* IMPORTANT: make sure you got all dependencies in `models.py` and `application.py`
    * `sudo nano models.py`
    * `engine = create_engine('postgresql://catalog:db-password@localhost/catalog')`
    * `sudo nano application.py`
    * `engine = create_engine('postgresql://catalog:db-password@localhost/catalog')`
* IMPORTANT the file name must be matching with name in wsgi file
    * `sudo mv application.py catalog.py`
    * add catalog user
        * sudo adduser catalog
    * login as postgres super user `sudo su - postgres`
    * enter postgres `psql`
    * create user 'catalog' in psql
        * `CREATE USER catalog WITH PASSWORD 'db-password';`
    * change role of user catalog to createDB
        * `ALTER USER catalog CREATEDB;`
* list all users and roles to verify `\du`
* create new DB 'catalog' with own of catalog `CREATE DATABASE catalog WITH OWNER catalog;`
* connect to database `\c catalog`
* revoke all rights `REVOKE ALL ON SCHEMA public FROM public;`
* give access to only catalog role `GRANT ALL ON SCHEMA public TO catalog;`
* quit postgres `\q`
* log out from postgres super user `exit`
* set up database schema `python database_setup.py`
##### Fix OAUTH to work with hosted application
* Google wont allow the IP address to make redirects so we need to set up the host name address to be usable.
* get host name by [http://www.hcidata.info/host2ip.cgi](http://www.hcidata.info/host2ip.cgi)
* open apache configbfile `sudo nano /etc/apache2/sites-available/catalog.conf`
* below the `ServerAdmin` paste `ServerAlias YOURHOSTNAME`
* make sure the virtual host is enabled `sudo a2ensite catalog`
* restart apache server `sudo service apache2 restart`
* in your google developer console add your host name and IP address to Authorized Javascript origins. And add YOURHOSTNAME/ouath2callback to the Authorized redirect URIs.
