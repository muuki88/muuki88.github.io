---
author: wp_admin
comments: false
date: 2012-07-27 10:04:31+00:00
layout: post
link: http://mukis.de/pages/devvm-part-1-gerrit-on-ubuntu-12-04-server/
slug: devvm-part-1-gerrit-on-ubuntu-12-04-server
title: DevVM Part 1 - Gerrit on Ubuntu 12.04 Server
wordpress_id: 206
categories:
- Tutorials
---

I'm currently working on a little development VM and want to share some of my insides I gain and how I managed to get things work. The series will start with the tutorial to install Gerrit.


### What is Gerrit?


[Gerrit](https://code.google.com/p/gerrit/) provides a powerful server to integrate a code-review process in your git-driven development process. These are the main reasons I picked gerrit:



	
  * Support git as versioning system - awesome

	
  * Integration with buildservers like [jenkins](http://jenkins-ci.org/) to run test automatically and the CI-server is a part of the code review process

	
  * Great Eclipse integration with [EGit](http://wiki.eclipse.org/EGit/User_Guide#Working_with_Gerrit)




### Install Gerrit


All you need is root shell access to your server and a working internet connection (surprise!)


#### Generate gerrit2 user


First we generate a group _gerrit2_ and a user _gerrit2_ with a home directory located at _/usr/local/gerrit2_

    
    sudo addgroup gerrit2
    sudo adduser --system --home /usr/local/gerrit2 --shell /bin/bash --ingroup gerrit2 gerrit2


I use my own MySQL database instead of the integrated h2 database. You have to generate a user _gerrit2_ too and a database called _reviewdb_. On the shell you can do this via

    
    mysql --user=root -p



    
    CREATE USER 'gerrit2'@'localhost' IDENTIFIED BY 'secret';
    CREATE DATABASE reviewdb;
    ALTER DATABASE reviewdb charset=latin1;
    GRANT ALL ON reviewdb.* TO 'gerrit2'@'localhost';
    FLUSH PRIVILEGES;
    exit;


Last thing to do as a root is to generate a default config file for gerrit. When

    
    sudo touch /etc/default/gerritcodereview


and insert with a editor of your choice

    
    GERRIT_SITE=/usr/local/gerrit2


Now we log into our gerrit2 user and install gerrit.

    
    sudo su gerrit2
    cd ~
    wget http://gerrit.googlecode.com/files/gerrit-2.4.2.war
    java -jar gerrit-2.4.2.war init -d /usr/local/gerrit2


The address may have altered, so check that.

Fill out everything for your needs. The database password is your _secret_. Check that everything works b starting gerrit with

    
    cd ~/bin
    ./gerrit.sh start
    ./gerrit.sh stop


When everything worked fine, you can updated your _init.d_ to start gerrit automatically on startup. You do this by the following commands.

    
    sudo ln -snf /usr/local/gerrit2/bin/gerrit.sh /etc/init.d/gerrit
    sudo update-rc.d gerrit defaults


Now your gerrit sever starts each time your machine starts.


### Troubleshooting


I made some errors during the installation which almost drove me crazy.


#### Authentication via OpenID - Register new Email


It's great that you can access the gerrit server with OpenID. However if you have another email on your OpenID account (like *@gmail) than you have on your ssh-key (like *@your-company.com) than you must register a new Email on your account. That does only work if your smtp-server is correctly configured.

By default gerrit uses "user@hostname" as sender. Well for me it was "gerrit@server" which isn't a valid emailadress. You can configure your user in the [user-section](http://gerrit.googlecode.com/svn/documentation/2.1/config-gerrit.html#user) of gerrit.

    
    [user]
          name = Your name
          email = name@your-company.com
