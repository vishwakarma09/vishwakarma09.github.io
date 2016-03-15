---
title: website backup using google drive (via browser)
layout: post
---

# website backup using google drive (via browser)

Hi friends,

In the last post, we saved the files in the client's drive that's too when the files were uploaded from the browser. This time we are using a little console and some web as well. This is a hybrid solution when the script is executed via browser (where it performs oAuth2 token exchange with the server and then finally proceeds with uploading the content to Google Drive).

For this app too, you will need to create a 'Client ID for web application' and use its credentials in the code. NOTE: Set Redirect URIs to this page:

Connect Me!
{% highlight php %}
echo pageFooter(__FILE__);
{% endhighlight %}

Just need to change the line to contain the file name that needs to be uploaded:
{% highlight php %}
DEFINE("TESTFILE", 'latest.zip');
{% endhighlight %}

After accessing the page, check your Google Drive for the file.

Complete example accessible at :
	http://candyartist.tk/google/google-api-php-client/examples/simplefileupload.php

Your comments and suggestions are always welcome.

{% highlight php %}

{% endhighlight %}