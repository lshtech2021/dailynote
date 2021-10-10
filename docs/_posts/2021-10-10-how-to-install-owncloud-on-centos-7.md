## Introduction

**OwnCloud** is a popular and open source file hosting service. You can deploy OwnCloud in your own server and secure your files by multi-factor authentication, encryption, and file lifecycle management. You can also share files with others and control users' access right of the files and folders.

In this tutorial, we will setup a personal file hosting service with ownCloud 10.8 on a CentOS 7 server.

## Prerequisites

Create a server instance with CentOS 7 and login as `root`.

## 1. Install LAMP (Linux, Apache, MySQL, and PHP) Stack

ownCloud requires a PHP-enabled webserver and a MySQL database, we need to install them at first.

### Install MySQL

Update and install EPEL repository with following command:

    $ yum update
    $ yum install epel-release yum-utils -y

Download and prepare MySQL repository:

    $ wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm --no-check-certificate
    $ rpm -Uvh mysql80-community-release-el7-3.noarch.rpm

Note, if you don't use `--no-check-certificate` option, this command may prompt following error:
    ERROR: cannot verify download.opensuse.org's certificate, issued by ‘/C=US/O=Let's Encrypt/CN=R3’:
      Issued certificate has expired.
    To connect to download.opensuse.org insecurely, use `--no-check-certificate'.

It's ready to install MySQL now:

    $ yum install mysql-server

Start MySQL server with following command:

    $ systemctl start mysqld

Check MySQL server status:

    $ systemctl status mysqld`

If you see the following message, means MySQL is running:

    Active: **active (running)** since Sat 2021-10-09 10:09:08 PDT; 7h ago

MySQL generates a temporary root password after installed, we need to change it. Find the temporary root password with following command:

    $ grep 'password' /var/log/mysqld.log

The bold text after `root@localhost:` is the temporary password:

    2021-10-09T03:13:33.139046Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: **pz7iaSrsCK>9**

Run following command:

    $ mysql_secure_installation

Input the temporary password (in this tutorial is **pz7iaSrsCK>9**), and set a new strong password as prompts:

    The existing password for the user account root has expired. Please set a new password.
    New password:
    Re-enter new password:

    Tips:  
    A strong password is 8-12 characters long and includes numbers, special characters, uppercase, and lowercase letters.

Connect to MySQL with `mysql` client:

    $ mysql -u root -p

Input your new MySQL root user password, if the installation works, you will login the console:

    mysql>

### Install Apache Web Server (httpd)

Install httpd with following command:

   $ yum -y install httpd mod_ssl

Start the service:

   $ systemctl start httpd.service

Check service status:

   $ systemctl status httpd.service

When the following message, means it is running:

    Active: **active (running)** since Sat 2021-10-09 10:09:08 PDT; 7h ago

### Install PHP 7.3

Install REMI repository and enable PHP 7.3:

    $ yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
    $ yum-config-manager --enable remi-php73

Install PHP modules and other related software packages:

    $ yum install openssl php php-common php-opcache php-mcrypt php-cli php-gd php-curl php-mysqlnd php-imagick php-imap php-intl php-json php-mbstring php-mysql php-ssh2 php-xml php-zip php-apcu php-redis redis-server wget

Restart httpd service:

    $ systemctl restart httpd.service

## 2. Install ownCloud

We install ownCloud with binary distribution from [offical site](https://download.owncloud.org/community). You can find the latest release from this site, here we use `owncloud-complete-20200731`:

    $ cd /var/www/html
    $ wget https://download.owncloud.org/community/owncloud-complete-20200731.tar.bz2
    $ tar -xjf owncloud-complete-20200731.tar.bz2

Create data and other folders:

    $ mkdir -p /var/owncloud_data
    $ mkdir -p /var/www/html/owncloud/apps-external

Modify folder's owner/group and read/write permission:

    $ chown apache:apache /var/owncloud_data
    $ chown apache:apache /var/www/html/owncloud/apps-external
    $ chmod 0770 /var/owncloud_data

Configure SELinux:

    $ semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/owncloud/config'
    $ semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/owncloud/apps'
    $ semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/owncloud/apps-external'
    $ semanage fcontext -a -t httpd_sys_rw_content_t '/var/owncloud_data'
    $ restorecon '/var/www/html/owncloud/config'
    $ restorecon '/var/www/html/owncloud/apps'
    $ restorecon '/var/www/html/owncloud/apps-external'
    $ restorecon '/var/owncloud_data'

### Configure Database

Connect to MySQL with root user:

    $ mysql -u root -p

Input MySQL root user password, and create a new database:

    mysql> CREATE DATABASE IF NOT EXISTS owncloud; 

Create a new user for ownColud:

    mysql> CREATE USER 'owncloud'@'localhost' IDENTIFIED BY 'password';

Replace the `password` with your own (use strong password). Grant privileges for user `owncloud` to database `owncloud`:

    mysql> GRANT ALL PRIVILEGES ON owncloud.* To 'owncloud'@'localhost' WITH GRANT OPTION;
    mysql> FLUSH PRIVILEGES;

Run the following command, or else error will occurr when install ownCloud. The `password` is the same as the above:

    mysql> ALTER USER 'owncloud'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';

### Install OwnCloud

Now it is ready to install ownCloud. Open `http://<your server domain name or ip>/owncloud`. Input the administrator name and password. Click 'Storage & database':

* Data folder use `/var/owncloud_data`
* Database user use `owncloud`, the password use the one you create above.
* Database name use `owncloud`, leave others no change.

Click button **Finish setup**, and wait until install finished. This will take some times for ownCloud will create and initialize some database tables and setup configurations.
