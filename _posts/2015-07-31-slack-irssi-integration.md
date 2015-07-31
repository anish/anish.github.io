---
layout: post
title: Connect to slack via irc/irssi
author: anish_bhatt
---

  > Note : Slack teams need to have the irc gateway enabled by the team admin for this to work.

This post provides a configuration example for irssi. Generic irc connection information for slack can be found at [this page](https://slack.zendesk.com/hc/en-us/articles/201727913-Connecting-to-Slack-over-IRC-and-XMPP).

First, obtain your irc gateway priveleges by going to <code>https://\<teamname>.slack.com/account/gateways</code>

 and fill out <code>\<user></code>, <code>\<host></code> and <code>\<pass></code> in the following commands.
{% highlight bash %}

/network add -nick <user> slack
/server add -auto -ssl -network slack <host> 6697 <pass>
/save
{% endhighlight %}
You only need the <code>-ssl</code> flag if your team admin has required ssl for irc connections. If no ssl is required, ports <code>6667</code> and <code>8000</code> can be used as well.

You can now connect to slack by typing

{% highlight bash %}
/connect slack

{% endhighlight %}

This will open up new windows for every channel in your team and you should be good to go.

As slack users become active or inactive, you'll see the following active/inactive messages in your channel.

{% highlight bash %}

11:02 -!- mode/#general [+v foo : active] by foo
11:03 -!- mode/#general [-v foo : away] by foo

{% endhighlight %}

You can hide these messages via

{% highlight bash %}

/ignore -network slack * MODES

{% endhighlight %}

