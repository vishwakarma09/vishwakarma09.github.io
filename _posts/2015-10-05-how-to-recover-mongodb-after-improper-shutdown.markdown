---
title: How to recover mongoDB after improper shutdown
layout: post
---

# How to recover mongoDB after improper shutdown

{% highlight javascript %}
	c:\Program Files\MongoDB\Server\3.0\bin>mongod.exe
2015-10-05T11:00:18.845+0530 I CONTROL
2015-10-05T11:00:18.846+0530 W CONTROL  32-bit servers don't have journaling enabled by default. Please use --journal if you want durability.
2015-10-05T11:00:18.846+0530 I CONTROL
2015-10-05T11:00:18.853+0530 W -        [initandlisten] Detected unclean shutdown - c:\data\db\mongod.lock is not empty.
2015-10-05T11:00:18.861+0530 I STORAGE  [initandlisten] **************Unclean shutdown detected.Please visit http://dochub.mongodb.org/core/repair for recovery instructions.*************
2015-10-05T11:00:18.863+0530 I STORAGE  [initandlisten] exception in initAndListen: 12596 old lock file, terminating
2015-10-05T11:00:18.863+0530 I CONTROL  [initandlisten] dbexit:  rc: 100
{% endhighlight %}

Hi friends,

After several hours, I opened up my mongoDB project to find out that the db was crashed. I was pretty scared because of the wise words said on stackoverflow:

> NoSQL is new and if any disaster occurs, your data is at the mercy of your developer

Oh boy! I started looking out the internet and finally found that I need to start the instance with repair option and specify the new database storage path:

	c:\Program Files\MongoDB\Server\3.0\bin>mongod.exe --dbpath=c:\data\db --repair --repairpath=c:\data\db0

where, db0 is new recovery location. If I don't want to preserve data, simply omit the repairpath and start