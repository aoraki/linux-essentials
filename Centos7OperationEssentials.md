# CentOS Enterprise Linux 7 Operation Essentials - Course Notes

Path to pluralsight course : https://app.pluralsight.com/library/courses/lfcs-linux-operation-essentials/table-of-contents

## Introduction

### Reading operating system data
To get information relating to your version of CentOS
```
[centos@server1 ~]$ cat /etc/system-release
CentOS Linux release 7.9.2009 (Core)
```

Using `lsb_release`, it might need to be installed if it's not already installed
```
[centos@server1 ~]$ lsb_release -d
Description:	CentOS Linux release 7.9.2009 (Core)
```

RPM query file command to find out what package a file belongs to
```
[centos@server1 ~]$ rpm -qf $(which lsb_release)
redhat-lsb-core-4.1-27.el7.centos.1.x86_64
```

To find out the version of kernel being run
```
[centos@server1 ~]$ uname -r
3.10.0-1160.15.2.el7.x86_64
```
To get more indepth version info
```
[centos@server1 ~]$ cat /proc/version
Linux version 3.10.0-1160.15.2.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) ) #1 SMP Wed Feb 3 15:06:38 UTC 2021
```
During the boot process we pass command line arguments through to the kernel
To find out what those arguments are;
```
[centos@server1 ~]$ cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-3.10.0-1160.15.2.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=en_IE.UTF-8
```

To see the various partitions including the boot partition use lsblk //
```
[centos@server1 ~]$ lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0    8G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0    7G  0 part
  ├─centos-root 253:0    0  6.2G  0 lvm  /
  └─centos-swap 253:1    0  820M  0 lvm  [SWAP]
sr0              11:0    1 1024M  0 rom
```

## Starting and Stopping CentOS 7

### Messaging and using wall

Messaging a specific user (`centos` in this case)
```
write centos
```

The above starts a write session and you can start writing a message.  When you hit
enter it sends the line to the recipient.  You can type more lines and hit enter
each time, and each line will be sent.  The messages will come through in real time.
To exit the write session, `CTRL-D`. An EOF entry will appear at the end of the message
on the recipients screen, and they just need to hit enter to clear it.

To send a message to everyone, first create a message using a `HERE` document.
The message is completed when you type `END`.

```
[root@server1 ~]# cat > message <<END
> The server is coming down at 5pm.
> Please be sure everything is done by then.  Thanks
> END
```

Then you post the message to wall using the following;
```
[root@server1 ~]# wall < message
[root@server1 ~]#
Broadcast message from root@server1.example.com (Thu Feb 25 11:37:22 2021):

The server is coming down at 5pm.
Please be sure everything is done by then.  Thanks
```

If we are sending this as `root` (whether we use wall or write), it doesn't matter
if the recipients have messaging enabled or disabled - they will get the message

To see if messaging is enabled for a user
```
[centos@server1 ~]$ mesg
is y
```

To disable or enable messaging for the current user
```
[centos@server1 ~]$ mesg n
[centos@server1 ~]$ mesg
is n
[centos@server1 ~]$ mesg y
[centos@server1 ~]$ mesg
is y
```

### Shutdown Commands
* `reboot` - Restarts the machine
* `poweroff` - Instructs the system to power down
* `halt` - Like a sleep mode, instructs the hardware to stop all CPU functions

Some legacy commands can be used to perform the same as above.  The issue with
these commands is that they execute immediately, and the user has no real control
once the command has been issued;
* `init` - this has options to poweroff, reboot, enter rescue mode, etc.
* `telinit` - pretty much the same as init

To issue a shutdown command. This sends out a message to all users via `wall`
```
root@server1 ~]# shutdown -h 1 "The system is closing in 1 minute"
Shutdown scheduled for Thu 2021-02-25 11:53:37 GMT, use 'shutdown -c' to cancel.
[root@server1 ~]#
Broadcast message from root@server1.example.com (Thu 2021-02-25 11:52:37 GMT):

The system is closing in 1 minute
The system is going down for power-off at Thu 2021-02-25 11:53:37 GMT!
```
The above message will be sent out every minute until the shutdown.

One mechanism that exists in linux, is that once the shutdown counter gets under
the 5 minute mark, the `/run/nologin` file is created.  This prevents any user
other than root from logging in and cancelling the shutdown.

To cancel a shutdown command (This will also delete any /run/nologin file, if one exists)
```
shutdown -c
```

### Changing runlevels

These commands tells us what run level we're at.  run-level 5 tells us it's a graphical environment.
This is a legacy term, it's now known as the "graphical target"
```
[root@server1 ~]# who -r
         run-level 5  2021-02-24 20:56

[root@server1 ~]# runlevel
N 5
```

To get the default boot-up environment
```
[root@server1 ~]# systemctl get-default
graphical.target
```

To set a new default target use the following;
```
[root@server1 ~]# systemctl set-default multi-user.target
Removed symlink /etc/systemd/system/default.target.
Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/multi-user.target.
```
Multi-user is the old run-level 3, thats the same as the graphical environment,
but without the graphics.  It allows multiple users to be logged on over the network but we don't
start up the display manager service

To change the run levels in the current session
```
[root@server1 ~]# systemctl isolate multi-user.target
```

To see the current run level
```
[root@server1 ~]# who -r
         run-level 3  2021-02-25 21:06                   last=5

[root@server1 ~]# runlevel
N 3
```

Rescue mode (single user mode) provides the ability to boot a small linux environment entirely
from a disk, or some other boot method, instead of the systems hard drive.
With this target the network services are not run so any remote connection is lost
```
[root@server1 ~]# systemctl isolate rescue.target
```

#### Changing the run level at boot
When the machine is starting up it will display the `GRUB2` menu with a list of kernels
that you can boot to.  Hit the `esc` key to halt the process.  The default entry is
the one that is selected.

Then hit `e` to edit the default entry.  You have to scroll down to the line that starts
with linux16 which is where we're going to boot from the kernel. Hit `CTRL-E` to navigate to the
end of that line.  Add `systemd.unit=rescue.target` to the end of the line.  Then hit `CTRL-x`
to continue booting.  The machine will boot to rescue mode, which is command line only. You
need to enter the root password.  Once logged in run the `runlevel` or `who -r`
command to see what run level we're at.

## The Boot Process

// Enable Recovery Mode //
When you reboot a machine the GRUB2 menu gets displayed.  It shows the kernels
that are installed including a "rescue" kernel.  This is a fallback kernel, so if
your other kernels get corrupted, at least you have a way to boot up the system.
This is not to be confused with the rescue.target, it's just a rescue image that we
can boot to.
