# Nextcloud installation on Centos 7 with php7

## OS and repo preparation

Add the EPEL repo and the IUS repo for php7:

```
yum install -y epel-release
yum install -y http://dl.iuscommunity.org/pub/ius/stable/CentOS/7/x86_64/ius-release-1.0-14.ius.centos7.noarch.rpm
yum -y update
```

Install apache and php7:

```
yum -y install httpd sqlite php70u php70u-mysqlnd php70u-mysql php70u-dom php70u-opcache php70u-pdo php70u-mbstring php70u-gd php70u-devel php70u-intl php70u-pdo php70u-json php70u-xml php70u-zip php70u-gd  php70u-curl php70u-mcrypt php70u-pear php70u-bcmath php70u-iconv php70u-pecl-smbclient php70u-ldap openldap-clients

systemctl start httpd ; systemctl enable httpd
```

Install and initialize mariadb:

```
yum install -y mariadb-server mariadb
systemctl start mariadb; systemctl enable mariadb

mysql_secure_installation
```

Create nextcloud database and user:
```
mysql -u root -p

create database nextcloud_db;
grant all on nextcloud_db.* to 'nxtuser'@'localhost' identified by 'somepassword';
flush privileges;
```


Download, unpack and fix permissions:
```
wget https://download.nextcloud.com/server/releases/latest.tar.bz2
tar xvfj latest.tar.bz2 -C /var/www/html
chown -R apache:apache /var/www/html/nextcloud
```


## Apache 

Set up HSTS in /etc/httpd/conf.d/ssl.conf:

```
<IfModule mod_headers.c>
        Header always set Strict-Transport-Security "max-age=15768000; includeSubDomains; preload"
</IfModule>
```

Set the standard stuff on /etc/httpd/conf/httpd.conf: `AllowOverride`, `ServerName`, `ServerAdmin` etc...


## SSL

Install mod_ssl, your certs and optionally [enforce https](https://github.com/Simone-Zabberoni/misc-one-liners/blob/master/APACHE.md)


## php.ini and .htaccess

Set various params according to your setup (ie: `post_max_size`).
**Important**: nextcloud's own .htaccess (`/var/www/html/nextcloud/.htaccess`) could override them:

```
<IfModule mod_php7.c>
  php_value upload_max_filesize 511M
  php_value post_max_size 511M
  php_value memory_limit 512M
  php_value mbstring.func_overload 0
  php_value default_charset 'UTF-8'
  php_value output_buffering 0
  <IfModule mod_env.c>
    SetEnv htaccessWorking true
  </IfModule>
</IfModule>
```

The override behavoir is controlled by Apache, see the `AllowOverride` directive.



## Opcache

Nextcloud suggests the use of opcache for better performances.

Usually the opcache related configuration is stored in `/etc/php.d/`, replace it with:

```
# cat /etc/php.d/10-opcache.ini
; Enable Zend OPcache extension module
zend_extension=opcache.so

opcache.enable=1
opcache.enable_cli=1
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.memory_consumption=128
opcache.save_comments=1
opcache.revalidate_freq=1
```

## SELINUX

See the official guide: 
[https://docs.nextcloud.com/server/13/admin_manual/installation/selinux_configuration.html](https://docs.nextcloud.com/server/13/admin_manual/installation/selinux_configuration.html) or switch to *permissive* while testing.




## Ready

Restart httpd to apply the configuration (`systemctl restart httpd`)

Connect to http://cloud.mydomain.tld/nextcloud and follow the instructions


## Post-install  stuff

Set up crontab action under the apache user. 
You'll need at least the cron.php job and it's useful to run periodically the occ scan:
```
crontab -l -u apache
*/15 * * * * php -f /var/www/html/cron.php
*/15 * * * * php -f /var/www/html/occ  files:scan --all
```

**TODO**: logrotate script for /var/www/html/data/nextcloud.log






