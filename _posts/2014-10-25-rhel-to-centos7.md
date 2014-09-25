---
layout: post
title: Migrate from RHEL7 to CentOS7 repos
author: anish_bhatt
---

***Damn that RHN subscription***

  > Note : The rpm versions keep changing with every package update and might be out of date. You can check the correct current versions by going [here](http://mirror.centos.org/centos/7/os/x86_64/Packages)

{% highlight bash %}
# Remove the RHEL7 release packages
yum remove rhnlib redhat-support-tool redhat-support-lib-python
rpm -qa| egrep "rhn|redhat" | xargs rpm -e --nodeps

# These create an issue later, so remove these directories now
rm -rf /usr/share/redhat-release/
rm -rf /usr/share/doc/redhat-release/

# Import the CentOS 7 GPG key
rpm --import http://mirror.centos.org/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7

# Install the CentOS 7 release packages
yum install http://mirror.centos.org/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-24.el7.noarch.rpm
yum install http://mirror.centos.org/centos/7/os/x86_64/Packages/yum-3.4.3-118.el7.centos.noarch.rpm
yum install http://mirror.centos.org/centos/7/os/x86_64/Packages/centos-logos-70.0.6-1.el7.centos.noarch.rpm
yum install http://mirror.centos.org/centos/7/os/x86_64/Packages/centos-release-7-0.1406.el7.centos.2.3.x86_64.rpm

# Clean database & upgrade all packages
yum clean all
yum install deltarpm yum-presto
yum upgrade

# Change GRUB2 entries
grub2-mkconfig -o /boot/grub2/grub.cfg
{% endhighlight %}
You should have moved completely from RHEL packages to CentOS packages at the end of this step. Please note that these instructions are for x86\_64, you need to replace 'x86\_64' with 'i386' in the urls if you are on a 32-bit system.
