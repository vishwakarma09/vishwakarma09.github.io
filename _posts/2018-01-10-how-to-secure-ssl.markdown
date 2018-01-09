---
title: How to secure your SSL sites
layout: post
---

# How to secure your SSL sites

Hi friends,

There is SSLv3 vulnerability and other on SSLv2 also. Now TLSv2 will come handy. So this is the configuration for hardening:

{% highlight javascript %}
#Edit webserver Config File for disabling SSLV2, V3 & Weak Ciphers

#SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
SSLCipherSuite ALL:!aNULL:!eNULL:!EXP:!LOW:!MD5:!RC4:!SHA1:!DH:!DES:!3DES
SSLProtocol All -SSLv2 -SSLv3 -TLSv1.0
SSLHonorCipherOrder On
Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains; preload"
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff

# This setting will work on Apache  versions >= 2.4
SSLCompression off 
SSLSessionTickets Off
SSLUseStapling on 


#Enter the following line in : /etc/apache2/mods-available/ssl.conf
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
{% endhighlight %}