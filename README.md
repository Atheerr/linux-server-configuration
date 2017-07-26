# Linux Server Configuration
> By Dylan Blur

![Part of the Udacity Front-End Web Development Nanodegree](https://img.shields.io/badge/Udacity-Front--End%20Web%20Developer%20Nanodegree-02b3e4.svg)

Project 6 : Linux server configuration

# Server Details
* URL: [52.56.113.173](https://52.56.113.173) or [http://ec2-52-56-113-173.compute-1.amazonaws.com](http://ec2-52-56-113-173.compute-1.amazonaws.com)

* SSH port: changed from ~~**22**~~ to **2200**

* IP adress: `52.56.113.173`

# Configuration
## Instantiate on [AWS LightSail](https://lightsail.aws.amazon.com)

1. Sign in to Amazon Lightsail using an Amazon Web Services account

2. Follow the udacity steps for creating an instance

## Connection to the instance from a local machine
The following steps outline how to connect to the instance via a terminal on a local machine.

1. Download the instance's private key by navigating to the Amazon Lightsail 'Account page'

2. Click on '**Download default key**'

3. A file with extension `.pem` will be downloaded; open it.

4. Copy the text and put it in a file called `lightrail_key.rsa` (or any other name to help you could recall anytime) in the local machine `~/.ssh/` directory

5. Run `chmod 600 ~/.ssh/lightrail_key.rsa` to give permissions to access the file

6. `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@aa.aa.aa.aa`, and aa.aa.aa.aa is the public IP. to login to the instance

## Update and Upgrade existing packages
`apt-get update` - to update the packages existing

`apt-get upgrade` - to upgrade the installed packages

If at login the message *** System restart required *** is shown, run `reboot` to restart the machine.

## Configuring the UFW (Uncomplicated Firewall)
>Note: The following commands have to be run as administrator so as an alternative to typing `sudo` before every command you can first of all run `sudo su`

0. Run `sudo ufw status` to check the status prior to the configuration

1. Changing the SSH port from 22 to 2200 (open up the /etc/ssh/sshd_config file `sudo nano /etc/ssh/sshd_config`, **change the port number on line 5 to 2200**, then **restart SSH** by running `sudo service ssh restart`

2. Run `sudo ufw default deny incoming` this is to set the ufw firewall to block everything coming in

3. Run `sudo ufw default allow outgoing` this is to to set the ufw firewall to allow everything outgoing

4. Run `sudo ufw allow ssh` to set the ufw firewall to allow SSH

5. Run `sudo ufw allow 2200/tcp` to allow all tcp connections for port 2200 so that SSH will work

6. Run `sudo ufw allow www` to set the ufw firewall to allow a basic HTTP server

7. Run `sudo ufw allow 123/udp` to set the ufw firewall to allow NTP (123/udp is actually the port for NTP)

8. Run `sudo ufw deny 22` to deny port 22 (deny this port since it will be configured to 2200)

9. Run `sudo ufw enable` to enable the ufw firewall

10. Run `sudo ufw status` to check which ports are open and to see if the ufw is active; if done correctly, it should look like this:

```
To                         Action      From
--                         ------      ----
22                         DENY        Anywhere
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
22 (v6)                    DENY        Anywhere (v6)
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
```
12. Update Amazon Lightsail firewall on the browser by changing the firewall configuration to match the internal firewall settings above (only ports 80(TCP), 123(UDP), and 2200(TCP) should be allowed)

13. Now, to login again, open up the Terminal and run:

`ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`, and XX.XX.XX.XX is the public IP address of the instance


>Note: If you try connecting without the flag `-p 2200` this will not work because the accessible port has been changed from 22 to 2200, Connecting to the instance through a browser now no longer works; this is because Lightsail's browser-based SSH access only works through port 22, which is now denied.

## Create a new user named `grader` for the reviewer

1. Run `sudo adduser grader`

2. Enter in a new UNIX password (twice) when prompted
(`pass:12345678`)

3. Fill the info for the new grader user or leave it empty

4. To switch to the grader user, run `su - grader`, and enter the password to test for accessibility

## Give grader user sudo permissions

0. Go back to the main user by typing `exit`

1. Run `sudo visudo`

3. Add the following line after to the ones that look same :`grader ALL=(ALL:ALL) ALL`

4. Save and close the visudo file

5. To verify that grader has sudo permissions, su as grader (run `su - grader`), enter the password, and run `sudo -l`; 

## Allow `grader` to log in to the virtual machine
1. Run `ssh-keygen` on the local machine

1. Choose a file name for the key pair (such as grader_key)

1. Enter in a passphrase

1. Log in to the virtual machine

1. Switch to `grader`'s home directory(`~`), and create a new `.ssh` directory `mkdir .ssh`

1. Run `touch .ssh/authorized_keys`

1. On the local machine, run `cat ~/.ssh/insert-name-of-file.pub` 

1. Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine

1. Run `chmod 700 .ssh` on the virtual machine

1. Run `chmod 644 .ssh/authorized_keys` on the virtual machine

1. Make sure key-based authentication is forced (log in as `grader`, open the `/etc/ssh/sshd_config` file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file; run `sudo service ssh restart`)

1. Log in as the grader using the following command:

	`ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX`

    `pass:12345678`

> Note that a pop-up window will ask for `grader`'s password.

## Preparing to deploy
### Change timezone to UTC
1. Check the timezone with the `date` command. This will display the current timezone after the time. 
1. If it's not UTC change it with this command: `sudo timedatectl set-timezone UTC`

### Install Apache
1. Run `sudo apt-get install apache2` to install Apache
1. Check this by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with apache title should load

### Install  mod_wsgi
1. `sudo apt-get install libapache2-mod-wsgi python-dev`
1. Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`

### Install PostgreSQL
1. run `sudo apt-get install postgresql`

### Install Python
1. Check if it's installed by running `python`
1. If not `sudo apt-get python2` for p2 or `sudo apt-get python3`

### Create a new PostgreSQL for catalog

1. PostgreSQL creates a Linux user with the name `postgres` during installation; switch to this user by running sudo `su - postgres`.

1. Connect to psql (the terminal for interacting with PostgreSQL) by running psql

2. Create the catalog user by running `CREATE ROLE catalog WITH LOGIN;` (Make sure to enter `;` without which it may NOT understand the statement)

3. Next, give the catalog user the ability to create databases: `ALTER ROLE catalog CREATEDB;` (Make sure to enter `;` without which it may NOT understand the statement)

4. Finally, give the catalog user a password by running `\password catalog`

5. Check to make sure the catalog user was created by running `\du`

6. A table will come up showing all the roles added, check on catalog role if its able to createDB

7. Exit psql by running `\q`

8. Switch back to ubuntu user through `exit`

## Create a Linux user called `catalog`

1. Create a new Linux user called catalog:

    * `sudo adduser catalog`
    * fill out information for catalog

2. Give the catalog user sudo permissions:

    * run sudo visudo
    * add catalog ALL=(ALL:ALL) ALL
    * save and close the visudo file

3. To verify that catalog has sudo permissions, login to catalog (run sudo su - catalog), and run `sudo -l`

```
 User catalog may run the following commands on
 	ip-XX-XX-XX-XX.ec2.internal:
     (ALL : ALL) ALL
```
4. Create a database called catalog by running `createdb catalog`

5. Run `psql` and from there run `\l` to see that the new database has been created

6. Switch back to the ubuntu user by running `exit`

## Install git and clone the catalog project
1. Install git by running `sudo apt-get install git`

2. Create a directory called **itemCatalog** and clone the [catalog project](https://github.com/blurdylan/item-catalog.git) by running `sudo git clone https://github.com/blurdylan/item-catalog.git itemCatalog`
    > Note that the _itemCatalog_ at the end is to stop git from adding the folder item-catalog because Apache doesn't like hyphens very much

3. Change the ownership of the 'itemCatalog' directory to ubuntu by running (while in /var/www): `sudo chown -R ubuntu:ubuntu itemCatalog/`

4. Go to the `cd /var/www/itemCatalog/itemCatalog` directory

5. run `mv main.py __init__.py` to change the name of the `main.py` file to **`__init__.py`**

6. Open `__init__.py` file and go to the last line, change from `app.run(host='0.0.0.0', port=5000)` to `app.run()`

## Adding Google Auth
1. Create a new project on the Google API Console

1. Go on credential tabs, create an auth and add http://XX.XX.XX.XX and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com as authorized JavaScript origins

2. Add http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/login and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/gconnect as authorized redirect URIs

> Where XX.XX.XX.XX and XX-XX-XX-XX is the public ip address numbers

5. Search for the words change this in login.html and base.html

5. add your clientID on the `meta tag`

## Setup Environment
> We setup this enviroment to work on the project aside without changing any global properties
1. Install python-pip `sudo apt-get install python-pip`

2. Install virtualenv with `sudo apt-get install python-virtualenv`

3. `cd` to the project folder and choose a name for the virtual env create it by running `virtualenv <ve-name>`

4. Activate the environment by running `. <ve name>/bin/activate`

4. > Note: Once you see `(<ve-name>) ubuntu@ip-XX-XX-XX-XX` you know you are on the virtualmachine

5. With the virtual environment active, install the following dependenies (note: with the exception of the libpq-dev package, make sure to not use sudo for any of the package installations as this will cause the packages to be installed globally rather than within the virtualenv):

### Install the required libraries
```
apt-get install python-psycopg2 python-flask
apt-get install python-sqlalchemy python-pip //pip and sql alchemy
pip install --upgrade oauth2client //oauth for login
pip install requests
pip install httplib2
pip install flask-seasurf
sudo apt-get install libpq-dev
```

6. To make sure everything works fine run `python __init__.py`

    It should return a line as such: `Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

7. `deactivate` is to deactivate the virtual env

## Setup Application
### Setup apache service
1. Create a configuration file (`itemCatalog.conf`) in `/etc/apache2/sites-available/` type this into it:
```
<VirtualHost *:80>
		ServerName XX.XX.XX.XX
		ServerAdmin youremail@domain.com
		WSGIScriptAlias / /var/www/itemCatalog/itemCatalog.wsgi
		<Directory /var/www/itemCatalog/itemCatalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/itemCatalog/itemCatalog/static
		<Directory /var/www/itemCatalog/itemCatalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

2. Run `sudo service apache2 reload`

### Create a wsgi file for the app
1. Create itemCatalog.wsgi in the cloned application folder

2. Edit the file with `sudo nano itemCatalog.wsgi` and add the following:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/itemCatalog')

from Itemcatalog import app as application

application.secret_key = 'secret'
```

3. `sudo service apache2 restart` to restart apache service

### Populating the database (Optional)
1. To populate the database we have to change the database engine from sqlite to postgre sql in both `db_setup.py` and `db_init.py`

2. Edit the `itemCatalog.wsgi` file or simply run the files with `python`
