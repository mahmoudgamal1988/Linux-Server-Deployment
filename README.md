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

## Step 3: Update and upgrade installed packages

```
sudo apt-get update
sudo apt-get upgrade

```

