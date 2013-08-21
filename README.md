所為何來
=
於 Ubuntu 上安裝 Nignx 做為網站伺服器，搭配 MariaDB 資料庫及 PHP 指令碼語言 (即 LEMP 架構)，並以 Adminer 做為資料庫管理工具，再另外加裝 Varnish 反向網站快取伺服器 (HTTP 加速器的一種)。  
本文將列出相對簡易之安裝步驟，供需求者參考。

使用環境
=
Ubuntu Server 12.04  
Nginx 1.5.3  
MariaDB 5.5.32  
PHP 5.5.2 
Adminer 3.7.1  
Varnish 3.0.4   

安裝方式
=
###安裝 MariaDB
登入 Ubuntu 後，於終端機中輸入以下指令，以下載及安裝 MariaDB 和相關套件：  
(本文安裝的版本為 5.5，欲安裝其它版本者，可至 [MariaDB 官網](https://downloads.mariadb.org/mariadb/repositories/) 取得下載指令，並替代以下指令的一至三行)
```bash
sudo apt-get install python-software-properties && 
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xcbcb082a1bb943db && 
sudo add-apt-repository 'deb http://ftp.yz.yamagata-u.ac.jp/pub/dbms/mariadb/repo/5.5/ubuntu precise main' && 
sudo apt-get update && 
sudo apt-get install mariadb-server libapache2-mod-auth-mysql php5-mysql
```
安裝過程中，請設定資料庫的最高權限使用者 root 的密碼。(本文使用 <code>root.password</code>)  
![Set Password for MariaDB root](https://lh5.googleusercontent.com/-FA1tujVUiDI/UhTDNnFiJYI/AAAAAAAAf8w/z-eoN762eZ0/w826-h551-no/Set+Password+for+MariaDB+root.png)
接著，請進行安全性設定：
```bash
sudo /usr/bin/mysql_secure_installation
```
過程中，需要輸入 root 的密碼，以提供權限進行設定。(本文 root 的密碼為 <code>root.password</code>)  
```text
Enter current password for root (enter for none): 
OK, successfully used password, moving on...
```
詢問是否要更換 root 的密碼時，請選 n。  
再來是幾項安全性的選擇，建議都選擇 y: 
```text
By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for them.  This is intended only for testing, and to make the installation go a bit smoother.  You should remove them before moving into a production environment.
Remove anonymous users? [Y/n] y                                            

Normally, root should only be allowed to connect from 'localhost'.  This ensures that someone cannot guess at the root password from the network.
Disallow root login remotely? [Y/n] y

By default, MySQL comes with a database named 'test' that anyone can access.  This is also intended only for testing, and should be removed before moving into a production environment.
Remove test database and access to it? [Y/n] y

Reloading the privilege tables will ensure that all changes made so far will take effect immediately.
Reload privilege tables now? [Y/n] y
```

###安裝 Nginx
請輸入：
```bash
sudo apt-get install nginx &&
sudo service nginx start 
```

###安裝 PHP
請輸入以下指令，以安裝PHP，並編輯其設定，以提高安全性：
```bash
sudo apt-get install php5-fpm &&  
sudo nano /etc/php5/fpm/php.ini
```
找到 <code>;cgi.fix_pathinfo=1</code> 這行，將 1 改為 0：  
(記得要刪除句首的分號，以取消其註解狀態)
```text
cgi.fix_pathinfo=0
```
儲存後退出。  
再來，請更改其 php5-fpm 的設定：
```bash
sudo nano /etc/php5/fpm/pool.d/www.conf
```
找到 <code>listen = 127.0.0.1:9000</cdoe> 這行，並將其改為：
```text
listen = /var/run/php5-fpm.sock
```
儲存後退出。  
接著，請重新啟動 php-fpm：
```bash
sudo service php5-fpm restart
```

###設定 Nginx
請開啟 Nginx 的 host 預設檔：
```bash
sudo nano /etc/nginx/sites-available/default
```
於 <code>Server {}</code> 段落中進行以下修改：
* 在 <code>index index.html index.htm;</code> 那行中增加 <code>index.php</code> 
* 主機如有特定域名 (Domain Name) 或 IP，可將 <code>server_name localhost;</code> 中的 <code>localhost</code> 替換為域名或 IP。 
* 取消 <code>location ~ \.php$ {}</code> 段落的井字註解符號
段落 <code>Server {}</code> 修改後，將類似：

```text
 [...]
server {
        listen   80;
     

        root /usr/share/nginx/www;
        index index.php index.html index.htm;

        server_name example.com;

        location / {
                try_files $uri $uri/ /index.html;
        }

        error_page 404 /404.html;

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
              root /usr/share/nginx/www;
        }

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        location ~ \.php$ {
                #fastcgi_pass 127.0.0.1:9000;
                # With php5-fpm:
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
                
        }

}
[...]
```


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
sudo curl http://repo.varnish-cache.org/debian/GPG-key.txt | sudo apt-key add - && 
echo "deb http://repo.varnish-cache.org/ubuntu/ precise varnish-3.0" | sudo tee -a /etc/apt/sources.list && 
sudo apt-get update && sudo apt-get install varnish
```
```bash
sudo nano /etc/default/varnish
```
```text
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







DONE.
<br>
<br>

補充說明
=
* 

參考資源
=
* [MariaDB](https://mariadb.org/)
* [Adminer - Database management in single PHP file](http://www.adminer.org/)
* [How to Install Linux, nginx, MySQL, PHP (LEMP) stack on Ubuntu 12.04 | DigitalOcean](https://www.digitalocean.com/community/articles/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-12-04)
