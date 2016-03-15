---
title: Automatic Deploy website script
layout: post
---

# Automatic Deploy website script	

Hello fellas,

Here I am providing a script to automatically install and configure your server for a domain for Debian style system (Ubuntu, Debian, etc..). The only argument provided to script is the name of website. It will automatically do the following:

- Create document root.
- Create logs directories.
- Create access and error logs.
- Copy sample config to apache.
- Edit the sample config to meet the new website.
- Enable the site.
- Restart the web service.

After the script has been invoked, you can access the site.

{% highlight javascript %}
function success()
{
if [ $? == 0 ]
then
	echo "[OK]"
else
	echo "[Failed]"
fi
}

echo "installing $1 domain on web server..."
doc=$1
echo "Creating document root" && mkdir -p /var/www/html/$doc && success
echo "creating logs directory" && mkdir -p /var/log/apache2/$doc && success
echo "creating access logs" && touch /var/log/apache2/$doc/access.log && success
echo "creating error logs" && touch /var/log/apache2/$doc/error.log && success
echo "copying sample config to apache" && cp sample.cfg /etc/apache2/sites-available/$doc
echo "creating configuration file" && cat sample.cfg | sed "s/sandzvps.tk/$doc/g" >  /etc/apache2/sites-available/$doc && success
echo "enabling site" && a2ensite $doc && success
echo "restarting apache2" && service apache2 restart && success
{% endhighlight %}