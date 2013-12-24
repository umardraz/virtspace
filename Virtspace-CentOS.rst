==========================================================
  Virtspace Ubuntu CentOS 6.4
==========================================================

:Version: 2.0
:Source: https://app.box.com/s/3uejhccsalswcvs56s6a
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
  5. KVM Hosts configuration
  6. Bridge Networking
  7. Virtspace Installation

1. Requirements
===============

* Virtspace based on php-libvirt for managing kvm virtulization.
* MongoDB database server required, we will see later on how to installl.
* PHP v4 and above with pecl mongo supported.

2. MongoDB Server Installation
============================

Create a **/etc/yum.repos.d/mongodb.repo** file to hold the following configuration information for the MongoDB repository.

::

  [mongodb]
  name=MongoDB Repository
  baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
  gpgcheck=0
  enabled=1

Issue the following command (as root or with sudo) to install the latest stable version of MongoDB and the associated tools:
  
::

  yum install mongo-10gen mongo-10gen-server

When this command completes, you have successfully installed MongoDB!

Start the mongod process by issuing the following command (as root or with sudo):

::

  setsebool httpd_can_network_connect 1 # setsebool httpd_can_network_connect_db 1 
  service mongod start

3. WebServer Installation
=========================

Apache is easily installed by entering the following command.

::

  yum install httpd

**Configure Name-based Virtual Hosts**

There are different ways to set up Virtual Hosts, however we recommend the method below. By default, Apache listens on all IP addresses available to it.

Now we will create virtual host entries for virtspace.yourdomain.com site that we need to host with this server. Here is this.

**File:** /etc/httpd/conf.d/vhost.conf

::

  NameVirtualHost *:80
  <VirtualHost *:80>
     ServerAdmin webmaster@yourdomain.com
     ServerName yourdomain.com
     ServerAlias virtspace.yourdomain.com
     DocumentRoot /var/www/virtspace
     ErrorLog /var/log/httpd/error.log
     CustomLog /var/log/httpd/access.log combined
  </VirtualHost>

Before you can use the above configuration you'll need to create the specified directories. For the above configuration, you can do this with the following commands:

::

  mkdir -p /var/www/virtspace

Virtspace depends on url rewriting for SEO purpose. In order to take advantage of this feature we need to edit httpd.conf file as follows.

Edit /etc/httpd/conf/httpd.conf file and change **AllowOverride None** to **AllowOverride All** under / directory e.g.

::

  <Directory />
    Options FollowSymLinks
    AllowOverride All
  </Directory>

After you've set up your virtual hosts, issue the following command to run Apache for the first time:

::

  service httpd restart

Installing PHP
-----------------

We will therefore install PHP with the following command.

::

  yum install gcc php php-devel php-mbstring php-pear php-xml

Once PHP5 is installed we'll need to tune the configuration file located in /etc/php.ini to enable more descriptive errors, logging, and better performance. These modifications provide a good starting point if you're unfamiliar with PHP configuration.

Make sure that the following values are set, and relevant lines are uncommented (comments are lines beginning with a semi-colon (;)):

**File:** /etc/php.ini

::

  error_reporting = E_COMPILE_ERROR|E_RECOVERABLE_ERROR|E_ERROR|E_CORE_ERROR
  display_errors = Off
  log_errors = On
  error_log = /var/log/php.log
  max_execution_time = 300
  memory_limit = 64M
  register_globals = Off
  max_execution_time = 1200

Nex we need to install MongoDB driver for php

::
  
  pecl install mongo
  echo "extension=mongo.so" >> /etc/php.ini

Whenver you change anything in php.ini file then you need to rstart apache server.

::

  service httpd restart
  
If everything has gone according to plan you should be able to open a browser and navigate to virtspace.yourdomain.com where you will see a directory listing.

4. PHP-libvirt Installation
===========================

For php-libivrt first we need to install some dependencies packages.

::

  yum install git libvirt-devel libxml2-devel libxml2 libxml2-devel lvm2 libvirt-python numpy python-paramiko

After installting the dependencies packages, we need to download the php-libvirt from the following link and then compile it.

::

  git clone git://libvirt.org/libvirt-php.git
  ./autogen.sh
  make
  make install

After installing libvirt library we need to restart httpd service.

::

  service httpd restart

Web server installation is now completed, next we need to configure all KVM hosts, so SSH to all of your KVM host and do the following only on KVM hosts machines.

Disable SELinux
===============

edit **/etc/selinux/config** and change the SELINUX line to SELINUX=disabled: 

::

  SELINUX=disabled
  
... and then reboot the system. 


5. KVM Hosts configuration
===========================

First delete the **default virtual bridge**

::

  virsh net-destroy default
  virsh net-undefine default

For live migration uncomment these lines in libvirt settings.

::
  
  nano /etc/libvirt/libvirtd.conf 

  listen_tls = 0
  listen_tcp = 1
  auth_tcp = "none"

Uncomment the line in the file /etc/sysconfig/libvirtd:

::

  LIBVIRTD_ARGS="--listen"
  
Restart the libvirt service to apply the changes:

::

  service libvirtd restart

6. Bridge Networking
====================

For virtspace working corectly you need to configure bridge networking on each **KVM Host**. The bridge network should start with **br-** string. First we need to install bridge utils.

::

  yum install bridge-utils 

Following is the example of my KVM hosts bridge configuration.

::

  nano /etc/sysconfig/network-scripts/ifcfg-eth0 
  
  DEVICE=eth0
  TYPE=Ethernet
  ONBOOT=yes
  NM_CONTROLLED=yes
  BRIDGE=br-net
  
  nano /etc/sysconfig/network-scripts/ifcfg-br-net
  
  DEVICE=br-net
  TYPE=Bridge
  BOOTPROTO=static
  DNS1=192.168.0.1
  GATEWAY=192.168.0.1
  IPADDR=192.168.0.100
  NETMASK=255.255.255.0
  ONBOOT=yes
  SEARCH="example.com"

After changing in the network configuration file, you need to restart the network services.

::

  service network restart


7. Virtspace
============

First download virtspace source from this url https://app.box.com/s/3uejhccsalswcvs56s6a

After downloading the virtspace.tar.gz just extract the source. 

Then first remove the /var/www/virtspace directory and move extracted source into /var/www/virtspace/ let's do it.

::

  cd /tmp/
  tar xzvpf virtspace.tar.gz
  rm -rf /var/www/virtspace
  mv virtspace /var/www/virtspace

Next restore the database, with the following command

::

  cd /var/www/virtspace/setup/  
  mongorestore virtspace

5.1. Configure Virtspace
------------------------

Edit the inc/config.inc.php and change the **admin** user password as well as the bridge configuration according to your kvm hosts networking settings.

::

  $CONF['username'] = 'admin';
  $CONF['password'] = 'vspace';
  $CONF['bridges'] = array ('br-net','br-int');

After that just restart the apache service and access the virtspace.

::

  setsebool httpd_can_network_connect 1
  service httpd restart
  
Brwose the url e.g. http://virtspace.yourdomain.com/, and enjoy :)
