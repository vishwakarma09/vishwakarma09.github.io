---
title: Upload server backup directly to dropbox
layout: post
---

# Upload server backup directly to dropbox

Hi friends,

Drop-box provides an elegant way to upload, download and view filed.  It is a good choice for storing personal files with the advantage of sharing public links. Today, lets take a peep into dropbox PHP api and see how it can be auomate the task of creating online backups in no time.

1. To get started, you need to create a dropbox app by loging into your dropbox account at https://www.dropbox.com/developers/apps
2. Once done with app creation, download the SDK. I am using PHP SDK available at https://www.dropbox.com/developers/core/sdks/php
As mentioned in the above guide, create a .json file containing the app credentials. After first authentication, your app will get access to all the dropbox files.
3. To list all files, use the below PHP CLi command:

{% highlight php %}

{% endhighlight %}