Build-up-Drupal-on-Ubuntu--LEMP-Drush-Sytle-
============================================

於 Ubuntu 上架設 Drupal (LEMP 架構 + Drush 工具)


sudo apt-get install python-software-properties && 
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xcbcb082a1bb943db && 
sudo add-apt-repository 'deb http://ftp.yz.yamagata-u.ac.jp/pub/dbms/mariadb/repo/5.5/ubuntu precise main' && 
sudo apt-get update && 
sudo apt-get install mariadb-server libapache2-mod-auth-mysql php5-mysql && 
sudo /usr/bin/mysql_secure_installation

sudo apt-get install nginx &&
sudo service nginx start 


sudo nano /etc/php5/fpm/php.ini

cgi.fix_pathinfo=0

sudo nano /etc/php5/fpm/pool.d/www.conf

listen = /var/run/php5-fpm.sock

sudo service php5-fpm restart


sudo nano /etc/nginx/sites-available/default


sudo nano /usr/share/nginx/www/info.php

sudo service nginx restart


INSTALL ADMINER
```bash
cd /usr/share/nginx/www &&
sudo wget http://www.adminer.org/latest.php && 
sudo mv latest.php adminer.php
```
http://10.10.10.10/adminer

Install Varnish
```bash
sudo curl http://repo.varnish-cache.org/debian/GPG-key.txt | sudo apt-key add -
```
```bash
echo "deb http://repo.varnish-cache.org/ubuntu/ precise varnish-3.0" | sudo tee -a /etc/apt/sources.list
```
```bash
sudo apt-get update && sudo apt-get install varnish
```
```bash
sudo nano /etc/default/varnish
```
```text
START=no
NFILES=131072
MEMLOCK=82000

DAEMON_OPTS="-a :80 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
```
```bash
sudo nano /etc/nginx/sites-available/default
```
```text
[...]
server {
        listen  127.0.0.1:8080; ## listen for ipv4; this line is default and implied
        [...]
```
```bash
sudo service nginx restart && sudo service varnish restart
```




[Adminer - Database management in single PHP file](http://www.adminer.org/)
