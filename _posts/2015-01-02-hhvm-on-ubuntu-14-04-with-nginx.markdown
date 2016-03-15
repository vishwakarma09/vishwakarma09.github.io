---
title: HHVM on Ubuntu 14.04 with Nginx
layout: post
---

# HHVM on Ubuntu 14.04 with Nginx

Hi,

Below commands:

{% highlight javascript %}
wget -O - http://dl.hhvm.com/conf/hhvm.gpg.key | apt-key add *-*
echo deb http://dl.hhvm.com/ubuntu saucy main | tee /etc/apt/sources.list.d/hhvm.list
apt-get update
apt-get install hhvm
service apache2 stop
service httpd status
service hhvm status
mkdir -p /var/www/
cd /var/www/
ll
vim hello.php
hhvm -m server
ls /usr/share/hhvm/install_fastcgi.sh
/usr/share/hhvm/install_fastcgi.sh
service apache2 status
apt-get install nginx
service nginx status
/usr/share/hhvm/install_fastcgi.sh
service nginx status
service nginx start
service nginx status
{% endhighlight %}

{% highlight javascript %}
root@kvmnode:~# php -v
HipHop VM 3.0.1 (rel)
Compiler: tags/HHVM-3.0.1-0-g97c0ac06000e060376fdac4a7970e954e77900d6
Repo schema: a1146d49c5ba0d6db903beb3a4ed8a3766fef182
root@kvmnode:~#
{% endhighlight %}
