# Steps to set up a new Graphical Interfaced Centos 7 Server on Virtualbox

## Download the Centos7 ISO File

Pick the minimal one, we will install the rest of the required packages as we
go along;

http://mirror.strencom.net/centos/7.9.2009/isos/x86_64/

## Virtual Box Set-up

1) Create a new Server instance on VirtualBox.

* Name : server1
* Type : Linux
* Version : Red Hat (64 bit)

2) Select Memory Size of 1024 Ram

3) Select "Create a virtual hard disk now" option, 8GB is fine

4) Select VDI Hard Disk file type.  Accept the suggested File Location and Size.  Click "Create", this will create a server in your virtual box.

5) Select your new server and click on Storage and Optical Drive.  Select the CentOS-7-x86_64-Minimal-2009.iso you downloaded earlier.

6) Click on Network.  For Adapter 1, ensure "Enable Network Adapter" is selected and that it is attached to "NAT Network".  For Adapter 3, ensure "Enable Network Adapter" is selected and that it is attached to "Host-only Adapter".  The name should be something like vboxnet0.

7) Start the server and select "Install CentOS 7" in the boot up menu.

8) In the installation select and language and region and click continue. In the next screen, go into "Installation Destination" and click "Done" to confirm the setting.

9) In "Network and Host Name" enter your hostname, server1.example.com, and ensure both
network adapters are enabled, and click done.

10) Click "Begin Installation" to start the actual installation.  When the installation is running,
set the root password, and then create an additional user.  The username should be "centos", and ensure
you make the user an administrator.

## Configure networking on server
1) Log in as root.  Check the ip address settings of the server

```
[root@master1 ~]# ip a s
```

To show the IP addresses associated with the server.  Device no. 3 will not have
an IP address, this means that one of the network adapters is not up and running.


We need to do is ensure both network adapters are
up and running.  Run;

```
[root@master ~]# nmcli conn show
NAME    UUID                                  TYPE      DEVICE
enp0s3  e596a2c1-f21f-498d-b090-2daf612aa967  ethernet  enp0s3
enp0s8  993075f3-e472-4b5e-92a3-fe3b2b0f2f70  ethernet  --
```

One of the adapters will not have it's device name populated, this means that it's
not up and running.  To enable it run the following;

```
[root@master1 ~]# nmcli conn up enp0s8.
```

Check the connection status again, and you will see both adapters have a device
name associated with them;

```
[root@master ~]# nmcli conn show
NAME    UUID                                  TYPE      DEVICE
enp0s3  e596a2c1-f21f-498d-b090-2daf612aa967  ethernet  enp0s3
enp0s8  993075f3-e472-4b5e-92a3-fe3b2b0f2f70  ethernet  enp0s8
```
If you run;
```
[root@master1 ~]# ip a s
```
You will see that your server now has an IP address.  Networking is now configured.

Finally, make sure your network adapter settings are reboot persistent, run the following commands;

```
[root@master1 ~]# sed -i s/ONBOOT=no/ONBOOT=yes/ /etc/sysconfig/network-scripts/ifcfg-enp0s3
[root@master1 ~]# sed -i s/ONBOOT=no/ONBOOT=yes/ /etc/sysconfig/network-scripts/ifcfg-enp0s8
```
## Install the necessary software on your new machine to make it a GUI based server

1) First, perform a general update on all packages;

```
[root@master1 ~]# yum update
```
2) Next, install some tools we'll be needing
```
[root@master1 ~]# yum install -y redhat-lsb-core net-tools epel-release kernel-headers kernel-devel
```
3) Install the graphical packages
```
[root@master1 ~]# yum groupinstall -y "X Window System" "MATE Desktop"
```
4) Finally, set the server to use the graphical target as the default and then call the isolate command to switch to it.

```
[root@master1 ~]# systemctl set-default graphical.target

[root@master1 ~]# systemctl isolate graphical.target
```
