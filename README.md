This following wiki includes the steps to setup wordpress on centos 8 for my family blog: https://yehfamily.us/wordpress
https://github.com/Miker48/wordpress-on-centos/wiki


This wiki includes the steps to setup wordpress on centos 8 for my family blog: https://yehfamily.us/wordpress

## 1. Install CentOS

1. Download CentOS 8 iso image from http://isoredirect.centos.org/centos/8/isos/x86_64/
1. Copy ths iso to USB drive
1. Insert the USB drive to the Dell T30 server
1. Power on Dell T30, and hit F12 to enter to boot menu
1. Select boot from USB, and follow the instruction to load CentOS
 > NOTE: we need setup software mirror (RAID1) on 2 x 2 TB disks

## 2. Install MariaDB, Apache, PHP on CentOS

1. Install mariadb, apache, php rpms
 >  dnf install mariadb-server httpd php-json php-mysqlnd php-fpm 

2. Enable / Start mariadb and apahce web server
 >  systemctl enable --now mariadb httpd

3. Open firewall for http (port 80) / https (port 443) traffic
 > firewall-cmd --permanent --zone=public --add-service=https
 > firewall-cmd --permanent --zone=public --add-service=https
 > firewall-cmd --reload

4. Secure mariadb and set the root password
 >  mysql_secure_installation

5. Create the database "wordpress" and give the user "admin" access to the new created "wordpress" database with password xxx

 > mysql -u root -p     <BR>
 > mysql> CREATE DATABASE wordpress; <BR>
 > mysql> CREATE USER `admin`@`localhost` IDENTIFIED BY 'xxx';  <BR>
 > mysql> GRANT ALL ON wordpress.* TO `admin`@`localhost`;  <BR>
 > mysql> FLUSH PRIVILEGES;  <BR>
 > mysql> exit <BR>

## 3. Install  WordPress
1. Download WordPress from wordpress.org
 > wget https://wordpress.org/latest.tar.gz -O /tmp/wordpress.tar.gz

2. Extarct files in wordpress.tar.gz the to /var/www/html
 > tar xvf /tmp/wordpress.tar.gz -C /var/www/html

3. Change ownership to user / group of "apache", and change file SELinux security context:
 > chown -R apache:apache /var/www/html/wordpress
 > chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R

4. Perform the actual WordPress installation / setup by accesing http://localhost/wordpress and follow the instructions. 

5. install plugins
 >  1. Akismet Anti-Spam
 >  2. Import External Images
 >  3. Ivory Search
 >  4. PDF Embedder
 >  5. WP Githuber MD

## 4. Download / install wordpress app on iPhone
1. In order to use the wordpress app, we need to install php-xml, and restart httpd.
  > dnf install php-xml-y <BR>
  > systemctl restart httpd

## 5. Setup free SSL cert by using  "let's encrypt"
I'm using letsencrypt for free SSL cert.
1. install EPEL repo
 >  yum install https://dl.fedoraproject.org/pub/epel/epel-     release-latest-8.noarch.rpm

2. install python3-mock
 >  dnf --enablerepo=PowerTools install python3-mock

3. install cerbot
 >  sudo dnf install certbot python3-certbot-apache

4. Update /etc/httpd/conf/http.conf file and add virtual host like this in order to make certbot working

 > \#<VirtualHost 192.168.1.10>  <BR>
 > \#  DocumentRoot "/var/www/html" <BR>
 > \#  ServerName yehfamily.us <BR>
 > \#<\/VirtualHost> <BR>

and restart http service
 > systemctl restart httpd
5. get certificate
 > sudo certbot certonly --apache

6. update /etc/httpd/conf.d/ssl.conf, and add key / cert

 > SSLCertificateFile /etc/letsencrypt/live/yehfamily.us/fullchain.pem <BR>
 > SSLCertificateKeyFile  /etc/letsencrypt/live/yehfamily.us/privkey.pem


 and roll back the httpd.conf changes (remove the virtual host part)

7. restart httpd
> systemctl restart httpd
