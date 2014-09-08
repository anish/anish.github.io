---
layout: post
title: Migrate from RHEL6 to CentOS6 repos
author: anish_bhatt
---

***Damn that RHN subscription***

  > Note : The rpm versions keep changing with every package update and might be out of date. You can check the correct current versions by going [here](http://mirror.centos.org/centos/6/os/x86_64/Packages)

While the instructions on the [CentOS wiki](http://wiki.centos.org/HowTos/MigrationGuide) are pretty good, they seem to be lacking removal of redhat-release-server, which is required to actually move to RHEL to CentOS repos. This is an updated version of that page, with slightly fewer commands.
{% highlight bash %}

yum remove rhnlib abrt-plugin-bugzilla redhat-release-notes*
rpm -e --nodeps redhat-release redhat-release-server redhat-indexhtml
yum install http://mirror.centos.org/centos/6/os/x86_64/Packages/centos-release-6-5.el6.centos.11.1.x86_64.rpm
yum install http://mirror.centos.org/centos/6/os/x86_64/Packages/centos-indexhtml-6-1.el6.centos.noarch.rpm
yum install http://mirror.centos.org/centos/6/os/x86_64/Packages/yum-3.2.29-40.el6.centos.noarch.rpm
yum install http://mirror.centos.org/centos/6/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.30-14.el6.noarch.rpm
yum clean all
yum upgrade

{% endhighlight %}
You should have moved completely from RHEL packages to CentOS packages at the end of this step. Please note that these instructions are for x86\_64, you need to replace 'x86\_64' with 'i386' in the urls if you are on a 32-bit system.
