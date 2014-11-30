---
layout: post
title: Windows Images for Openstack
tags: [How-to, Openstack, glance, Windows, cloud-init]
---

If you are starting to work with Openstack (like me) you should know now that unfortunately not all on it is the beautiful Linux world and maybe know you are going to realize that somethimes you also need to work with Windows systems, so I have done a little research about that topic and I can share my findings with you.

> The goal of this example will be to create a custom **Windows Server 2008 R2** cloud image supported by a Openstack cluster environment using a **QEMU/KVM** hypervisor, all of that based on a Red Hat Linux (RHEL) system.

<h2>Prerequisite</h2>
We will need the follow files to achieve the creation of this Windows cloud image.

* A Windows installer ISO image,  *that surprisingly for me we can download directly from the Microsoft Download Center site* [Windows Server 2008 R2 Evaluation ](http://www.microsoft.com/es-mx/download/details.aspx?id=11093) with a limited functionality (obviously) of 180 days but I guess it's okay for our training proposes.

* The latest version of [VirtIO drivers for Windows](http://alt.fedoraproject.org/pub/alt/virtio-win/latest/images/), which will allow a better performance for our customized Windows cloud image.

* The latest version of [Cloud-Init for Windows](http://www.cloudbase.it/cloud-init-for-windows-instances/) that the guys of [CloudBase](http://www.cloudbase.it) generously made and share with us.

-----

## Base image creation
We need to complete a basic Windows installation as first step to build our customized image.

### Create an empty raw image
First of all we have to create a image/disk in qcow2 format (which is the recommended for KVM hypervirors) will be used a container of our customized Windows image.

<pre># qemu-img create -f qcow2 -o IMAGE_NAME.qcow2 10G </pre>

<small>We recommend use 10G of space for the basic Windows Server 2008 installation but feel free to change that value for whatever you want.</small>

### Windows installation

Now we have to launch a KVM instance/VM and perform a basic Windows installation on it using the follow instructions.

<pre>
# /usr/libexec/qemu-kvm \
-m 2048 \
-cdrom Windows_Server_2008_R2_Eval.iso \
-drive file=Window-Server-2008-R2.qcow2,if=virtio \
-drive file=virtio-win-0.1-81.iso,index=3,media=cdrom \
-net nic,model=virtio \
-net user \
-nographic \
-usbdevice tablet \
-vnc :9</pre>

<small>Note we are referring to some VirtIO virtual devices on the last command sentence in order to our Windows image use them to get a better performance. Also we are using the parameter <code>-usbdevice tablet</code> in order to force to the mouse pointer to be more precise on the VNC client.</small>

Connect to the instance launched by VNC protocol
<pre># xvncviewer 127.0.0.1:9</pre>

<small>You can use the <code>xvncviewer</code> or any other VNC client that you want to use.</small>

Then we can start with the normal windows installation.

![Windows Installer]({{ baseurl }}/public/img/installer.png)

In the disk setup section we have to select the VirtIO driver from the ISO image attached on the second CDROM drive in order to move forward as follow:

<small>*Browse > CD Drive(E:) > WLH > AMD64 > Ok*</small>

![VirtIO Disk Driver]({{ baseurl }}/public/img/virtio_driver.gif)

Once finished with the installation you must complete the setup enabling the <code>Remote Desktop</code> access, enabling it and allowing the access in the firewall rules.

#### Complete the installation

Once you have completed the Windows base installation we must setup the missing VirtIO drivers, in this case we are going to update the network drivers as follow:

1. On the Device Manager
2. Select the missing ethernet device (Ethernet controller)
3. Update driver software...
4. Select browse my computer for driver software

   <small>Note: It's important to select this option because another it could select a wrong driver.</small>
5. Browse and select the cdrom with VirtIO drivers

   <small>*Browse > CDROM (E:) > <b>XP</b> > AMD64 > OK*</small>

   ![Network Driver]({{ baseurl }}/public/img/network_driver.png)

7. Next and Install the "**Red Hat VirtIO Ethernet Adapter**"
8. Done

#### Enable RDP

We need to enable the Remote Desktop in order to get access to the new instance from the network.

#### Setup firewall to allow access by RDP

Allow access by RDP enabling the follow firewall rule:

<small><code>Windows Firewall > Advanced Settings > Inbount Rules > Remote Desktop (TCP-in)</code></small>

> <small>***IMPORTANT NOTE:*** *I'm not a Windows Expert I'm just trying to show you the way that worked for me to complete the Windows image setup. So if you know a better method to complete these tasks please feel free to use them.*</small>

### Cloud-Init Setup


Now to complete our cloud image creation we need install the [cloud-init](http://#) packages in order to it has full functionality in our Openstack compute cloud.

There are several ways to attach the necessary files to our instance, but as I didn't have Internet access and either have the necessary packages to mount a external filesystem into it, I choised the easier way for me to share those files to it [making a ISO image](http://www.linuxlookup.com/howto/create_iso_image_file_linux) and mount it as cdrom to get access to the **cloud-init installer for Windows**. After that we just need to follow the wizard (in Windows style).

*On this time we have completed the creation of our own Windows Cloud Image!!*.

-----

## Import the image to Openstack

Now we are ready to import the new image to our Openstack environment and test it.

<pre># glance add name="Windows-2k8-x64" is_public=true container_format=bare disk_format=qcow2 < Window-Server-2008-R2.qcow2 </pre>

#### Launch an instance based on the new image for first time

From the dashboard launch a new instance using our new Windows image.

#### Get the password

We need to run the follow command in order to get the credentials to access to the new instance by RDP using the <code>key pair</code> used during its creation.

<pre># nova get-password INSTANCE_NAME KEY_NAME.key </pre>

#### Connect to the Windows instance

In this point you should be able to connect to the new instance through your favourite RDP client pointing to the IP address assigned to it.

So, enjoy it!!

-----

### References
* [Windows Images for OpenStack](http://www.florentflament.com/blog/windows-images-for-openstack.html)
* [Building a Windows Image for Openstack](http://networkstatic.net/building-a-windows-image-for-openstack/)
* [Windows 2008 Server R2 install guide for KVM and OpenStack](https://unicornclouds.telegr.am/blog_posts/kvm_windows_server_2008_r2_install_openstack)
