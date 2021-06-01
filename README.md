NConf README
============

**NConf is not dead.** It's just no longer developed by the original developer(s). Here is the most current NConf build, maintained by Robbie Ferguson for NEMS Linux.

NConf homepage:
http://www.nconf.org/

NEMS homepage:
https://nemslinux.com/

**THIS VERSION HAS BEEN RE-WORKED BY ROBBIE FERGUSON TO OPERATE UNDER PHP 7.0+ / mySQLi**




Start guide:
http://www.nconf.org/dokuwiki/doku.php?id=nconf:help:documentation:start:main

Complete documentation:
http://www.nconf.org/dokuwiki/doku.php?id=nconf:help:documentation:detail:main

Table of contents
-----------------
A. Requirements
B. Manual installation
C. Manual update


A. Requirements
---------------

NConf system requirements:

    * Apache webserver
    * PHP 7 or higher, php-mysql, php-ldap (only if using LDAP auth)
    * MySQL 5.0.2 or higher (with InnoDB)
    * Perl 5.6 or higher, perl-DBI, perl-DBD-MySQL
    * Nagios 3.x or Icinga 0.8x (binary necessary for testing generated config)


php.ini settings:

    * short_open_tag = On
    * register_globals = Off
    * magic_quotes_gpc = Off 


Compatibility:
NConf has been tested on Red Hat, Fedora and CentOS Linux.
It has not been tested on Windows or any UNIX, or with SELinux.


Security considerations:
NConf is intended to be deployed within a private network or intranet. As such, development focus is clearly not that of a secure Internet application. Although it can be run anywhere, we do not advise users to publish NConf as a public site. 


B. Manual installation
----------------------

1. Download and unpack
Download and unpack the NConf archive to your webserver's document root folder. 


2. Set permissions
Make sure the following directories are writable for your webserver user: 

   ./config
   ./output
   ./static_cfg
   ./temp


3. Create the database

---
:information_source: Confirm that MySQL is installed and running (via apt and systemctl on Ubuntu 20.04)

```bash
$ sudo apt install mysql-server
...
$ sudo systemctl status mysql
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: inactive (dead)
$ sudo systemctl start mysql
$ sudo systemctl status mysql
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2021-06-01 09:20:17 AEST; 1s ago
    Process: 944391 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
   Main PID: 944421 (mysqld)
     Status: "Server is operational"
      Tasks: 38 (limit: 28775)
     Memory: 332.6M
     CGroup: /system.slice/mysql.service
             └─944421 /usr/sbin/mysqld
```
---

Create a new MySQL database for NConf, create a user to access the database, grant the appropriate privileges (make sure InnoDB for MySQL is set up properly prior to creating the database). 

On the commandline, you would proceed like this: 

   $> mysql -u root -p
   Enter password:
   mysql> CREATE DATABASE DBNAME;
   mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER ON `DBNAME`.* TO 'DB_USER'@'localhost' IDENTIFIED BY 'DB_PASS';

Please refer to the MySQL manual on how to set up InnoDB (the steps might vary depending on your OS distribution). 

---
:information_source: MySQL 8.0 for Ubuntu 20.04 includes
[InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-check-availability.html) as the default engine.

```bash
$ echo 'SHOW ENGINES;' | mysql DBNAME -u root -p | grep InnoDB
Enter password:
InnoDB  DEFAULT Supports transactions, row-level locking, and foreign keys  YES YES YES
```
---

4. Create the database structure

$> mysql -u DBUSER -p DBNAME < INSTALL/create_database.sql


5. Configure NConf
Copy the contents of ./config.orig to ./config. Make sure you also copy the hidden files. 

Edit ./config/mysql.php, set at least the following values: 

   DBHOST 
   DBNAME 
   DBUSER 
   DBPASS

Edit ./config/nconf.php, set at least the following values: 

   NCONFDIR 
   NAGIOS_BIN


6. Clean up
Remove the INSTALL and UPDATE folders, as well as the INSTALL.php and UPDATE.php scripts. 


7. Get OS logo icons (optional, but nice)
If you like, download the OS logo icons and unpack them to ./img/logos/base/ 

The icons can be downloaded here:
http://www.monitoringexchange.org/p/47

8. Authentication
If you need authentication, configure ./config/authentication.php appropriately.


C. Manual update
----------------

1. Backup
Make a backup copy of your whole 'nconf/' folder and create a dump of your 'nconf' database. 
The update process will not overwrite your configuration, but if you have made changes to any other files, these changes will be lost! This is also true for icons that you might have installed manually ('img/logos/base/'). 

    Example: 

    cp -rp nconf/ nconf.BAK 
    mysqldump -u DBUSER -p DBNAME > nconf-dbdump.sql


2. Download and unpack
Download and unpack the NConf archive to a temporary location on your webserver. 

    Example: 

    tar -zxf nconf-VERSION.tgz -C /tmp/


3. Update existing files
Copy the unpacked files to the 'nconf/' folder and overwrite any existing files of your current NConf installation. 

    Example: 

    /bin/cp -rv /tmp/nconf/* /path/to/nconf/  (specify the full path to the 'cp' binary!)

This step cannot be undone! Make sure you have made a backup of your 'nconf/' folder. 


4. Check permissions
Make sure the following directories are still writable for your webserver user:

   ./config 
   ./output 
   ./static_cfg
   ./temp


5. Update the database
The 'UPDATE/' folder contains a subfolder for each previous release of NConf. The folders are named like this: x.x.x_to_y.y.y 

Make sure you know your current version of NConf. Next, chose the appropriate folder and look for a script named update_database_x.x.x_to_y.y.y.sql. 

Update the database like this: 

    $> mysql -u DBUSER -p DBNAME < UPDATE/x.x.x_to_y.y.y/update_database_x.x.x_to_y.y.y.sql

If you are updating from an older version of NConf, make sure you apply all available updates for all versions between your version and the latest version of NConf. Also, make sure you apply the updates in the right order! 

Important: make sure you don't skip any updates, and that you apply ALL of them in the right order. Failure to do so may result in data corruption and instability of NConf! 


6. Additional tasks
Check the version-specific 'README' file within the current subfolder for update-related instructions. If you have applied more than one update, make sure you study all README files in all subfolders. These files contain important instructions about necessary config changes etc. 

The release notes for the current release can also be found here:
http://www.nconf.org/dokuwiki/doku.php?id=nconf:download:releasenotes


7. Restore icons (optional)
If you were using additional icons prior to the update, you will have to copy these back from your backup folder. 


8. Clean up
Finally, remove the following files and directories:

   * INSTALL
   * INSTALL.php
   * UPDATE
   * UPDATE.php 

The update should now be complete.

Refresh browser cache:

If NConf does not look or feel right after the update, push CTRL-F5.
Your browser might have to reload some cached CSS & JavaScript files.  
