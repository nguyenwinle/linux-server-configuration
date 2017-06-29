# Linux Configuration Server


Project Overview
In this project, I will be using a Linux virtual machine configurated to support the Item Catalog website. I will then deploy my application and will be learning about using the Apache HTTP Server to develop and maintain an open-source HTTP server for modern operating systems including UNIX and Windows.

Why this Project?
A deep understanding of exactly what web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer.

Learn how to access, secure, and perform the initial configuration of a bare-bones Linux server. Learn how to install and configure a web and database server and actually host a web application.

Deploying web applications to a publicly accessible server is the first step in getting users
Properly securing application ensures my application remains stable and that my user’s data is safe

1. Login to working environment
Install packages:
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install finger

2. Connect using ssh

Add New User
1. add new user
sudo adduser grader
pw: grader
confirm grader is created:
sudo cat /etc/passwd
sudo ls /etc/sudoers.d

giving sudo access to user
  change 90-init-cloud-users file to grader
sudo cp /etc/sudoers.d/90-init-cloud-users /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader
  change ubuntu to grader
  
2nd way authentication user (key-based authorization)
generate key pair on local machine, not server. 
open new shell (never share private key)
  ssh-keygen
  create keygen in folder name as you like. I called mine linuxCourse
  linuxCourse.pub will be placed on server to enable Key based authorization
  
Installing Public Key (placed in remote server so ssh can use to login)
In grader environment
  sudo su - grader
  mkdir .ssh
  touch .ssh/authorized_keys
  
In new shell
  cat .ssh/linuxCourse.pub
    copy text
 
In grader Environment
  nano .ssh/authorized_keys
    paste text
  chmod 700 .ssh
  chmod 644 .ssh/authorized_keys
  

  
