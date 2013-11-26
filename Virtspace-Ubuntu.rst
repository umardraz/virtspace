==========================================================
  Virtspace Ubuntu 12.04 LTS Server
==========================================================

:Version: 2.0
:Source: https://app.box.com/s/oyve7lnn63vngf6xxhvm
:Keywords: Linux KVM administration, PHP-libvirt, MongoDB, PHP, libvirt, Ubuntu

Author
==========

Copyright (C) Umar Draz <umar_draz@yahoo.com>

Table of Contents
=================

::

  1. Requirements
  2. MongoDB Installation
  3. Webserver Installation
  4. php-libvirt installation
  5. libvirt configuration

1. Requirements
===============

* Virtspace based on php-libvirt for managing kvm virtulization.
* MongoDB database server required, we will see later on how to installl.
* PHP v4 and above with pecl mongo supported.

2. MongoDB Server Installation
============================

Installing MongoDB Server on Ubuntu is a quick and easy process. In classic fashion let’s get the process underway by updating our system.

::

  sudo apt-get update
  sudo apt-get upgrade

Accept any updates that are available to you and then install MongoDB Server like so:
  
::

  sudo apt-get install mongodb

When complete, run the following command to check mongodb working or not

::

  sudo mongo

That's All MongoDB server has been installed, now lest install Apache and PHP.

3. WebServer Installation
=========================

Apache is easily installed by entering the following command.

::

  sudo apt-get install apache2 -y

During the install you may notice the following warning:

::

  apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName

This comes from Apache itself and means that it was unable to determine its own name. The Apache server needs to know its own name under certain situations. For example, when creating redirection URLs.

To stop this warning we can create an Apache config file to store the name. You can set this as either a hostname or a FQDN, but here we will use this as only "localhost"

::

  echo "ServerName localhost" > /etc/apache2/conf.d/servername.conf
  
In order for this change to take effect restart Apache. The warning should no longer appear.

::

  sudo service apache2 restart

Virtspace depends on url rewriting for SEO purpose. In order to take advantage of this feature we need to enable Apache's rewrite module with the a2enmod command.

::

  sudo a2enmod rewrite
  sudo service apache2 restart

Installing PHP
-----------------

We will therefore install PHP with the following command.

::

  sudo apt-get install build-essential php5-curl php5-gd php5-mcrypt php5-mysql php-pear -y

Configuring the Apache Virtual Host
-----------------------------------

We will use /var/www/virtspace for our document root of Postfix vManager, now create the directory and apply proper permission

::

  mkdir -p /var/www/virtspace
  chown -R www-data:www-data /var/www/

We will create a simple virtual host configuration file that will instruct Apache to serve the contents of the directory /var/www/vmanager for any requests to example.yourdomain.com

::

  sudo bash -c "cat >> /etc/apache2/sites-available/virtspace.yourdomain.com <<EOF
  <VirtualHost *:80>
    ServerName virtspace.yourdomain.com
    ServerAlias yourdomain.com
    DocumentRoot /var/www/virtspace
    ErrorLog /var/log/httpd/virtspace.error.log
    CustomLog /var/log/httpd/virtspace.access.log combined
  </VirtualHost>
  EOF"

As you notice, I have use /var/log/httpd directory for our application logs. We need to create this directory, before enabling our virtualhost.

::

  mkdir /var/log/httpd

Using the a2ensite command and restarting Apache will load the new configuration file. But before this we will remove the existing link from site-enabled directory.

::

  rm /etc/apache2/sites-enabled/000-default
  sudo a2ensite virtspace.yourdomain.com
  sudo service apache2 restart

Next we need to install the mongo library for php using pecl.

::
  
  pecl install mongo
  
After installing mongo extension we need to enable this into php.

::

  echo 'extension=mongo.so' > /etc/php5/conf.d/mongo.ini
  sudo service apache2 restart

If everything has gone according to plan you should be able to open a browser and navigate to virtspace.yourdomain.com where you will see a directory listing.

4. PHP-libvirt Installation
=========================

For php-libivrt first we need to install some dependencies packages.

::

  sudo apt-get install git libvirt-dev xsltproc libxml2-dev libxml2 libxml2-utils lvm2 python-libvirt python-numpy

After installting the dependencies packages, we need to download the php-libvirt from the following link and then compile it.

::

  wget http://libvirt.org/sources/php/libvirt-php-0.4.8.tar.gz
  tar xzvpf libvirt-php-0.4.8.tar.gz

  cd libvirt-php-0.4.8
  ./configure --disable-option-checking --enable-feature=yes
  make
  make install


Now let's start the installation of Postfix vManager

5. Virtspace
============

First download postfix virtspace source from this url :Source: https://app.box.com/s/oyve7lnn63vngf6xxhvm

After downloading the postfix-vmanager-2.0.tar.gz just extract the source. 

Then first remove the /var/www/vmanager directory and move extracted source into /var/www/vmanager/ let's do it.

::

  tar xzvpf postfix-vmanager-2.0.tar.gz
  rm -rf /var/www/vmanager
  mv postfix-vmanager-2.0 /var/www/vmanager
  
Next restore the database, with the following command

::

  cd /var/www/vmanager/  
  mysql -uroot -proot_pass vmanager < setup/vmanager.sql

5.1. Configure Postfix vManager
----------------------

Edit the inc/config.inc.php file and add your settings there. The most important settings are those for your database server.

::

  $CONF['database_host'] = 'localhost';
  $CONF['database_user'] = 'vadmin';
  $CONF['database_password'] = 'vadmin_password';
  $CONF['database_name'] = 'vmanager';
  $CONF['database_port'] = '3306';
  $CONF['database_prefix'] = '';

Postfix vManager require write access to its directory. So you need to change the vmanager directory ownership with that user as web server running.

::

  chown -R www-data:www-data /var/www/vmanager/

5.2. Check settings, and create Admin user
------------------------------------------

Hit :Source: https://example.yourdomain.com/ in a web browser. You should see a list of 'OK' messages. Otherwise reslove the issue if found. 

Create the admin user using the form displayed. This is all that is needed.

5.3. Vacations
--------------

The vacation script runs as service within Postfix's master.cf configuration file. Mail is sent to the vacation service via a transport table mapping. When users mark themselves as away on vacation, an alias is added to their account sending a copy of all mail to them to the vacation service.

To use vacation services you need to first create vacation domain. Just login as Super Admin account and then 

5.4. Installing Vacations
-------------------------

Login as Super Admin and then create Vacation domain following this.

::

  Go to Settings -> Vacation Domain.

There are a bunch of Perl modules which we need to install for Vacation setup.

::

  apt-get install libmime-encwords-perl libemail-valid-perl libemail-sender-perl libmail-sender-perl liblog-log4perl-perl liblog-dispatch-perl libdbi-perl libdbd-mysql-perl libmime-charset-perl

**Create Vacation Account:**

Create a dedicated local user account called "vacation". This user handles all potentially dangerous mail content - that is why it should be a separate account.

Do not use "nobody", and most certainly do not use "root" or "postfix". The user will never log in, and can be given a "*" password and non-existent shell and home directory.

Create the user with the following command.

::

  useradd vacation -c "Vacation Owner" -d /nonnonexistent -s /bin/false

**Create a directory:**

Create a directory, for example  /var/spool/vacation, that is accessible only to the "vacation" user. This is where the vacation script is supposed to store its temporary files. 

::

  mkdir /var/spool/vacation
  
**Copy Files:**

Copy the vacation.pl file to the directory you created above:

::

  cp setup/vacation.pl /var/spool/vacation/vacation.pl
  chown -R vacation:vacation /var/spool/vacation/
  
Which will then look something like:

::

  -rwx------   1 vacation  vacation  3356 Dec 21 00:00 vacation.pl*

**Setup the transport type:**

Define the transport type in the Postfix /etc/postfix/master.cf file:

::

  vacation    unix  -       n       n       -       -       pipe
    flags=Rq user=vacation argv=/var/spool/vacation/vacation.pl -f ${sender} -- ${recipient}
    
Here we need to restart postfix service.

::

  service postfix restart

**Configure vacation.pl"**

The perl /var/spool/vacation/vacation.pl script needs to know which database you are using, and also how to connect to the database.

Change any variables starting with '$db_' and '$db_type'

Change the $vacation_domain variable to match what you entered through your Super Admin login.

Here is the example of vacatino.pl settings for database and domain name

::

  our $db_type = 'mysql';
  our $db_host = 'localhost';
  our $db_username = 'username';
  our $db_password = 'password';
  our $db_name     = 'dbname';
  our $vacation_domain = 'autoreply.yourdomain.com';

Done! When this is all in place you need to have a look at the Postfix vManager inc/config.inc.php. Here you need to enable Virtual Vacation for the site.

6. Domain Keys
==============

I’m going to show you how to run Postifx with OpenDKIM on a Ubuntu LTS Server.

Let’s start installing OpenDKIM.

::

  apt-get install opendkim opendkim-tools

Edit Postfix configuration file.

::

  nano /etc/postfix/main.cf

And instruct postfix to use dkim milter:

::

  smtpd_milters = inet:127.0.0.1:8891
  non_smtpd_milters = $smtpd_milters
  milter_default_action = accept

Create configuration file for OpenDKIM

::

  nano /etc/opendkim.conf
  
Feel free to use the following one slightly edited to work with **yourdomain.com** domain:

::

  LogWhy            yes
  Syslog            yes
  SyslogSuccess     yes
  Canonicalization  relaxed/simple
  KeyTable          /etc/opendkim/KeyTable
  SigningTable      /etc/opendkim/SigningTable
  InternalHosts     /etc/opendkim/TrustedHosts
  Socket            inet:8891@localhost
  ReportAddress     root
  SendReports       yes

Edit /etc/opendkim/TrustedHosts

::

  nano /etc/opendkim/TrustedHosts

Add domains, hostnames and/or ip’s that should be handled by OpenDKIM. Don’t forget localhost.

::

  127.0.0.1
  localhost
  x.253.204.64
  x.253.204.32/27

**Generate keys**

Now generate the keys: one will be used by opendkim to sign your messages and the other to be inserted in your dns zone:

::

  mkdir -p /etc/opendkim/yourdomain.com
  opendkim-genkey -D /etc/opendkim/yourdomain.com/ -d yourdomain.com -s default
  
Here you need to move **default.private** to **default**

::

  cd /etc/opendkim/yourdomain.com/
  mv default.private default
  chown opendkim:opendkim /etc/opendkim/
  
Add domain to KeyTable /etc/opendkim/KeyTable

::

  default._domainkey.yourdomain.com yourdomain.com:default:/etc/opendkim/yourdomain.com/default

Add domain to SigningTable /etc/opendkim/SigningTable

::

  yourdomain.com default._domainkey.yourdomain.com

**Add to DKIM public key to DNS**

Add an entry for the public key to the DNS server you are using for your domain. You find the public key here:

::

  cat /etc/opendkim/yourdomain.com/default.txt
  
The above output should be like.

::

  default._domainkey IN TXT "v=DKIM1; g=*; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQClJj0qvcQvX7ssbGNBqFCTt+Wrh9G15QIXkFPbspt4uUOthLR8yl56CKohRVFfQTjoZjrmxSYDD8ZfV4rnPUu5bz07w7hbL3X1N5rLOM7RTDWU0NrYzGNVS07H4XNUJQRifVULREEqqvjASX6ivp1AH+OvvKn9mQTaSTjviD2cdQIDAQAB"

Now test the key using an OpenDKIM utiliy:

::

  opendkim-testkey -vvv -d yourdomain.com -s default -k /etc/opendkim/yourdomain.com/default
  
The above command will verify your zone settings.

Now start both OpenDKIM and Postifix:

::

  /etc/init.d/opendkim start
  /etc/init.d/postfix restart

Look at the DKIM-signature, there it is.

Further check and analysis can be made also on the website http://www.brandonchecketts.com/emailtest.php
