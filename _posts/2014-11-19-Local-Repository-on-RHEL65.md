---
layout: post
title: Create a local repository (with group options) on RHEL 6.5
tags: [Linux, RHEL, How-to]
---

This is an useful mini guide to create a RHEL local repository with package group support, the idea is to create a local repository in order to share it with other host on our local network principally for study cases or when we haven't a **Red Hat** subscription or simply when we haven't Internet access.

In addition to create a classic local repository we are going to setup the package group support which usually be a good tool to manage entire group of required packages for an specific propose or to cover a requirement.

> i.e. Install the Desktop environment or X support for a system where they weren't installed.

-----

<h2>Pre-requisites </h2>

Mount CD/DVD media disk

<pre># mount -o loop /dev/cdrom /mnt</pre>

Create your repository layout

<pre> # mkdir -p /var/repo/rhel/6/u5/x86_64 </pre>

Copy the <code>rpm</code> files and go to the new packages directory

<pre> # cp -a /mnt/Packages/*.rpm /var/repo/rhel/6/u5/x86_64/ </pre>

<pre> # cd /var/repo/rhel/6/u5/x86_64</pre>

Install the necessary packages

<pre> # rpm -iUvh deltarpm-* python-deltarpm-* createrepo-* </pre>
------

<h2>Create the repository </h2>

Now we are ready to create our new local repository.

<pre> # createrepo . </pre>


**Enable group support**

<pre> # cp /mnt/repodata/*comps*.xml repodata/comps.xml </pre>

<pre># create -g repodata/comps.xml . </pre>

**Setup your local repo file**

Create a file called <code>/etc/yum.repo.d/rhel65-local.repo</code> and add the follow lines and save it:

<pre>
 [rhel65_x64-local]
 name=RHEL 6.5 local repository
 baseurl=file:///var/repo/rhel/6/u5/x86_64
 enabled=1
 gpgcheck=0
</pre>

Clean and recreate the yum db

<pre> # yum clean all </pre>
<pre> # yum makecache </pre>

Test it

<pre> # yum list </pre>
<pre> # yum grouplist </pre>
<pre> # yum groupinstall "X Window System" Desktop -y </pre>


Enjoy it!! :)

**Questions???**
*Google is your friend! ;)*
