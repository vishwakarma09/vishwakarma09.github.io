---
title: How geeks get their food ordered :P
layout: post
---

# How geeks get their food ordered :P

Hi friends,

It was a everyday story that my mate and I used to miss ordering of food in slack bot. Company had developed a weird system to order meal through slack commands.

So, we came up with idea to create slack hook and make the slack bot say loud that I need to order food:
{% highlight php %}
root@ip-172-31-32-255:~# cat pallav.sh 
curl -X POST --data-urlencode 'payload={"channel": "@pallav.popli", "username": "Max", "text": "Order your meal Dear! :)", "icon_emoji": ":ghost:"}' https://hooks.slack.com/services/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
{% endhighlight %}

And it was added in system cron for twice! Ahahaha