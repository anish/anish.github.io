---
layout: post
title: Identify text crossing the 80 char limit in vim
author: anish_bhatt
---
***First we assume a spherical cow with a vt100***

If you use the [Kernel coding style](https://www.kernel.org/doc/Documentation/CodingStyle), one of the more taxing issues is staying within the 80 char limit. Instead of using something like [checkpatch.pl](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/scripts/checkpatch.pl) to identify code failing this requirement after creating patches, you can use vim's inbuilt long line highlight options to identify offending lines in code. The following snippet will highlight all any text that crosses virtual column 80
{% highlight bash %}
:match ErrorMsg '\%>80v.\+'
{% endhighlight %}
If you prefer something less in your face, the following snippet will only highlight characters at column 81
{% highlight bash %}
:2mat ErrorMsg '\%81v.'
{% endhighlight %}
You can also directly add these to your .vimrc file, removing the colon the at the start. More details can be found at the [vim help page](http://vim.wikia.com/wiki/Highlight_long_lines)
