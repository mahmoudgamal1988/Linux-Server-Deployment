# Linux-Server-Deployment
This is the last project in the Udacity Full Stack Nanodegree, it is for linux server deployment configs.

## Server Details

1- This is an amazon lightsail virtual machine hosting a linux OS.

2- the IP for this server is 35.156.112.27.



## Step 1: Create an instance on amazon lightsail.
- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Create Instance.
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Name the new machine or keep the current name.

## Step 2: Create a SSH session via putty and ssh to the virtual machine.

- Use Putty and make a ssh session.
- Use the machine's IP and use port 22 for now, will be changed later.
- at Connection/Data enter the user name `ubuntu`.
- at SSH/AUTh attach the key file and press enter to login to the server.

## Step 3: Update and upgrade installed packages.

```
sudo apt-get update
sudo apt-get upgrade
```

## Step 4: Change SSH port to be 2200.

- use the command `sudo nano /etc/ssh/sshd_config`
- Find the Port 22 line.
- Change the port to be 2200.
- Exit and save the changes.
- Restart the SSH, `sudo service ssh restart`.

## Step 5: Configure the UFW.

- Verify the status of the Firewall, `sudo ufw status`.
- Deny all the incoming connections, `sudo ufw default deny incoming`.
- Allow all the outgoing connections, `sudo ufw default allow outgoing`.
- Allow port 2200 for ssh, `sudo ufw allow 2200/tcp`.
- Allow port 80 for http, `sudo ufw allow`.
- Allow port 123, `sudo ufw allow 123/udp`.
- Deny port 22, `sudo ufw deny 22/tcp`.
- Enable the uFW, `sudo ufw enable`.

## Step 6: Apply some security by installing fail2ban

- `sudo apt-get install fail2ban`.
- `sudo apt-get install sendmail iptables-persistent`.
- `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
- `sudo nano /etc/fail2ban/jail.local`
  ```
  destemail = useremail@domain
  action = %(action_mwl)s 
  ```
- Under `[sshd]` change `port = ssh` by `port = 2200`.
-`sudo service fail2ban restart`.

## Step 7: Set the automatic updates.

- `sudo apt-get install unattended-upgrades`.
- `sudo nano /etc/apt/apt.conf.d/50unattended-upgrades`, uncomment the line `${distro_id}:${distro_codename}-updates` and save it.
- Enable it: `sudo dpkg-reconfigure --priority=low unattended-upgrades`.

## Step 8: Update & Upgrade packages.
```
sudo apt-get update
sudo apt-get upgrade
```

## Step 9: Create Grader user.

- `sudo adduser grader`
- set password to root.
- add the new user to the sudoers file so he will be able to excute the command `sudo`, 
  `sudo visudo`
  add the below to the file
  `grader  ALL=(ALL:ALL) ALL`
  exit & save.
- switch to the grader user, `sudo su - grader`
- create a new folder called .ssh. `mkdir .ssh`.
- Create a new file, `touch /.ssh/authorized_keys`.
- add the public key into this file, ` sudo cat >> /.ssh/authorized_keys`, paste the key and press CTRL+D.

## Step 10: install apache2 and the wsgi.

```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3
sudo a2enmod wsgi
```

## Step 11: Install and configure PostgreSQL.

```
sudo apt-get install postgresql
sudo su - postgres
psql
postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
postgres=# ALTER ROLE catalog CREATEDB;
\q
exit
```
i had given a sudo access to the catalog user.

- While logged in as `catalog`, create a database: `createdb catalog`.


## Step 12: Clone the Item Catalog project.

- While logged in as grader, create the folder catalog, `mkdir /var/www/catalog`.
- install git, `sudo apt-get install git`
- Cd to that folder and clone the repo, `sudo git clone https://github.com/mahmoudgamal1988/Item-Catalog.git catalog`.
- Cd to the new created catalog folder and rename the `application.py` o `__init__.py` by `mv application.py __init__.py`.
- In the __init__.py change the line `app.run(host="0.0.0.0", port=8000, debug=True)` to `app.run()`
- in the files database_setup.py, data.py & __init__.py change the engine line to `engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`.
- create a new client_secret via google oauth and change the content of the current file with the new one and change the client ID in the login.html to the new provided key, and the same has been done for the FB login.


## Step 13: Install the virtual environment.

```
sudo apt-get install python3-pip
sudo apt-get install python-virtualenv
```

- Change to the `/var/www/catalog/catalog/` directory.
```
sudo virtualenv -p python3 venv3
sudo chown -R grader:grader venv3/
. venv3/bin/activate
```

## Step 14: install some needed packges.

```
 pip install httplib2
 pip install requests
 pip install --upgrade oauth2client
 pip install sqlalchemy
 pip install flask
 sudo apt-get install libpq-dev
 pip install psycopg2
 ```
 
 ## Step 15: Create and enable a virtual host
 
 - Add the following line in `/etc/apache2/mods-enabled/wsgi.conf` file 
to use Python 3.

  ```
  #WSGIPythonPath directory|directory-1:directory-2:...
  WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages
  ```

- Create `/etc/apache2/sites-available/catalog.conf` and add the 
following lines to configure the virtual host:

  ```
  <VirtualHost *:80>
	  ServerName 13.59.39.163
    ServerAlias ec2-13-59-39-163.us-west-2.compute.amazonaws.com
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

- Enable virtual host: `sudo a2ensite catalog`.
- Reload Apache: `sudo service apache2 reload`.

## Step 16: Set up the flask app.

- Create `/var/www/catalog/catalog.wsgi` file add the following lines:

  ```
  activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
  with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/catalog/")
  sys.path.insert(1, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = "..."
  ```
  
  ## Step 17: set uo the database scheme and populate it.
  
  - Edit `/var/www/catalog/catalog/data.py`.
- Replace `lig.random_para(250)` by `lig.random_para(240)` on lines 86, 143, 191, 234 and 280.

- Add the these two lines at the beginning of the file.

  ```
  import sys
  sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.5/site-packages") 
  ```

- Add the following lines under `create_db()`.

  ```
  # Create database.
  create_db()

  # Delete all rows.
  session.query(Item).delete()
  session.query(Category).delete()
  session.query(User).delete()
  ```

- From the `/var/www/catalog/catalog/` directory, 
activate the virtual environment: `. venv3/bin/activate`.
- Run: `python data.py`.

## Step 18: Prepare the app to be launched.

- Disable the default Apache site: `sudo a2dissite 000-default.conf`. 
The following prompt will be returned:

  ```
  Site 000-default disabled.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Reload Apache: `sudo service apache2 reload`.

- Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
- Restart Apache again: `sudo service apache2 restart`.



