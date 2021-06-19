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

Search for **cgi.fix_pathinfo** parameter, uncomment and change its value like below: 

**cgi.fix_pathinfo=0**

Save and close file.


**Now edit /etc/php/7.4/fpm/php.ini file:**

sudo nano /etc/php/7.4/fpm/php.ini

**Uncomment and update its value with your timezone:**

[Date]

; Defines the default timezone used by the date functions

; http://php.net/date.timezone

date.timezone = Asia/Karachi


Save and close.

**Edit the /etc/php/7.4/cli/php.ini file:**

sudo nano /etc/php/7.4/cli/php.ini


**Uncomment date.time parameter and update its value with your timezone:**

[Date]
; Defines the default timezone used by the date functions

; http://php.net/date.timezone

date.timezone = Asia/Karachi


Save and close.


**Restart PHP service to take changes into effect:**

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

**Enter current password for root (enter for none):**

OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

**Set root password? [Y/n] y**

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

**Remove anonymous users? [Y/n] y**

 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

**Disallow root login remotely? [Y/n] y**

 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

**Remove test database and access to it? [Y/n] y**

 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

**Reload privilege tables now? [Y/n] y**

 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.


**Thanks for using MariaDB!**

######################################################################

**Installing Web Server**

We will install and use Nginx as our web server:

sudo apt-get -y install nginx-full


**Adding LibreNMS User**

Type the following commands to add a librenms user:

sudo useradd librenms -d /opt/librenms -M -r

sudo usermod -a -G librenms www-data


**Creating Database**


You need to create a database to use with librenms like below:

sudo mysql -u root -p

Type the following at mysql prompt to create a database, user and password:

CREATE DATABASE librenms CHARACTER SET utf8 COLLATE utf8_unicode_ci;

CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'librenms';

GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';

FLUSH PRIVILEGES;

exit


Now edit 50-server.cnf file:

sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

Within the **[mysqld]** section, add below parameters:

innodb_file_per_table=1

sql-mode=""

lower_case_table_names=0

Save and close file when you finished.

Restart MariaDB service to take changes into effect:
sudo systemctl restart mariadb

**Downloading LibreNMS**

Now you need to download librenms on your Ubuntu server like below:
cd /opt
sudo git clone https://github.com/librenms/librenms.git librenms

**Configuring Nginx**

Create a librenms configuration file within nginx to make its web interface accessible:
sudo nano /etc/nginx/sites-available/librenms.conf

Add the below parameters and make sure you replace **your_server_name_or_ip** with yours:

server {
 listen      80;
 server_name **your_server_name_or_ip**;
 root        /opt/librenms/html;
 index       index.php;

 charset utf-8;
 gzip on;
 gzip_types text/css application/javascript text/javascript application/x-javascript image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;
 location / {
  try_files $uri $uri/ /index.php?$query_string;
 }
 location /api/v0 {
  try_files $uri $uri/ /api_v0.php?$query_string;
 }
 location ~ \.php {
  include fastcgi.conf;
  fastcgi_split_path_info ^(.+\.php)(/.+)$;
  fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
 }
 location ~ /\.ht {
  deny all;
 }
}

Save and close file when you are finished.

Now you need to create a symbolic link to librenms.conf file like below:

sudo ln -s /etc/nginx/sites-available/librenms.conf /etc/nginx/sites-enabled/
sudo unlink /etc/nginx/sites-enabled/default

**Restart the service to take changes into effect:**
sudo systemctl restart nginx
sudo systemctl restart php7.4-fpm

**Configuring SNMPD**
Type the following commands to configure snmp to use with librenms:
sudo cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf

sudo nano /etc/snmp/snmpd.conf
Replace the text which says **RANDOMSTRINGGOESHERE** and set your own community string like below:
com2sec readonly  default         **public**

Save and close when you are finished


sudo curl -o /usr/bin/distro https://raw.githubusercontent.com/librenms/librenms-agent/master/snmp/distro
sudo chmod +x /usr/bin/distro
sudo systemctl restart snmpd

**Adding CronJob**
sudo cp /opt/librenms/librenms.nonroot.cron /etc/cron.d/librenms


LibreNMS keeps logs in /opt/librenms/logs. Over time these can become large and be rotated out. You can rotate out the old logs using the below logrotate config file:
sudo cp /opt/librenms/misc/librenms.logrotate /etc/logrotate.d/librenms

**Applying Permissions**
sudo chown -R librenms:librenms /opt/librenms
sudo setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs
sudo setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs

**Run Composer Wrapper**

You will need to run composer wrapper script from /opt/librenms directory like below:
sudo su - librenms
/opt/librenms/scripts/composer_wrapper.php install --no-dev

You will see the output similar to the below while running composer wrapper script and it will take few minutes to complete.
When its done, type the exit command to return back to sudo non-root user terminal:
exit


**Adding Firewall Rules**
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 161/udp
sudo ufw enable

Run the 'ufw status' command to see the firewall status.

sudo ufw status


**LibreNMS Web Installer**

In this step, you will run LibreNMS web installer by navigating to **http://your_server_name** or **http://your_server_ip** in the web browser address bar and press Enter.
You will see the below **install.php page showing the result of pre-install checks**. Make sure all status are installed, yes as shown in the screenshot below.
Click 'Next Stage' to continue.


![1](https://user-images.githubusercontent.com/43593501/122636034-8b2b2d80-d100-11eb-8829-84f1f8fabb1c.png)


Provide database credentials you created earlier and click Next Stage.


![2](https://user-images.githubusercontent.com/43593501/122636044-9a11e000-d100-11eb-8f0e-43a4bc70ae75.png)


This will import librenms database schema and when you see Success click Goto Add User


![3](https://user-images.githubusercontent.com/43593501/122636072-ac8c1980-d100-11eb-8983-6fe1aae66fd2.png)


Add a user, this will be your librenms administrative user:


![4](https://user-images.githubusercontent.com/43593501/122636091-c6c5f780-d100-11eb-8339-0168c4c20e4c.png)


Click Generate Config

![5](https://user-images.githubusercontent.com/43593501/122636104-d7766d80-d100-11eb-93ef-80d9b9765066.png)


**Now stop here and copy this entire script:**

![6](https://user-images.githubusercontent.com/43593501/122636110-e2c99900-d100-11eb-82d9-8e8aa74c45fc.png)



Go back to Ubuntu terminal and create config.php file like below:

sudo nano /opt/librenms/config.php

Paste entire script into it, save and close the file when you are finished.

**Update the permission**

sudo chown -R librenms:librenms /opt/librenms

Now run the validation check

sudo /opt/librenms/validate.php

and you will see the output like below:

![7](https://user-images.githubusercontent.com/43593501/122636180-405de580-d101-11eb-969e-c8b4fe0ae889.png)



If you see any warning other than the adding host you got to fix it first before moving to next step:
Now go back to your browser you left unfinished and click Finish:
As you have already done with validation check so you just need to click on validate your install and fix any issues:


![8](https://user-images.githubusercontent.com/43593501/122636197-55d30f80-d101-11eb-9d40-ec8622258ee5.png)


This will bring you to the below login page of librenms. You can log in with the user and password you created just a moment ago.

![9](https://user-images.githubusercontent.com/43593501/122636214-671c1c00-d101-11eb-8878-a76798673adf.png)


Login and add Devices :D :)


**Wrapping up**

You have successfully completed librenms installation and added localhost as an example of adding device. Now you can start adding your devices like network switches, routers, firewalls, Windows, Linux and Unix servers to monitor their utilization.

