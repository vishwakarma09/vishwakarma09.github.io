---
title: Install and configure PHP-FPM via source with Nginx. 
layout: post
---

# Install and configure PHP-FPM via source with Nginx. 

Hi friends,

Below I have install and configure PHP-FPM via source with Nginx:

{% highlight javascript %}
uname -a
ls
cd opt/
ls
wget "http://nginx.org/download/nginx-1.6.2.tar.gz"
apt-get install wget screen vim
wget "http://nginx.org/download/nginx-1.6.2.tar.gz"
tar -xzf nginx-1.6.2.tar.gz 
ll
cd nginx-1.6.2
ll
./configure 
apt-get install build-essential
aptitude update && aptitude install build-essential
apt-get install aptitude
aptitude update && aptitude install build-essential
apt-get update
apt-get install libxml2 libxml2-dev libssl-devpkg-config curl libcurl4-nss-dev enchant libenchant-dev libjpeg8 libjpeg8-dev libpng12-0 libpng12-dev libvpx1 libvpx-dev libfreetype6 libfreetype6-dev libt1-5 libt1-dev libgmp10 libgmp-dev libicu48 libicu-devmcrypt libmcrypt4 libmcrypt-devlibpspell-dev libedit2 libedit-dev libsnmp15 libsnmp-dev libxslt1.1 libxslt1-dev
apt-get install libxml2 libxml2-dev libssl-devpkg-config curl libcurl4-nss-dev enchant libenchant-dev libjpeg8 libjpeg8-dev libpng12-0 libpng12-dev libvpx1 libvpx-dev libfreetype6 libfreetype6-dev libt1-5 libt1-dev libgmp10 libgmp-devlibicu-devmcrypt libmcrypt4 libmcrypt-devlibpspell-dev libedit2 libedit-devlibsnmp-dev libxslt1.1 libxslt1-dev
cd ..
wget "http://museum.php.net/php5/php-5.4.17.tar.gz"
tar -xzf php-5.4.17.tar.gz 
cd ..
ll
cd opt/
ll
rm -rf *
cd ..
ll
mkdir /opt/php-5.5.7
mkdir /usr/local/src/php5-build
cd /usr/local/src/php5-build
wget http://de.php.net/get/php-5.5.7.tar.bz2/from/this/mirror -O php-5.5.7.tar.bz2
tar jxf php-5.5.7.tar.bz2
cd php-5.5.7/
apt-get install build-essential
apt-get build-dep php5
apt-get install libfcgi-dev libfcgi0ldbl libjpeg62-dbg libmcrypt-devlibssl-dev libc-client2007e libc-client2007e-dev
ln -s /usr/lib/libc-client.a /usr/lib/x86_64-linux-gnu/libc-client.a
./configure --help
./configure --prefix=/opt/php-5.5.7 --with-pdo-pgsql --with-zlib-dir --with-freetype-dir --enable-mbstring --with-libxml-dir=/usr --enable-soap --enable-calendar --with-curl --with-mcrypt --with-zlib --with-gd --with-pgsql --disable-rpath --enable-inline-optimization --with-bz2 --with-zlib --enable-sockets --enable-sysvsem --enable-sysvshm --enable-pcntl --enable-mbregex --enable-exif --enable-bcmath --with-mhash --enable-zip --with-pcre-regex --with-mysql --with-pdo-mysql --with-mysqli --with-jpeg-dir=/usr --with-png-dir=/usr --enable-gd-native-ttf --with-openssl --with-fpm-user=www-data --with-fpm-group=www-data --with-libdir=/lib/x86_64-linux-gnu --enable-ftp --with-imap --with-imap-ssl --with-kerberos --with-gettext --with-xmlrpc --with-xsl --enable-opcache --enable-fpm
ls -l /usr/include/freetype2/freetype.h
mkdir /usr/include/freetype2/freetype
ln -s /usr/include/freetype2/freetype.h /usr/include/freetype2/freetype/freetype.h
./configure --prefix=/opt/php-5.5.7 --with-pdo-pgsql --with-zlib-dir --with-freetype-dir --enable-mbstring --with-libxml-dir=/usr --enable-soap --enable-calendar --with-curl --with-mcrypt --with-zlib --with-gd --with-pgsql --disable-rpath --enable-inline-optimization --with-bz2 --with-zlib --enable-sockets --enable-sysvsem --enable-sysvshm --enable-pcntl --enable-mbregex --enable-exif --enable-bcmath --with-mhash --enable-zip --with-pcre-regex --with-mysql --with-pdo-mysql --with-mysqli --with-jpeg-dir=/usr --with-png-dir=/usr --enable-gd-native-ttf --with-openssl --with-fpm-user=www-data --with-fpm-group=www-data --with-libdir=/lib/x86_64-linux-gnu --enable-ftp --with-imap --with-imap-ssl --with-kerberos --with-gettext --with-xmlrpc --with-xsl --enable-opcache --enable-fpm
make
screen -ls
make install
ll
cd ..
ll
cd ..
ll
cd ..
ll
cd /
ll
mkdir /opt/nginx-1.6
mkdir /usr/local/src/nginx-build
cd /usr/local/src/nginx-build/
ll
wget "http://nginx.org/download/nginx-1.6.2.tar.gz"
tar -xzf nginx-1.6.2.tar.gz 
ll
cd nginx-1.6.2
ll
./configure --help
cp /usr/local/src/php5-build/php-5.5.7/php.ini-production /opt/php-5.5.7/lib/php.ini
cp /opt/php-5.5.7/etc/php-fpm.conf.default /opt/php-5.5.7/etc/php-fpm.conf
vim /opt/php-5.5.7/etc/php-fpm.conf
mkdir /opt/php-5.5.7/etc/pool.d
vi /etc/init.d/php-5.5.7-fpm
chmod 755 /etc/init.d/php-5.5.7-fpm
ll
insserv php-5.5.7-fpm
/etc/init.d/php-5.5.7-fpm start
netstat -tulpn | grep :9000
php -v
ls
free -m
echo 3 > /proc/sys/vm/drop_caches
free -m
apt-get install build-essential zlib1g-dev libpcre3-dev libssl-dev libxslt1-dev libxml2-dev libgd2-xpm-dev libgeoip-devlibgoogle-perftools-devlibperl-dev
ls
./configure --help
mkdir -P /var/log/nginx/
mkdir -p /var/log/nginx/
./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_spdy_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module --with-http_image_filter_module --with-http_geoip_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_stub_status_module --with-http_perl_module --with-mail --with-mail_ssl_module --with-pcre --with-google_perftools_module --with-debug
make
useradd -r nginx
service nginx start
make install
service nginx start
make clean
ll
make dist-clean
make help
make --help
make
ll
$ ./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_spdy_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module --with-http_image_filter_module --with-http_geoip_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_stub_status_module --with-http_perl_module --with-mail --with-mail_ssl_module --with-pcre --with-google_perftools_module --with-debug
./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_spdy_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module --with-http_image_filter_module --with-http_geoip_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_stub_status_module --with-http_perl_module --with-mail --with-mail_ssl_module --with-pcre --with-google_perftools_module --with-debug
make
make install
cd /usr/local/nginx/
cd ../sbin/nginx
cd ../sbin/nginx/
cd .
cd ..
ll
cd  sbin/
ll
./nginx start
./nginx
netstat -tulpn | grep :80
cd ../bin/
ll
cd ..
ll
cd bin/
ll
cd ../
ll
cd nginx/
ll
cd html/
ll
vim index.html 
ls
cd ..
ll
cd ..
ll
cd /opt/php-5.5.7/etc/pool.d/
vim www.conf
/etc/init.d/php-5.5.7-fpm stop
/etc/init.d/php-5.5.7-fpm start
netstat -tulpn | grep :9000
netstat -tulpn | grep :900
netstat -tulpn | grep 990
netstat -tulpn | grep :80
vim /etc/nginx/fastcgi.conf
cd /etc/nginx/
ll
mkdir sites-availabe sites-enabbled
cd sites-availabe/
vim default
vim /usr/local/nginx/html/info.php
cd /opt/php-5.5.7/sbin/
ll
/etc/init.d/php-5.5.7-fpm stop
/etc/init.d/php-5.5.7-fpm start
ps
s aux
ps aux
kill -9 2123
kill -9 11581
kill -9 11582
kill -9 11599
kill -9 11600
kill -9 11601
kill -9 11693
ps aux
kill -9 2124
kill -9 2125
ps aux
/etc/init.d/php-5.5.7-fpm start
/usr/local/sbin/nginx start
/usr/local/sbin/nginx --help
/usr/local/sbin/nginx
netstat -tulpn | grep :9000
/etc/init.d/php-5.5.7-fpm stop
netstat -tulpn | grep :9000
/etc/init.d/php-5.5.7-fpm start
ps aux
kill -9 11774
vim /etc/nginx/sites-availabe/default 
ps aux
/etc/init.d/php-5.5.7-fpm stop
/etc/init.d/php-5.5.7-fpm start
ps aux
kill -9 11775
ps aux
/usr/local/sbin/nginx
ps aux
netstat -tulpn | grep :9000
ls -l /opt/php-5.5.7/var/run/php-fpm.pid
history
netstat -tulpn | grep :9000
ps aux
ls
ps aux
ll
 ..
ll
cd bin/
ll
php -v
./php -v
./php -i
./php -m
path
env
echo $PATH
ll
vim /etc/profile.d/php-nginx-path.sh
echo $PATH
vim /etc/profile.d/php-nginx-path.sh
export PATH=$PATH:/opt/php-5.5.7/bin
echo $PATH
php -v
service php-fpm start
/etc/init.d/php-5.5.7-fpm start
/etc/init.d/php-5.5.7-fpm status
/etc/init.d/php-5.5.7-fpm force-quit
ps aux
vim /opt/php-5.5.7/var/run/php-fpm.pid
ll
cd .
cd ..
ll
cd sbin/php-fpm 
cd sbin/
ll
./php-fpm s
./php-fpm -t
./php-fpm -i
./php-fpm -t
./php-fpm
ls -l /var/run/php5-fpm.sock
ps aux
vim /etc/nginx/nginx.conf
vim /etc/nginx/sites-availabe/default 
vim /etc/nginx/nginx.conf
ps aux
kill 11794
ps aux
/usr/local/sbin/nginx
ps aux
vim /opt/php-5.5.7/etc/php-fpm.conf
/etc/init.d/php-5.5.7-fpm restart
s aux
ps aux
kill -9 11698
ps aux
kill -9 11699
kill -9 11700
ps aux
/etc/init.d/php-5.5.7-fpm start
ps aux
history | les
history | less
cd /var/log/nginx/error.log 
cat /var/log/nginx/error.log 
ls
netstat -tulpn | grep :80
netstat -tulpn | grep :9000
vim /opt/php-5.5.7/etc/pool.d/www.conf 
vim /opt/php-5.5.7/etc/php-fpm.conf
ll /var/run/php5-fpm.sock
vim /opt/php-5.5.7/etc/php-fpm.conf
vim /opt/php-5.5.7/etc/pool.d/www.conf
rm /opt/php-5.5.7/etc/pool.d/www.conf
ps aux
kill -9 11863
ps aux
kill -9 11848
ps aux
ll
cd ..
ll
cd var/
ll
cd log/
ll
vim php-fpm.log 
ll
cd ..
ll
cd run/
ll
pwd
vim /etc/nginx/nginx.conf
ll /var/run/php5-fpm.sock 
vim /usr/local/nginx/fastcgi_temp/
vim /var/log/nginx/access.log 
vim /var/log/nginx/error.log 
ll
netstat -tulpn | grep :80
netstat -tulpn | grep :9000
ps aux
/etc/init.d/php-5.5.7-fpm start
ps aux
/etc/init.d/php-5.5.7-fpm 
/etc/init.d/php-5.5.7-fpm start
/etc/init.d/php-5.5.7-fpm stop
/etc/init.d/php-5.5.7-fpm start
ps aux
kill -9 11864
kill -9 11865
ps aux
/etc/init.d/php-5.5.7-fpm start
ps aux
netstat -tulpn | grep :9000
vim /var/log/nginx/access.log 
tail -f /var/log/nginx/access.log 
vim /usr/local/nginx/html/info.php
tail -f /opt/php-5.5.7/var/log/php-fpm.log 
ps aux
vim /etc/init.d/php-5.5.7-fpm 
ls -l /opt/php-5.5.7/var/run/php-fpm.pid
find / -name php-fpm.pid
find / -name *.pid
vim /etc/init.d/php-5.5.7-fpm 
cd ..
cd ..
ll
cd sbin/
ll
./php-fpm
./php-fpm help
vim /etc/init.d/php-5.5.7-fpm 
cd /usr/src/
ll
cd /usr/local/src/php5-build/php-5.5.7/
ll
./configure --help
ls
cd /opt/php-5.5.7/
ll
cd etc/
ll
viphp-fpm.conf
ls
netstat -tulpn | grep : 9000
netstat -tulpn | grep :9000
vi /opt/php-5.5.7/etc/php-fpm.conf
ps aux
kill -9 11991
ps aux
netstat -tulpn | grep :9000
/etc/init.d/php-5.5.7-fpm start
netstat -tulpn | grep :9000
vi /etc/nginx/nginx.conf
ll
vi /etc/nginx/fastcgi.conf
vi /etc/nginx/nginx.conf
ps aux
kill -9 11849
ps aux
/usr/local/sbin/nginx
ps aux
vi /etc/nginx/nginx.conf
ps aux
kill -9 12287
ps aux
kill -9 12288
/usr/local/sbin/nginx
ps aux
vi /etc/nginx/nginx.conf
vi /etc/nginx/sites-availabe/
vi /etc/nginx/sites-availabe/default 
vi /etc/nginx/fastcgi_params
vi /etc/nginx/nginx.conf
cd /etc/nginx/
ll
vinginx.conf
ls
ls sites-enabbled/
rm sites-availabe/default 
ll
vinginx.conf.default
vinginx.conf
/usr/local/sbin/nginx
ll
vinginx.conf
ll
mkdirssl
cd ssl/
ll
sudoopensslreq -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
cd ..
vinginx.conf
ps aux
kill -9 12309
ps aux
kill -9 12310
ps aux
/usr/local/sbin/nginx
vinginx.conf
ps aux
kill -9 12331
kill -9 12332
ps aux
/usr/local/sbin/nginx
vinginx.conf
ps aux
kill -9 12337
kill -9 12338
/usr/local/sbin/nginx
vinginx.conf
ps aux
kill -9 12342
kill -9 12343
/usr/local/sbin/nginx
vinginx.conf
ps aux
kill -9 12347
kill -9 12348
/usr/local/sbin/nginx
vinginx.conf
/usr/local/sbin/nginx
vinginx.conf
/usr/local/sbin/nginx
vinginx.conf
/usr/local/sbin/nginx
vinginx.conf
/usr/local/sbin/nginx
vinginx.conf
/usr/local/sbin/nginx
vinginx.conf
/usr/local/sbin/nginx
vinginx.conf
/usr/local/sbin/nginx
vinginx.conf
/usr/local/sbin/nginx
ps aux
kill -9 12367
kill -9 12368
ps aux
vinginx.conf
netstat -tulpn | grep :80
netstat -tulpn | grep :9000
netstat -tulpn | grep :8080
netstat -tulpn | grep :443
/usr/local/sbin/nginx
curl -I "domain1.com"
vi /etc/hosts
curl -I "domain1.com"
ping domain1.com
vi /etc/hosts
ping domain1.com
curl -I "domain1.com"
netstat -tulpn | grep :[80-8080]
vi /etc/nginx/nginx.conf
ps aux
kill -9 12383
kill -9 12384
/usr/local/sbin/nginx
root@1f5d62208f8f:/etc/nginx# netstat -putan|grep LISTEN
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      -               
root@1f5d62208f8f:/etc/nginx# php -m |grepmcrypt
mcrypt
root@1f5d62208f8f:/etc/nginx#
{% endhighlight %}

This installs the php with mcrypt module.