---
layout: post
title: LDAP integration with phabricator
author: anish_bhatt
---

***Where RTFM is no good***

  > This post tells you how to use connect phabricator to your LDAP server for auth credentials

When it comes to code review [Gerrit](https://code.google.com/p/gerrit/) is pretty much unbeatable. Sadly, since it's git only, and our internal codebase is hosted on Mercurial, this was of no use to us. 

Enter [phabricator](http://phabricator.org/). While the setup is pretty easy (even the manual installation on RHEL 5), the configuration is not as straightforward as I would like, with using LDAP integrtion for authentication not quite working as expected. Specifically our LDAP server did not support *anonymous* searches. While there are a few examples floating around the web, I couldn't really find one that worked. ldapsearch actually proved to be quite helpful here.  An easy way to try this out is to run the following command :
{% highlight bash %}
ldapsearch -h bar.foodomain.com -p 389 -x -b "CN=Users,dc=foodomain,dc=com" -D "foodomain\user" -W | less
{% endhighlight %}
Where <code>bar.foodomain.com</code> is your ldap server, and <code>user</code> is your ldap login name. The <code>-x</code> indicates use [simple authentication](http://docs.oracle.com/javase/jndi/tutorial/ldap/security/simple.html), and <code>-W</code> prompts for the password for 'user'. If this command worked for you, then the following configuration should work for LDAP in phabricator

{% highlight bash %}
* Hostname: bar.foodomain.com
* Port: 389
* Base DN:  dc=foodomain,dc=com
* Search Attribute: sAMAccountName
* Always Search : No
* Username Attributes: sAMAccountName
* Real Name attributes: displayName
* LDAP Version: 3
* Referrals: No
* Use TLS: No
* Active Directory Domain:  foodomain
{% endhighlight %}

A sample return for ldapsearch looks something similar to this (abbreviated for brevity) :
{% highlight bash %}
dn: CN=Bob Bobberson,CN=Users,DC=foodomain,DC=com
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Bob Bobberson
sn: Bobberson
description: Burger Wrangler
telephoneNumber: 311-555-2368
givenName: Bob
distinguishedName: CN=Bob Bobberson,CN=Users,DC=foodomain,DC=com
displayName: Bob Bobberson
mailNickname: burgerguy
name: Bob Bobberson
sAMAccountName: burgerguy
sAMAccountType: 705403368
userPrincipalName: burgerguy@foodomain.com
objectCategory: CN=Person,CN=Schema,CN=Config,DC=foodomain,DC=com
mail: burgerguy@fodomain.com
{% endhighlight %}

Username Attributes & Real name attributes were based on this sample return. The phabricator config actually recommends using <code>sn</code> instead of <code>sAMAccount</code>, this did not work for me. These values are not case sensitive. You can replace <code>displayName</code> with <code>givenName,sn</code> or pretty much any combination of returned attributes.

Note : a lot of places seem to recommend specifying Real Name attributes as <code>["givenName","sn"]</code> instead of <code>givenName,sn</code> but only the format without the brackets & quotes worked for me. You can also use the auth test utility shipped with phabricator to test ldap via <code>phabricator/bin/auth ldap</code>. It uses ldap settings set in phabricator > auth > ldap.

Simple LDAP configuration only allows you to login via ldap, you still need to *import* ldap users into phabricator for it to work correctly. To do this go to phabricator > people > import from ldap. Enter any useable username/password combo, and an ldap query to search for users to import. Based on the sample return above, a useable query would be <code>(objectClass=person)</code>. Note: if the root user uses an email address found in ldap search, you won't be able to add that particular user to phabricator, as phabricator will detect a duplicate entry of the email address.
