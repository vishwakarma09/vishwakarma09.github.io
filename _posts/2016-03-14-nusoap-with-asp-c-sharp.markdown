---
title: NuSOAP with asp C sharp
layout: post
---

# NuSOAP with asp C#

Hi friends,

Working on PHP SOAP web service with asp c#, here is what to do:

The nusoap server is initialized in the index.php. The server listens for requests from php://input

Register Class methods with $server->register calls.

Nusoap full code is at https://github.com/vishwakarma09/nusoap-with-asp

Now the second thing - ASP c#

create a new windows forms application and add web service points by adding service reference. Then go in advance and also add service reference.

To call submitLogin call, use code like this

{% highlight %}
public Form1()
{
	InitializeComponent();
	server svc= new  server();
	string s = svc.MarketClass.SubmitLogin("{\"userid\":\"1\",\"activationcode\":\"abc\"}\");
	MessageBox.Show(s);
}
{% endhighlight %}

This is all to do to query data from nuSOAP services.