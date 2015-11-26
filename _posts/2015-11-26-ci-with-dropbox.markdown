---
title: CI with Dropbox
layout: post
---

# CI with Dropbox

Hi friends,

Now-a-days I am busy building a online tour page; A part of the project is reviews - that happen too often. I develop on local XAMPP environment and has to deploy all the changes to my Ubuntu server. All the code happens so rapidly that it is difficult to keep track of which files to upload.

I have tried installing vsftpd on server, but it was a trouble to keep track of files.

Today I came across dropbox CLI; and I thought - why not!

So, I installed on server:

{% highlight javascript %}
cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -
~/.dropbox-dist/dropboxd
{% endhighlight %}

On first run, you get a URL; that you have to paste on your local browser and login with your dropbox details. Tadaa! all done. Dropbox synced all my local dropbox files on to server. They are under *~/Dropbox* folder

# Problem

Althogh I can see my local files on server, I can not see how my */var/www/html/project/* files will be available to my local. Then I read somewhere in dropbox page that in windows you can sync files out of dropbox folder by creating shortcut. Wao! I created symbolic link to my project by:

{% highlight javascript %}
cd ~/Dropbox && ln -s /var/www/html/project/ .
{% endhighlight %}

I opened my local dropbox agent; and Voila! the files are syncing ;)

##Credits

https://www.dropbox.com/install?os=lnx
https://www.dropbox.com/en/help/12
