---
layout: post
title: Multi Node devstack setup
author: anish_bhatt
---

  > Note : A detailed guide to a Multi-Node devstack setup can be found in [openstack documentation](http://http://docs.openstack.org/developer/devstack/guides/multinode-lab.html), this guide simply elaborates on the node configuration.

This guide covers setting up a separate controller, cinder and compute node via devstack. Configuration of each node is addressed individually.

**Controller Node**

The controller node must be brought up first as the cinder/compute node rely on it for services. You would bring up devstack regularly using the following configuration :

````bash
# Controller configuration
[[local|localrc]]

SERVICE_TOKEN=azertytoken
ADMIN_PASSWORD=foobar
MYSQL_PASSWORD=foobar
SERVICE_PASSWORD=foobar
DATABASE_PASSWORD=foobar
RABBIT_PASSWORD=foobar

HOST_IP=10.192.194.1
PUBLIC_INTERFACE=ens0

MULTI_HOST=True

disable_service c-vol
disable_service n-cpu
disable_service n-net
````
We disable <code>c-vol</code>, <code>n-cpu</code> & <code>n-net</code> because we do not want to run the cinder volume, nova compute or nova network on the controller node, as we have separate nodes for this. Setting <code>MULTI_HOST=True</code> is required for a multi node setup.

While the cinder and compute node can be brought up simulatenously, ensure that the controller node is completely up and running before bringing up the other nodes.

**Cinder Node**

Next we bring up the cinder node. The configuration here is slightly more complicated.

````bash
# Cinder configuration
[[local|localrc]]

SERVICE_TOKEN=azertytoken
ADMIN_PASSWORD=foobar
MYSQL_PASSWORD=foobar
SERVICE_PASSWORD=foobar
DATABASE_PASSWORD=foobar
RABBIT_PASSWORD=foobar

SERVICE_HOST=10.192.194.1 # controller ip
HOST_IP=10.192.194.2
PUBLIC_INTERFACE=ens0

MULTI_HOST=True

MYSQL_HOST=$SERVICE_HOST
RABBIT_HOST=$SERVICE_HOST
CINDER_SERVICE_HOST=$SERVICE_HOST
GLANCE_HOSTPORT=$SERVICE_HOST:9292
DATABASE_TYPE=mysql

ENABLED_SERVICES=c-sch,c-vol

[[post-config|$CINDER_CONF]]
[DEFAULT]
my_ip=$HOST_IP
CINDER_ENABLED_BACKENDS=lvm:lvmdriver-1

[lvmdriver-1]
VOLUME_GROUP="stack-volumes"
VOLUME_NAME_PREFIX="volume-"
````

We only enable the required services, <code>c-sch</code> and <code>c-vol</code> here as this will be a cinder only node. Note the difference in format while disabling individual services and enabling an array.

We also add a post config section to cinder to make some changes required for a multi node setup. The <code>my\_ip</code> value, used to indicate iscsi target address defaults to <code>SERVICE\_HOST</code> which normally indicates the devstack host ip. For a multi-node setup we need to change it to <code>HOST\_IP</code>. We also declare an lvm backend with a backend name (lvmdriver-1 in this case). When the cinder service with an lvm backend is started, it creates a new volume group named in the format <code>\<VOLUME\_GROUP\>-\<backend_name\></code>. By specifying the <code>VOLUME\_GROUP</code> under the named lvm backend configuration section, we can control the name to be <code>stack-volumes-lvmdriver-1</code>. By creating this volume group before devstack is brought up, when can make devstack use our preconfigured VG. 

**Compute Node**

For the compute node, the configuration is much simpler.

````bash
# Compute configuration
[[local|localrc]]

SERVICE_TOKEN=azertytoken
ADMIN_PASSWORD=foobar
MYSQL_PASSWORD=foobar
SERVICE_PASSWORD=foobar
DATABASE_PASSWORD=foobar
RABBIT_PASSWORD=foobar

SERVICE_HOST=10.192.194.1 # controller ip
HOST_IP=10.192.194.3
PUBLIC_INTERFACE=ens0

MULTI_HOST=True

MYSQL_HOST=$SERVICE_HOST
RABBIT_HOST=$SERVICE_HOST
CINDER_SERVICE_HOST=$SERVICE_HOST
GLANCE_HOSTPORT=$SERVICE_HOST:9292
DATABASE_TYPE=mysql

ENABLED_SERVICES=n-cpu,n-net
````

For the compute node, we similarly enable only the required services. Once you have all three nodes up and running, you should have a fully working multi-node setup deployed. Happy stacking !

 > Note : devstack code changes quite rapidly and might be sometimes unstable. This instructions were verified working at the time of the Kilo Release, but YMMV.
