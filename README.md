所為何來
=
於 Ubuntu 上安裝 Nignx 做為網站伺服器，搭配 MariaDB 資料庫及 PHP 指令碼語言 (即 LEMP 架構)，並以 Adminer 做為網頁資料庫管理工具，再另外加裝 Varnish 反向網站快取伺服器 (HTTP 加速器的一種)。本文將列出相對簡易之安裝步驟，供需求者參考。
  
本文 Ubuntu 的環境設定及帳號權限依存於[此文](https://github.com/meyu/Initial-Ubuntu-HTTP-Server)，也請參考。
  
適用環境
=
作業系統：Ubuntu Server 12.04  
網頁伺服器：Nginx 1.5.3  
資料庫系統：MariaDB 5.5.32  
指令碼語言：PHP 5.3.10  
資料庫管理介面：Adminer 3.7.1  
反向快取伺服器：Varnish 3.0.4   

安裝方式
=
###安裝 MariaDB
登入 Ubuntu。  
(本文使用 <code>WebAdmin</code> 登入，為網站管理者的專用身份，具有可使用 sudo 的權限)
  
於終端機中輸入以下指令，以下載及安裝 MariaDB 和相關套件：  
(本文安裝的版本為 5.5，欲選擇其它版本者，可至 [MariaDB 官網](https://downloads.mariadb.org/mariadb/repositories/) 取得下載指令，並替代以下指令的前三行)
```bash
sudo aptitude install python-software-properties && 
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xcbcb082a1bb943db && 
sudo add-apt-repository 'deb http://download.nus.edu.sg/mirror/mariadb/repo/5.5/ubuntu precise main' && 
sudo aptitude update && 
sudo apt-get install mariadb-server libapache2-mod-auth-mysql php5-mysql
```
安裝過程中，請設定資料庫的最高權限使用者 root 的密碼 (本文使用 <code>root.password</code>)。
接著，請進行安全性設定：
```bash
sudo /usr/bin/mysql_secure_installation
```
過程中，需要輸入剛剛設定 root 的密碼，以提供權限進行操作；
```text
Enter current password for root (enter for none): 
```
詢問是否要更換 root 的密碼時，請選 n；再來是幾項安全性的選擇，建議都選擇 y：  
(更換 root 密碼、移除匿名登入、關閉root遠端登入、移除test資料庫、立刻套用設定)
```text
Change the root password? [Y/n] n
Remove anonymous users? [Y/n] y                                            
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```
   
###安裝 Nginx
請輸入：
```bash
sudo aptitude install nginx &&
sudo service nginx start 
```
   
###安裝 PHP
請輸入以下指令，以安裝PHP，並編輯其設定，以提高安全性：
```bash
sudo aptitude install php5-fpm &&  
sudo nano /etc/php5/fpm/php.ini
```
找到 <code>;cgi.fix_pathinfo=1</code> 這行，將 1 改為 0：(記得要刪除句首的分號，以取消其註解狀態)
```text
cgi.fix_pathinfo=0
```
儲存後退出。接著請更改 PHP-FPM 的設定：
```bash
sudo nano /etc/php5/fpm/pool.d/www.conf
```
找到 <code>listen = 127.0.0.1:9000</code> 這行，並將其改為：
```text
listen = /var/run/php5-fpm.sock
```
儲存後退出。接著，請重新啟動 php-fpm：
```bash
sudo service php5-fpm restart
```
   
###設定 Nginx
請開啟 Nginx 的預設檔：
```bash
sudo nano /etc/nginx/sites-available/default
```
於 Server {} 段落中進行以下修改：
* 在 <code>index index.html index.htm;</code> 那行中增加 <code>index.php</code> 
* 可將 <code>server_name localhost;</code> 的 localhost 修改為特定域名或 IP
* 取消 <code>location ~ \.php$ {}</code> 段落的井字註解符號，並於其中增加一行 <code>fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;</code>

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
另請記得，/usr/share/nginx/www 為 Nginx 放置網站資料的位置；但因 Ubuntu 版本差異，亦有可能會設定為 /usr/share/nginx/html，故請自行前往 /usr/share/nginx 資料夾查看，並依實際情況修改上文所有之 /usr/share/nginx/www 設定。  
儲存後退出，並重新啟動 Nginx：
```bash
sudo service nginx restart
```
將 /usr/share/nginx/www 的使用權限分配予網站管理者：  
(本文的管理者為 <code>WebAdmin</code>)
```bash
sudo chown -R WebAdmin:WebAdmin /usr/share/nginx/www && 
chmod -R 775 /usr/share/nginx/www
```
   
### 測試運作情況
請製作一 info.php 檔，內含顯示 PHP 運作資訊的指令：
```bash
sudo cat <<EOF>/usr/share/nginx/www/info.php
<?php
phpinfo();
?>
EOF
```
使用瀏覽器查看您的網頁伺服器，並後綴 URL <code>/info.php</code>。  
(您的網址看起來會類似 http://10.10.10.10/info.php 或 http://example.com/info.php)  
若有秀出各類 PHP 資訊，即代表運作正常；如出現錯誤畫面，請再參考一次 Nginx 及 PHP 的設定步驟。
   
###安裝 Adminer
請輸入以下指令，以下載並安裝 Adminer：
```bash
cd /usr/share/nginx/www &&
sudo wget http://www.adminer.org/latest.php && 
sudo mv latest.php adminer.php
```
欲使用 Adminer 時，以瀏覽器查看您的網頁伺服器，並後綴 URL <code>/adminer.php</code>。     
(您的網址看起來會類似 http://10.10.10.10/adminer.php 或 http://example.com/adminer.php)  
   
###安裝 Varnish
請輸入以下指令，以添加 Varnish 的套件來源，並安裝之：
```bash
sudo curl http://repo.varnish-cache.org/debian/GPG-key.txt | sudo apt-key add - && 
echo "deb http://repo.varnish-cache.org/ubuntu/ precise varnish-3.0" | sudo tee -a /etc/apt/sources.list && 
sudo aptitude update && sudo aptitude install varnish
```
   
###設定 Varnish 與 Nginx
編輯其 <code>/etc/default/varnish</code>，使 Varnish 前端監聽 Port 80，做為 HTTP 的預設窗口：
```bash
sudo nano /etc/default/varnish
```
在 Alternative 2 的段落中，修改 DAEMON_OPTS 的 -a 參數，由 <code>6081</code> 改為 <code>80</code>：
```text
DAEMON_OPTS="-a :80 \
             -T localhost:6082 \
             -f /etc/varnish/default.vcl \
             -S /etc/varnish/secret \
             -s malloc,256m"
```
儲存後退出。  
查看 /etc/varnish/default.vcl 的 backend default {} 段落，可知 Varnish 的後端監聽 Port 8080；  
為使 Nginx 能與 Varnish 合作，請再次編輯 /etc/nginx/sites-available/default，使 Nginx 的前端監聽 Port 8080：
```bash
sudo nano /etc/nginx/sites-available/default
```
將 Server {} 段落中首行的 <code>listen 80;</code> 改為 <code>listen 8080;</code>：
```text
 listen  8080; ## listen for ipv4; this line is default and implied
```
儲存後退出。這時，可先檢查 Nginx 的設定檔語法：
```bash
sudo nginx -t
```
重新啟動 Nginx 及 Varnish。
```bash
sudo service nginx restart && sudo service varnish restart
```
   
DONE.
<br>
<br>

補充說明
=
###本文網站設定資訊

* 網站資料：<code>/usr/share/nginx/www/</code>
* 管理者：<code>WebAdmin</code>
* 密碼：<code>web.password</code>
* 本機網址：[http://localhost/](http://localhost/)
* 本機測試：[http://localhost/info.php](http://localhost/info.php)

###本文資料庫設定

* 網頁管理介面入口：[http://localhost/adminer.php](http://localhost/adminer.php)
* 資料庫系統管理者 <code>root</code>
* 資料庫系統密碼 <code>root.password</code>

###相關設定檔
* PHP：<code>/etc/php5/fpm/php.ini</code> 及 <code>/etc/php5/fpm/pool.d/www.conf</code> 及 <code>/etc/php5/fpm/pool.d/user.conf</code>
* Nginx：<code>/etc/nginx/sites-available/default</code> 及 <code>/etc/nginx/nginx.conf</code>
* Varnish：<code>/etc/default/varnish</code> 及 <code>/etc/varnish/default.vcl</code>

###補強  

* 安全考量下，如您不需要透過網頁介面管理資料庫時，建議移除 Adminer：  
```bash
sudo rm /usr/share/nginx/www/adminer.php
```
* 如要測試 Varnish 運作情況，可使用以下指，並參考[此文](http://helpdesk.getpantheon.com/customer/portal/articles/425726)：  
```bash
curl -I http://dev.pantheon.gotpantheon.com/
```


參考資源
=
* [MariaDB](https://mariadb.org/)
* [Adminer - Database management in single PHP file](http://www.adminer.org/)
* [Installation on Ubuntu | Varnish Community](https://www.varnish-cache.org/installation/ubuntu)
* [Installation | TuxLite](https://tuxlite.com/installation/)
* [How to Install Linux, nginx, MySQL, PHP (LEMP) stack on Ubuntu 12.04 | DigitalOcean](https://www.digitalocean.com/community/articles/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-12-04)
* [How to Install Wordpress, Nginx, PHP, and Varnish on Ubuntu 12.04 | DigitalOcean](https://www.digitalocean.com/community/articles/how-to-install-wordpress-nginx-php-and-varnish-on-ubuntu-12-04)
* [Pantheon | Varnish caching for high performance](http://helpdesk.getpantheon.com/customer/portal/articles/425726)
