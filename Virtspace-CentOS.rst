==========================================================
  Virtspace Ubuntu CentOS 6.4
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

  /etc/init.d/httpd restart
  
If you want to run Apache by default when the system boots, which is a typical setup, execute the following command:

::

  /sbin/chkconfig --levels 235 httpd on
  
Installing PHP
-----------------

We will therefore install PHP with the following command.

::

  yum install php php-mysql php-pdo php-mysqli php-mbstring php-pear

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

Whenver you change anything in php.ini file then you need to rstart apache server.

::

  /etc/init.d/httpd restart
  
If everything has gone according to plan you should be able to open a browser and navigate to virtspace.yourdomain.com where you will see a directory listing.

4. PHP-libvirt Installation
===========================

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

After installing mongo extension we need to enable this into php.

::

  echo 'extension=libvirt-php.so' > /etc/php5/conf.d/libvirt.ini
  sudo service apache2 restart

Web server installation is now completed, next we need to configure all KVM hosts, so SSH to all of your KVM host and do the following only on KVM hosts machines.

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

Edit **libvirtd_opts** variable in the /etc/init/libvirt-bin.conf file:

::

  env libvirtd_opts="-d -l"
  
Edit the same field in **/etc/default/libvirt-bin** and again, set it to:

::

  libvirtd_opts="-d -l"
  
Restart the libvirt service to apply the changes:

::

  service libvirt-bin restart

6. Bridge Networking
====================

For virtspace working corectly you need to configure bridge networking on each **KVM Host**. The bridge network should start with **br-** string. Following is the example of my KVM hosts bridge configuration.

::

  # The primary network interface
  auto eth0
  iface eth0 inet manual

  auto br-net
  iface br-net inet static
    address 192.168.1.20
    netmask 255.255.255.0
    gateway 192.168.2.1
    dns-nameservers 208.67.220.220
    bridge_ports eth0
    bridge_fd 9
    bridge_hello 2
    bridge_maxage 12
    bridge_stp off
    
After changing in the network configuration file, you need to restart the network services.

::

  service networking restart


7. Virtspace
============

First download virtspace source from this url https://app.box.com/s/oyve7lnn63vngf6xxhvm

After downloading the virtspace.tar.gz just extract the source. 

Then first remove the /var/www/virtspace directory and move extracted source into /var/www/virtspace/ let's do it.

::

  cd /tmp/
  tar xzvpf virtspace.tar.gz
  rm -rf /var/www/virtspace
  mv virtspace /var/www/virtspace

  cd /var/www/virtspace/noVNC/
  ln -s images/favicon.ico .
  
  cd /var/www/virtspace/noVNC/
  ln -s websockify websockify.py
  ln -s websockify wsproxy.py
  
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

  sudo service apache2 restart
  
Brwose the url e.g. http://virtspace.yourdomain.com/, and enjoy :)
