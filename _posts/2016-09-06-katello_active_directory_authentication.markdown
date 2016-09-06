---
layout: post
title:  "Active Directory authentication in Katello/Foreman"
date:   2016-09-06 15:00:00 -07:00
categories: active-directory katello foreman
tags:
- Katello
- Foreman
- active-directory
---

To set Katello/Foreman up with active directory authentication through LDAP, there are two steps required:
1. Create LDAP Authentication server entry
2. Create User Group in Katello/Foreman interface that maps to external Group

Starting with 1:
~~~
On the LDAP server tab:
Name: <anything>
Server: <AD server address, we have a cname like so set up: ldap.domain.com)
LDAPS: <unchecked unless AD is set up this way>
Port: <default, 389>
Server type: Active directory

On Account tab:
Account username: <serviceaccount@domain.com>
Account password: <password>
Base DN: <DC=domain,DC=com>
Groups base DN: <OU=Groups,DC=domain,DC=com>
LDAP Filter: <not required>
Automatically create accounts in Foreman: <checked>
Usergroup sync: <checked>

On Attribute mappings tab:
Login name attribute: sAMAccountName
First name attribute: givenName
Surname attribute: sn
Email address attribute: mail
Photo attribute: <empty>
~~~

Step 2, create a new User Group
~~~
On User group tab:
Enter appropriate name

On Roles tab:
select appropriate roles

On External Groups tab:
Add external user group and enter AD group name
~~~

[Foreman: Web Interface and LDAP Authentication](https://theforeman.org/manuals/1.12/index.html#4.1WebInterface)
