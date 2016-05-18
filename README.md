# Linux Server Configuration
------



##Overview
------


This project takes a baseline installation of a Linux distribution on a virtual machine and prepares it to host a web application by installing updates, securing it from a number of attack vectors, and installing/configuring web and database servers.



##Viewing the App Live
------


The web application, [Treasure Trove](https://github.com/D-O-M/P3-Catalog-Web-App-With-OAuth2/tree/master/catalog), is hosted via HTTP at both: 

- [http://ec2-52-33-104-21.us-west-2.compute.amazonaws.com/](http://ec2-52-33-104-21.us-west-2.compute.amazonaws.com/) 

and

- [52.33.104.21](http://52.33.104.21)



##Gaining Entry to the Server
------


To gain entry into the server, you can SSH into the server by executing the command:

```
ssh -i path/to/RSA_key_file -p 2200 grader@52.33.104.21
```

Ensure the RSA file is in the right place, no password is required to login. The password for sudo commands is supplied in the Notes with my submission.



##How it was set up
------


###Launch your Virtual Machine with your Udacity account.


- Follow the instructions provided to SSH into your server

```
ssh -i ~/.ssh/udacity_key.rsa root@52.33.104.21
```



###Create a new user named grader


#####Create an SSH key in local terminal

```
ssh-keygen
```

#####Create user in remote terminal

```
adduser grader
```

#####Setup SSH key for login

  - Open file:

```
nano /home/grader/.ssh/authorized_keys
```

  - Paste in previously generated public SSH key

  - Exit and save:

    <kbd>CTRL</kbd><kbd>X</kbd> followed by <kbd>Y</kbd> and press <kbd>ENTER</kbd>



###Give the grader the permission to sudo


- Create file in `sudoers.d` directory:

```
touch /etc/sudoers.d/grader
```

- Edit newly created file:

```
nano /etc/sudoers.d/grader
```

- Add lines:

```
grader    ALL=(ALL:ALL) PASSWD:ALL
```

(`PASSWD:ALL` requires a password when `sudo` command is used)


- Exit and save:

    <kbd>CTRL</kbd><kbd>X</kbd> followed by <kbd>Y</kbd> and press <kbd>ENTER</kbd>



###Update all currently installed packages


- Enter command to update list of available packages:

```
apt-get update
```

- Enter command to upgrade installed packages:

```
apt-get upgrade
```


###Change the SSH port from 22 to 2200


- Edit `sshd_config` file:

```
nano /etc/ssh/sshd_config
```

- Change 5th line from:

```
# What ports, IPs and protocols we listen for
Port 22
```
to

```
# What ports, IPs and protocols we listen for
Port 2200
```

- Exit and save:

    <kbd>CTRL</kbd><kbd>X</kbd> followed by <kbd>Y</kbd> and press <kbd>ENTER</kbd>



###Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)


- Deny all incoming connections/requests:

```
ufw default deny incoming
```

- Allow all outgoing connections/requests:

```
ufw default allow outgoing
```

- Allow incoming SSH connections on port 2200:

```
ufw allow 2200
```

- Allow incoming HTTP requests:

```
ufw allow www
```

- Allow incoming NTP connections/requests:

```
ufw allow ntp
```

- Enable the uncomplicated firewall:

```
ufw enable
```



###Configure the local timezone to UTC


```
timedatectl set-timezone UTC
```



###Install and configure Apache to serve a Python mod_wsgi application


```
apt-get install apache2
```



###Install and configure PostgreSQL:



#####Install postgresql package:

```
apt-get install postgresql
```


#####Do not allow remote connections

Ensure port 5432 is closed as we did earlier.


#####Create a new user named catalog that has limited permissions to your catalog application database

- Switch to postgresql user:

```
su - postgres
```

- Create user named catalog with password:

```
createuser catalog -d -P
```

- Create database named catalog:

```
createdb catalog
```


#####Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser

- Install Git package:

```
apt-get install git
```

- Clone catalog app repository from project 3:

```
git clone https://github.com/D-O-M/Udacity-Fullstack/tree/master/vagrant/catalog /var/www/CatalogApp
```

- Install libraries for setting up the app:

```
apt-get install libapache2-mod-wsgi python-dev
```

- Enable mod_wsgi:

```
a2enmod wsgi
```

- Install python-pip:

```
apt-get install python-pip
```

- Use pip to install virtual environment:

```
pip install virtualenv
```

- Create a virtual environment to serve the app:

```
virtualenv /var/www/CatalogApp/catalog/CatalogEnv
```

- Enter the virtual environment:

```
source /var/www/CatalogApp/catalog/CatalogEnv/bin/activate
```

- Install Flask inside the virtual environment:

```
pip install Flask
```

- Exit/deactivate the virtual environment:

```
deactivate
```

- Create and edit a config file for the app:

```
nano /etc/apache2/sites-available/CatalogApp.conf
```

- Add the following to the newly created file:

```
<VirtualHost *:80>
                ServerName ec2-52-33-104-21.us-west-2.compute.amazonaws.com
                ServerAlias 52.33.104.21
                ServerAdmin admin@dominic.com
                WSGIScriptAlias / /var/www/CatalogApp/CatalogApp.wsgi
                <Directory /var/www/CatalogApp/catalog/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/CatalogApp/catalog/static
                <Directory /var/www/CatalogApp/catalog/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /templates /var/www/CatalogApp/CatalogApp/templates
                <Directory /var/www/CatalogApp/catalog/templates/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

- Exit and save:

    <kbd>CTRL</kbd><kbd>X</kbd> followed by <kbd>Y</kbd> and press <kbd>ENTER</kbd>

- Enable the virtual host:

```
a2ensite CatalogApp
```

- Create and edit the `.wsgi` file:

```
nano /var/www/CatalogApp/CatalogApp.wsgi
```

- Add the following to the newly created file:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/CatalogApp/")

from CatalogApp import app as application
application.secret_key = '123456789'
```

- Edit the `database_setup.py` and `__init__.py` file to use the postgresql database instead of the sqlite database originally used:

Change:

```
engine = create_engine('sqlite:///catalog.db')
```

to:

```
engine = create_engine('postgresql://catalog:password@localhost:5432/catalog')
```

in both files.

- Restart the apache server for changes to be applied:

```
service apache2 restart
```

- Reboot the entire VM to ensure all configuration changes are in effect:

`reboot`


##Resources Used
------


- [Stack Overflow](http://www.stackoverflow.com)
- [Ask Ubuntu](http://www.askubuntu.com)
- [Digital Ocean Tutorial](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- 1:1 Session with Udacity Coach (Rahul)
- [Ubuntu Docs](https://help.ubuntu.com/)
- [Postgresql Docs](http://www.postgresql.org/docs/)