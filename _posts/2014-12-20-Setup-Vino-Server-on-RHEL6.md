---
layout: post
title: How to setup Vino on RHEL6 (Quick Guide)
tags: [How-to, Linux, Tools]
---

Vino is a little VNC server that allow you to share your desktop session with other users, it usually comes installed by default in almost the Linux distribution  but in this case we will focused in how to setup it on a RHEL 6.5 system.

> ***Important Note:*** *Vino doesn't work as dedicate VNC server, it mean it isn't listening all the time for new connections, we need to have opened a session in order to use it to share our desktop with other users*.


### Install and setup the X environment.

If you are trying to run VINO in a without X environment installed, the first step should be install and enable it as follow, otherwise if you have already your system working with the graphical environment please skip to [Vino Setup](#vinosetup) section.

You should follow these steps if you are going to enable the graphical environment.

<pre># yum install -y "X Window Manager" Desktop</pre>

Edit the file <code>/etc/inittab</code> and change the values as follow
<pre>
--
id:5:initdefault:
--
</pre>

At the end you are going to have to restart the system in order to it take the changes and start with the graphical environment.
<pre># reboot</pre>

Complete the first boot wizard

### <a name="vinosetup"></a>Vino Setup

You must follow the next instructions in order to enable Vino in your session: <code>Menu > Preferences > Administration > Remote Desktop </code> and setup it like this:
<pre>
+ Allow other users to view your desktop
+ Allow other users to control your desktop
- You must confirm each access to this machine
+ Require that user enter this password *****
</pre>

<small>Where "+" means enable and "-" disable.</small>

#### Setup firewall to accept VNC connections
<pre>
 # iptables -A INPUT -p tcp -m tcp --dport 5900 -j ACCEPT

 # service iptables save

 # service iptables restart
</pre>

#### Test connections
<pre># vncviewer  SERVER_ADDRESS </pre>

***Questions??*** Google is your friend! ;)
