---
layout: post
title: Replying to a thread on lkml via git send-email
author: anish_bhatt
---

***Where cli proves to be the best mail client***

  > This post tells you how to use git send-email's in-reply-to function to create a threaded reply

I recently had to reply to a thread on the [linux kernel mailing list](https://lkml.org/) that happened to be posted when I was not subscribed to it. While clients like Thunderbird/Outlook or even Gmail may be smart enough to string conversations together via the subject line or some variation thereof (eg, foo & Re: foo), mailing lists rely on message headers, specifically [*Message-Id* & *In-Reply-to*](http://cr.yp.to/immhf/thread.html).

What this basically means is that if you are sending message B as a reply to message A (thus creating a thread), the value of In-Reply-To field in the message headers of B should be the same as the Message-Id in the message headers of A. Similarly, message C will include the Message-Id of B in the In-Reply-To field when sent as a reply to B. Take a look [here](http://lkml.iu.edu//hypermail/linux/kernel/1406.2/index.html) for an example of a threaded view.

Now lkml has a [fairly strict](https://www.kernel.org/doc/Documentation/email-clients.txt) set of guidelines about how to format email sent to lkml, but it basically boils down no html, text only, no word wrapping and inline patches. If these do not apply to your mailing list, you might want to directly try [configuring Thunderbird](http://www.pixelbeat.org/docs/thunderbird-threading.html) to support In-Reply-To and skip the rest of this. Note: this did not seem to work on Thunderbird 24. Pretty much none of the email clients I tried (mutt, realpine) seemed to have an easy way to support in-reply-to

***If you're still here stranger, time to get excited about cli !***

To get past all these mailing constraints, and because Linus basically wrote Git, git supports the ability to directly send out email that passes all these requirements. Note, git send-email needs perl-MIME-Base64 and perl-Authen-SASL installed. There is also a bug in git send-email(v2.0.0.) that forces encryption even if your smtp server does not require it. Please (temporarily) apply the following patch if your outgoing/relay server does not use encryption
{% highlight diff %}
--- /usr/lib/git-core/git-send-email.orig       2014-06-06 12:26:00.982762503 -0700
+++ /usr/lib/git-core/git-send-email    2014-06-02 18:36:12.857747720 -0700
@@ -1056,7 +1056,7 @@ sub smtp_host_string {
 # (smtp_user was not specified), and 0 otherwise.

 sub smtp_auth_maybe {
-       if (!defined $smtp_authuser || $auth) {
+       if (defined $smtp_authuser || $auth) {
                return 1;
        }

{% endhighlight %}

Now, lets takes a sample message to reply to. Let's presume we want to reply to [Linus Torvalds](https://lkml.org/lkml/2014/6/16/1) (Please do not actually email Linus, you will be ignored and hated). First thing we want to do is fetch the headers for that message, which we can see by clicking [[headers]](https://lkml.org/lkml/headers/2014/6/16/1). Specifically, we only need one line
{% highlight bash %}
Message-Id	<CA+55aFzNDTm_O8rGYdNw1S99P06u6EeSdttXBvtURJocQT2O0g@mail.gmail.com>
{% endhighlight %}
The value inside <...> is the Message-Id we need to reply to create a thread around this message. Next, we create a text file, let's say msg1 that git can parse as an email draft. You can type out your email here if you like but you really need to have only two lines in there, message body goes below these two lines
{% highlight bash %}
From: Bob Bobberson <bob@bobberson.com>
Subject: Re: Bob Bobbersons Burgers is now open
{% endhighlight %}
Change your email address & subject as required, the 'Re: ' before the subject line is recommended to indicate a reply. Next, run git send-email as follows
{% highlight bash %}
git send-email --smtp-server smtp.bobberson.com --smtp-user bob --to torvalds@linux-foundation.org --cc linux-kernel@vger.kernel.org  --in-reply-to CA+55aFzNDTm_O8rGYdNw1S99P06u6EeSdttXBvtURJocQT2O0g@mail.gmail.com --annotate <path_to_msg1>
{% endhighlight %}
To set encryption, use --smtp-encryption ssl or tls. Using any other value disables encryption (but take note of previously mentioned bug). Using the --annotate flag lets you edit the email before it is sent out, simply type out your message below the subject line. If you skip --annotate, git will proceed to send email (but asks for confirmation).

Note, there are some caveats here. Some places like the [Indiana University Kernel Mailing List Archives](http://lkml.iu.edu/hypermail/) group messages into chunks of 15 days, you won't be able to see a threaded view unless you reply before that 15 day window expires, but you will still see your reply threaded at places without this restriction, for eg the actual [lkml archives](https://lkml.org)
