---
layout: post
title: Fix crypto failures while compiling RHEL kernel
author: anish_bhatt
---

***Disable module signature checking***

If you get the following error while compiling a kernel (I seem to only hit it when compiling RHEL kernels)
{% highlight bash %}
crypto/signature/ksign-publickey.c:2:17: error: key.h: No such file or directory
crypto/signature/ksign-publickey.c: In function ‘ksign_init’:
crypto/signature/ksign-publickey.c:10: error: ‘ksign_def_public_key’ undeclared (first use in this function)
crypto/signature/ksign-publickey.c:10: error: (Each undeclared identifier is reported only once
crypto/signature/ksign-publickey.c:10: error: for each function it appears in.)
crypto/signature/ksign-publickey.c:11: error: ‘ksign_def_public_key_size’ undeclared (first use in this function)
make[2]: *** [crypto/signature/ksign-publickey.o] Error 1
make[1]: *** [crypto/signature] Error 2
make: *** [crypto] Error 2
{% endhighlight %}
You need to disable signature checking for modules on module load to get around this compilation problem. Just disable these two options in your kernel config
{% highlight bash %}
[*] Enable loadable module support --->
	[ ] Module signature verification
-*- Cryptographic API --->
	[ ] In-kernel signature checker
{% endhighlight %}
And you should be able to compile just fine
