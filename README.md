# Linux Server Configuration



## About the project
This project shows how to access, secure, and perform the initial configuration of a bare-bones Linux server. It highlights how to install and configure a web and database server as well as hosting a web application.

## IP & Hostname
- IP Address: 3.121.86.252.xip.io

##  Setting Up Amazon Lightsail

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/) using an Amazon Web Services account.
- Once you are login into the site, click `Create instance`.
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Choose a instance plan (free for a month).
- Click the `Create` button to create the instance.
- Wait for the instance to start up.

## Linux Configuration

### For Security -- part 1 --
 1. Place your downloaded private key into`.ssh` at the root of your user directory of your local machine
 2. In the terminal `$ chmod 600 ~/.ssh/[YOUR PRIVATE KEY].pem` .
 3. Log into the server `$ ssh -i ~/.ssh/[YOUR PRIVATE KEY].pem ubuntu@[YOUR PUBLIC IP address]`
 4. Update all application with the following commands
 ```
 $ sudo apt-get update
 $ sudo apt-get upgrade

 ```

### User Management
 1. Install Finger package using `sudo apt-get install finger `
 2.  Create User grader as request `sudo adduser grader`
 3.  Configure sudoer file `sudo /usr/sbin/visudo`
 4. add **grader ALL=(ALL:ALL)** below root ALL=(ALL:ALL), then exist and save the changes.
 5. Assure that grader user has granted a superuser permission , search for it in the root users by typing `cut -d: -f1 /etc/passwd`

### For Security -- part 2 --
 1. Generate key pair in local machine not on server by opening new terminal and typing ` new terminal run the command: `$ ssh-keygen -f ~/.ssh/[SSH KEY NAME]`
 2.  Copy the value of the public key from here`$ cat ~/.ssh/[SSH KEY NAME].pub`
 3. Turn back to the server terminal then `cd /home/grader`.
 4. `$ mkdir .ssh` to create ssh directly
 5. `$ touch .ssh/authorized_keys` to create authorized_keys file to store the public key
 6. `$ nano .ssh/authorized_keys`, paste the copied key from step2 , then exist and save the file
 7. Change the permission for .ssh folder
 ```
 $ chmod 700 .ssh
 $ chmod 644 .ssh/authorized_keys
 ```
 8. `$ sudo chown -R grader:grader /home/grader/.ssh` to change the owner of ssh folder
 9. restart the server `$ sudo service ssh restart`
 10. `logout` from server
 11. Login `$ ssh -i ~/.ssh/[SSH KEY NAME] grader@[IP address]`
 12. To enforce key authentication from the ssh configuration file by editing `$ sudo nano /etc/ssh/sshd_config`. Find the line that says **PasswordAuthentication** and change it to no. Also find the line that says **Port 22** and change it to **Port 2200**.
 13. Restart ssh again: `$ sudo service ssh restart`
 14. `logout` and try step "**10.**" and add **-p 2200** at the end to connect.
 ### Configure the Uncomplicated Firewall (UFW)

- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
  ```

- Turn UFW on: `sudo ufw enable`. The output should be like this:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Check the status of UFW to list current roles: `sudo ufw status`. The output should be like this:

  ```
  Status: active

  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```

- Exit the SSH connection: `exit`.

- Click on the `Manage` option of the Amazon Lightsail Instance,
then the `Networking` tab, and then change the firewall configuration to match the internal firewall settings above.

- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

- From your local terminal, run: `ssh -i ~/.ssh/[YOUR PRIVATE KEY] -p 2200 grader@[YOUR PUBLIC KEY]`


# Setup process for delopying Flask application
 Hosting this application will require the Python virtual environment, Apache with mod_wsgi, PostgreSQL, and Git.
 1. Start by installing the required software
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git
$ sudo apt-get install python-pip
```
 2. `$ sudo a2enmod wsgi` to enable mod_wsgi
 3. `$ sudo service apache2 restart`. to restart Apache

### Clone Item Catalog Project
1. Clone the item catalog and change the owner of the catalog folder
```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
$ git clone [Repository URL] catalog
```

### Install and Activate the Virtual Environment and Required Packages to Run Flask Application
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git
$ sudo apt-get install python-pip
$ sudo apt-get install python-psycopg2
$ sudo pip install Flask
$ sudo pip install oauth2client
$ sudo pip install requests
$ sudo pip install httplib2
$ sudo pip install sqlalchemy
$ sudo service apache2 restart
```

## Configure .wsgi file
1. Create file: `$ sudo touch /var/www/catalog/catalog.wsgi`
2. Add content below to this file and save:
```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/catalog/catalog/")

   from application import app as application
   application.secret_key = 'super_secret_key'
```
### Python Code Modifications
1. `nano [Application Name].py` and update the path to client_secrets.js to `/var/www/catalog/catalog/client_secrets.json`
### Apache Configuration
1. `$ sudo nano /etc/apache2/sites-available/catalog.conf` to create configuration file
```
<VirtualHost *:80>
    ServerName [IP ADDRESS].xip.io
    DocumentRoot /var/www/catalog/catalog/
    ServerAdmin admin@[IP ADDRESS]
    WSGIDaemonProcess catalog
    WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
    <Directory /var/www/catalog>
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```
2. `sudo a2ensite catalog.conf` to enable the new host
3. `sudo a2dissite 000-default.conf ` to disable the default host
4. `service apache2 reload` to reload service.

### Google Sign-in
1. Update redirect url
   Add `xio.io` after your current ips added
3. download the new client_secrets.json and copy its content
2. Update clients_secrets.json in '/var/www/catalog/catalog/ with the new URIs

### Database Configuration
Switch to PostgreSQL defualt user postgres: $ sudo su - postgres
Connect to PostgreSQL: $ psql
Create user catalog with LOGIN role: # CREATE ROLE catalog WITH PASSWORD 'password';
Allow user to create database tables: # ALTER USER catalog CREATEDB;
Create database: # CREATE DATABASE catalog WITH OWNER catalog;
Connect to database catalog: # \c catalog
Revoke all the rights: # REVOKE ALL ON SCHEMA public FROM public;
Grant access to catalog: # GRANT ALL ON SCHEMA public TO catalog;
Exit psql: \q 10.Exit user postgres: exit
3. Update database_setup.py , data_seeder.py or items.py in my case and catalog.py with the new database
`postgresql://username:password@localhost/catalog`
### Run the Application
1. `$ nano database_setup.py
2.  `nano items.py`

### References
  -[Stackoverflow](https://stackoverflow.com/)
  - [Udacity](https://classroom.udacity.com/me)
  - [Apache Docs](https://httpd.apache.org/docs/)
