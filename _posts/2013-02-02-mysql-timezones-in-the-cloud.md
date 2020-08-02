---
date: 2013-02-02 15:00:29+00:00
layout: post
title: MySql Timezones in the Cloud
excerpt: On a small university project I found my self developing a web application with the playframework, mysql and some javascript libraries. After testing and developing on my local machine I want to deploy my application. Running this application on Openshift created some unexpected results.
categories:
- cloud
- mysql
- timezones
---

On a small university project I found my self developing a web application with [play](http://www.playframework.org/), [mysql](http://www.mysql.com/) and some javascript libraries. After testing and developing on my local machine I want to deploy my application.

Have heard of [Openshift](https://openshift.redhat.com/)? It's an amazing PaaS product by RedHat. It's currently in _Developer Preview_ and you can test it for free. To deploy it, follow  [this amazing good tutorial](https://github.com/opensas/play2-openshift-quickstart).


## What happend to my 24/7 chart?


[![The correct visualization](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_007-300x116.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_007.png)

The correct visualization

[![The incorrect visualization](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_008-300x113.png)](http://mukis.de/pages/wp-content/uploads/2013/02/Auswahl_008.png)

The incorrect visualization


## Timezones


Openshift uses Amazones EC2 service. In particular the servers are located in the [US-East](https://openshift.redhat.com/community/forums/openshift/server-time-zone) region. But that should be too hard to change, or?


1. Install PhpMyAdmin cartrige
2. Log into your app with ssh and import time zone tables to mysql
  mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u admin -p mysql
3. Login as admin in PhpMyAdmin
4. Set global timezone with
  -- Set correct time zone
  SET GLOBAL time_zone = 'Europe/Berlin';
  -- check if time zone is correctly set
  SELECT version( ) , @@time_zone , @@system_time_zone , NOW( ) , UTC_TIMESTAMP( );


Happy timezone :)


## Links

* [Per Connection](http://www.electrictoolbox.com/mysql-set-timezone-per-connection/)
* [MySql Doc](http://dev.mysql.com/doc/refman/5.5/en//time-zone-support.html)
* [With my.cnf](http://stackoverflow.com/questions/4562456/mysql-setting-time-zone-in-my-cnf-options-file)
* [Project](https://bartrend-mukis.rhcloud.com/trends)
