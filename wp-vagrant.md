#Vagrant Server Set-up
This serves as a guide to set-up a LAMP server and configure it for Wordpress. This is all done by hand. 
While you might run into road blocks this is at least a starting point.  If you need more resources I would suggest looking at this blog post (http://www.hongkiat.com/blog/install-wordpress-locally-vagrant/)

**Table of Contents**

- **[1.0 Initialize](#initialize)**
- **[2.0 Edit Vagrantfile](#edit-vagrantfile)**
- **[3.0 Provision Script](#provision-script)**
- **[4.0 Manual Vagrant](#manual-vagrant)**
- **[4.a First Launch](#first-launch)**
- **[4.b Install Extras](#install-extras)**
- **[4.c Install Apache](#install-apache)**
- **[4.d Install PHP](#install-php)**
- **[4.e Install mod_rewrite](#install-mod_rewrite)**
- **[4.f Install MySql](#install-mysql)**
- **[4.g Install phpMyAdmin](#install-phpmyadmin)**
- **[4.h Install Wordpress](#install-wordpress)**
- **[4.i Fix Permissions](#fixing-permissions)**
- **[5.0 Install Git](wp-git.md)**
- **[6.0 LAMP Tips](wp-stacktips.md)**

##Initialize
vagrant init [box-name] [box-url]

`vagrant init ubuntu/trusty64`

**Check to make sure Vagrantfile is created**

##Edit Vagrantfile
Configure Port Forwarding for local browers viewing
Add a provision if wanted on start-up to help load initialize the server

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.network "private_network", ip: "192.168.33.10"
    config.vm.provision "shell", path: "./scripts/wp-provision.sh"
end
```
##Provision Script
So if you don't want to do everything by hand, you should use a provision script.
This makes it a lot easier but you should learn how to do everything by hand.  This is why I added how to do it the old school way below :-)

If you are interested in the [provision script then click here](wp-provision.md)

#Manual Vagrant
Everything below is how to do things by hand, so you know what the heck is going on in the provision script!

##First Launch
**Start up the server**

`vagrant up`

**Connect to the server via ssh**

`vagrant ssh`

**Update your server to the latest**

`sudo apt-get update`
`sudo apt-get upgrade -y`

##Install Extras

```
sudo apt-get install -y software-properties-common
sudo apt-get install -y python-software-properties
sudo apt-get install -y zip unzip
sudo apt-get install -y curl
sudo apt-get install -y build-essential
sudo apt-get install -y vim
```

##Install Apache

`sudo apt-get install -y apache2`

**Remove default html location and create softlink

```
sudo rm -rf /var/www/html
sudo ln -fs /vagrant /var/www/html
```

##Install PHP

```
sudo apt-get install -y php5 libapache2-mod-php5 php5-mcrypt php5-curl php5-mysql php5-xdebug php5-gd
````

Or PHP 7
```
sudo apt-add-repository ppa:ondrej/php-7.0
sudo apt-get update
sudo apt-get install php7.0
sudo apt-get install php7.0-cli php7.0-common libapache2-mod-php7.0 php7.0 php7.0-mysql php7.0-fpm php7.0-curl php7.0-json php7.0-cgi php7.0-mcrypt

```
##Install mod_rewrite

```
sudo a2enmod rewrite
sudo service apache2 restart
```

###Configure mod_rewrite

Go to the following location `/etc/apache2/sites-available`

Edit the default conf file: `sudo nano 000-default.conf`

Add the following allow .htacess overrides
```
<Directory /var/www/html/>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    allow from all
</Directory>
```
Restart Apache
`sudo service apache2 restart`

Goto site directory and add .htaccess file.  In the .htaccess file add the following:
`RewriteEngine on`

##Install MySql

```
sudo apt-get install -y mysql-server mysql-client
```

##Install phpMyAdmin
```
sudo apt-get install phpmyadmin
````

Then make sure you select the correct web server, apache2. Configure database, provide admin password. The default user\root for mysql is root - secret

Now vi into /etc/apache/apache2.conf on the bottom line add 

```
Include /etc/phpmyadmin/apache.conf
```

Restart apache

To visit your phpmyadmin `http://ipaddress/phpmyadmin`

##Install Wordpress

Download and Upzip
```
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xfz latest.tar.gz
```

Move wordpress and delete shit
```
mv wordpress/* ./
rmdir ./wordpress/
rm -f latest.tar.gz
```

Make a mysql database
```
mysql -u username -p

create database dbname;
grant usage on *.* to username@localhost identified by 'password';
grant all privileges on dbname.* to username@localhost;
```

Test Database
```
use dbname;
```
It should say that the database changed, if it does you can exit by typing `exit`

**Run your Stuff**

Go through the set-up.

##Fixing Permissions
```
sudo vi /etc/apache2/apache2.conf
```

Type `/User` to find the nearest user section. In there you can define your user and the group.

Press "i" to go into Edit mode. **Edit the user** and the group to "vagrant", like so:
```
User vagrant
Group vagrant
```

Once done press esscape and type `:wp`.

Now `vagrant halt` and `vagrant up` to restart.

##[Install Git](wp-git.md)
##[LAMP Tips](wp-stacktips.md)
