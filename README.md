# Linux-Server-Deployment
This is the last project in the Udacity Full Stack Nanodegree, it is for linux server deployment configs.

## Server Details

1- This is an amazon lightsail virtual machine hosting a linux OS.
2- the IP for this server is 35.156.112.27



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


## Step 12: 

