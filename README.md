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


sudo apt-get install phpmyadmin
sudo ln -s /usr/share/phpmyadmin/ /usr/share/nginx/www



https://www.digitalocean.com/community/articles/how-to-install-phpmyadmin-on-a-lemp-server
sudo service nginx restart


