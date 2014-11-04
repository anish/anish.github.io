---
layout: post
title: Copying large blocks of text between different vim instances
author: anish_bhatt
---

Copying large blocks of text within vim instances can be messy, and often mess up formatting while pasting text in another vim instance. This post explains how to use a temporary buffer to store blocks of text you want to copy/paste into local or remote instances. This is fairly straightforward as long as you are a little comfortable with vim.

Enter visual mode (esc + v) and select the text you wish to copy. Yank your text (y) and then run the following commands to open a temporary buffer, paste contents into buffer, and close buffer and return to previous buffer and release the *.swp file
{% highlight bash %}
:e /tmp/dummy
paste the text to the new buffer (p)
:w
:bp
{% endhighlight %}
Here, we are using /tmp/dummy as our temporary buffer. Now copy your dummy file to your remote location, or skip this step if you're copying between local instances. Make sure you use the correct path if you rename or move the temporary buffer file. Pasting text is as simple as going to normal mode and reading in the file
{% highlight bash %}
:r /tmp/dummy
{% endhighlight %}
And your copying is done.

Originally seen on the vim wiki [here](http://vim.wikia.com/wiki/Copy,_cut_and_paste)
