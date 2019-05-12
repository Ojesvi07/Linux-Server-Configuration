# LINUX-SERVER-CONFIGURATION

In this project the linux virtual machine needs to be configured to support Item Catalog website.It includes installing updates and installing/configuring web and database servers.

- The Linux distribution is ubuntu 16.04 LTS
- Virtual Private Server is Amazon Lightsail 
- Database server is PostgreSQL
- Web application is Item_Catalog project created earlier in this nanodegree

### Information

- IP ADDRESS: 13.232.106.37
- SSH PORT : 2200
- SSH login username :grader
- URL : http://13.232.106.37.xip.io


##  Launch Virtual Machine

####  1.  Create a new instance on AmazonLightsail
  Login to amazon lightsail using your account on Amazon Web Services
 -  Create an instance 
 -  Select OS only and then select Ubuntu 16.04
 -  Choose your instance plan
 -  You can change the hostname or keep the same hostname for your instance
 -  Once the instance starts , you have the public IP.
 
 

#### 2. SSH into the server

- Download the default Key from your amazon lightsail instance
- To connect to the VPS using terminal use the following command :
` sudo ssh -i  Downloads/old.pem ubuntu@13.232.106.37 ` ,where 13.232.106.37 is the public IP of the instance and old.pem is the name of the downloaded default ssh key.

#### 3.Update all current packages

By using the following commands in terminal ,you can update as well as upgrade
 ```
sudo apt-get update
sudo apt-get upgrade
 ```
 
 #### 4. Change SSH port from 22 to 2200
 
 - Open file : ` sudo nano /etc/ssh/sshd_config`
 - Change port from 22 to 2200
 - Save the file and exit
 - Restart ssh using `sudo service ssh restart`
 
#### 5. Add port 2200 to Lightsail insatnce

- Click on Manage option of Amazon Lightsail instance and then click on Networking 
- Add port 2200 in Firewall and save .


### 6. Configure the UFW (Uncomplicated Firewall) 
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
 
 Run the following commands
-  To check status of firewall: `sudo ufw status`
- To deny all incoming traffic: `sudo ufw default deny incoming`
-  To allow all outgoing traffic:`sudo ufw default allow outgoing`
- To all port 2200,80,123 as incoming traffic:
 ```
 sudo ufw allow 2200/tcp
 sudo ufw allow www
 sudo ufw allow ntp
 ```
- To enable the changes : ` sudo ufw enabe `
-  To check status again : `sudo ufw status`


###### From terminal you can now login as :
`sudo ssh -i <private key of instance> -p 2200 ubuntu@13.232.106.37`, where 13.232.106.37 is the public IP of instance.



## Create user grader and give sudo access

- While logged in as ubuntu , create user grader with the following command: 
`sudo adduser grader`
- Enter and confirm password for grader
 #####  Give grader permission for sudo
- `nano /etc/sudoers`
- `nano /etc/sudoers.d/grader` and then type ` grader ALL=(ALL:ALL) ALL`
- save and quit the file

##  Create SSH login using ssh-keygen
- on your local machine. type `ssh-keygen`
- Enter the file in which you want to save the key
- Enter a passphrase and confirm it.
- Copy the contents of `.pub` file .
- Now login back to grader Virtual machine


- Run the following commands
```
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ sudo nano .ssh/authorized_keys
```
- Copy the contents of `.pub` file generated on local machine to above file.
- Give permissions using the following commands :
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```
- reload ssh using `sudo service ssh restart `
- Now you cand directly login using grader
` ssh  -i ~/home/user/.ssh/id_rsa grader@13.232.106.37 -p 2200` ,where /home/user/.ssh/id_rsa is the path where ssh key is saved.
  
##  Prepare to deploy your project
##### 1. Configure the local time to UTC zone using the command:
` sudo dpkg-reconfigure tzdata`
## Refrence:
[How do I change my timezone to UTC](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)


##  Install and configure Apache to serve a Python mod_wsgi application
- While logged in as grader,Install apache using : `sudo apt-get install apache2`
- Install python3 mod_wsgi package (if you have used python3 to build your project):
`sudo apt-get install libapache2-mod-wsgi-py3`
- Enable mod_wsgi using :`sudo a2enmod wsgi`

##  Install and configure PostgreSQL 

- While logged in as grader, install postgresql using: 
 `sudo apt-get install postgresql`
- Check if no remote connections are allowed: 
`sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
- Login as user postgres using :
`sudo su - postgres`
- Get into PostgreSQL shell using `psql`
- Create a new database named catalog1 using :
`postgres=# CREATE DATABASE catalog1 ;`
- Create a new user named catalog1 using :
` posstgres=# CREATE USER catalog1;`
- Set password for user catalog1 :
` postgres=# ALTER ROLE catalog1 WITH PASSWORD 'password'`
- Giver user catalog1 permission to database catalog1
`postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog1 TO catalog1;`
- Quit psql using `/q`
- Exit from user postgres using `exit`

## Reference:
[How to secure PostgreSQL on Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)


##  Install git
- Wile logged in as grader , install git using:
` sudo apt-get install git`

#### 1. Clone and setup Item_Catalog project
- Move into /var/www directory using `cd /var/www`
- Create a new folder named FlaskApp in this directory using `sudo mkdir FlaskApp`
- Now move to FlaskApp using `cd FlaskApp`
- Clone the Item_Catalog to vitural machine and name it as FlaskApp using 
` sudo git clone https://github.com/Ojesvi07/Item_Catalog.git FlaskApp`
- Move to the inner FlaskApp directory using ` cd FlaskApp`
- Now rename `SuperMartCatalog.py` to` __init__.py` using 
   `sudo mv SuperMartCatalog.py __init__.py`
- Edit `database.py` file by replacing `engine = create_engine('sqlite:///supermart.db')` with `engine=create_engine('postgresql://catalog1:password@localhost/catalog1')`
- Edit `__init__.py` file by replacing  `engine = create_engine('sqlite:///supermart.db')` with `engine=create_engine('postgresql://catalog1:password@localhost/catalog1')`


#### 2. Set up Virtual Environment and Install all the dependencies
`sudo apt-get install python3-pip`
- Install virtalenv using :
` sudo pip3 install virtualenv`
- Create a virtualenv using :
` sudo virtualenv venv`
- Activate venv using:
` sudo source venv/bin/activate`
- Now install all the dependencies: 
```
sudo pip3 install httplib2
sudo pip3 install requests
sudo pip3 install --upgrade oauth2client
sudo pip3 install sqlalchemy
sudo pip3 install flask
sudo apt-get install libpq-dev
sudo pip3 install psycopg2
```
- Run command `sudo python3 __iniy__.py ` to check if app is running
- Deactivate the virtual envinronment using : `deactivate`
  

#### 3. Configure and Enable a new Virtual host
- Give the following command to create a`.conf` file
`sudo nano /etc/apache2/sites-available/FlaskApp.conf`
- Add following lines of code to configure the virtualhost :
```
<VirtualHost *:80>
		ServerName 13.232.106.37
		ServerAlias 13.232.106.37.xip.io
		ServerAdmin ojesvi.bhat28@gmail.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Save and close the file.
- Enable virtual host using : `sudo a2ensite FlaskApp`


#### 4. Create the .wsgi file
- Move to the outer FlaskApp directory using `cd /var/www/FlaskApp`
- Create a file named flaskapp.wsgi using `sudo nano flaskapp.wsgi`
- Add the lines of code 
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'the_secretKey'
```
- Save and close
 #### 5. Restart apache
`sudo service apache2 restart`

## Refrence:
[ How to Deploy Flask Application on Ubuntu VPS ]( https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps )

## Useful commands to Debug
`sudo tail /var/log/apache2/error.log`
`sudo cat /var/log/apache2/error.log`










 



