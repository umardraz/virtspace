==========================================================
  Virtspace Ubuntu 12.04 LTS Server
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

  sudo apt-get install build-essential php5-curl php5-gd php5-mcrypt php-pear -y

Configuring the Apache Virtual Host
-----------------------------------

We will use /var/www/virtspace for our document root of Virtspace, now create the directory and apply proper permission

::

  mkdir -p /var/www/virtspace
  chown -R www-data:www-data /var/www/

We will create a simple virtual host configuration file that will instruct Apache to serve the contents of the directory /var/www/virtspace for any requests to example.yourdomain.com

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

For big volumes clone, migrate we need to update the **max_execution_time** parameter of php.ini so update the default time with the following.

::

  nano /etc/php5/apache2/php.ini
  max_execution_time = 1200

Next we need to restart the apache service.

sudo service apache2 restart

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

  sudo service apache2 restart
  
Brwose the url e.g. http://virtspace.yourdomain.com/, and enjoy :)
