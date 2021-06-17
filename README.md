# LibreNMS

**Set Timezone**

sudo timedatectl set-timezone Asia/Karachi

**Installing PHP**

sudo apt install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install acl curl composer fping git graphviz imagemagick mariadb-client mariadb-server mtr-tiny nginx-full nmap php7.4-cli php7.4-curl php7.4-fpm php7.4-gd php7.4-json php7.4-mbstring php7.4-mysql php7.4-snmp php7.4-xml php7.4-zip rrdtool snmp snmpd whois unzip python3-pymysql python3-dotenv python3-redis python3-setuptools

**Configuring PHP**

You also need to make few changes in PHP configuration file like below:
sudo nano /etc/php/7.4/cli/php.ini

Search for cgi.fix_pathinfo parameter, uncomment and change its value like below:
cgi.fix_pathinfo=0

Save and close file.

Now edit /etc/php/7.4/fpm/php.ini file:
sudo nano /etc/php/7.4/fpm/php.ini

Uncomment and update its value with your timezone:
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = Asia/Karachi


Save and close.

Edit the /etc/php/7.4/cli/php.ini file:

sudo nano /etc/php/7.4/cli/php.ini
Uncomment date.time parameter and update its value with your timezone:
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = Asia/Karachi


Save and close.


Restart PHP service to take changes into effect:
sudo systemctl restart php7.4-fpm

**Installing Database**

sudo apt-get -y install mariadb-client mariadb-server


**Securing Database**

sudo mysql_secure_installation




NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
