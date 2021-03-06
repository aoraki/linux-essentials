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

### Changing runlevels

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

#### Changing the run level at boot
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

### Enable Recovery Mode
When you reboot a machine the GRUB2 menu gets displayed.  It shows the kernels
that are installed including a `rescue` kernel.  This is a fallback kernel, so if
your other kernels get corrupted, at least you have a way to boot up the system.
This is not to be confused with the rescue.target, it's just a rescue image that we
can boot to.

You can configure GRUB so that it displays a recovery mode kernel for each of your normal
kernels.  You can do this by editing the grub file;
```
vi /etc/default/grub
```
Change the value of `GRUB_DISABLE_RECOVERY` to `false`

Then we need to remake the GRUB2 config;
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```
When that command has completed, reboot the machine by typing the command `reboot`.
When you get to the GRUB2 menu again, you will see that for each kernel you had previously
there will be an additional recovery mode kernel.  If you select the recovery mode of the
latest kernel, when you log in to that kernel and run `runlevel` you will see that the runlevel
is 1, which means it's in recovery mode.

### Reset Lost/Forgotten Root Passwords
To do this we need to interrupt the boot process, like we did previously.  When the GRUB2 menu
is displayed, hit esc or the up or down arrows.  Then hit `e` to edit the default entry.  You
have to scroll down to the line that starts with linux16 which is where we're going to boot
from the kernel. Hit `CTRL-E` to navigate to the end of that line.  Remove the words `quiet`
and `rhgb` from the end of the line, and add `rd.break` and `enforcing=0`.  The last option is
to switch off SELinux checks, this process would fail otherwise.  Hit `CTLR-x` to
continue the boot process  This will only going to boot as far as completing the RAM disk phase.

The `switch_root` prompt will then be displayed. This is where we are about to switch to the root file
system.  We need to remount the root file system.  Currently the file system is mounted to read only
but we need to set it to read-write so that we can reset the root password
```
switch_root:/# mount -o remount,rw /sysroot
switch_root:/# chroot /sysroot
```

Now we are on the real root file system that was previously mounted to sysroot.  We can reset the password
using the following command
```
sh-4.2# passwd
```
You should see a message `passwd: all authentication tokens updated successfully`

We now need to set everything back, exit out of the chroot'ed environment
```
sh-4.2# exit
```
Remount the root file system to sysroot as read only and exit.  Once exited the boot
process will continue
```
switch_root:/# mount -o remount,ro /sysroot
switch_root:/# exit
```

You should be able to log in as root with the new password.  But we still need to restore the security context
for the password file.  Changing it outside of SELinux has resulted in it losing some settings.
```
restorecon /etc/shadow

```
Finally, SELinux is still in permissive state which we can revert to it's normal state as follows
```
setenforce 1
```

## Managing GRUB2 Bootloader
`GRUB` - **G**rand **U**nified **B**ootloader.  The version used in Centos7 is `GRUB2`

### Re-installing GRUB2 (as root user)
On BIOS based machines we normally install the GRUB bootloader through to the master
boot record on the bootable device
```
grub2-install /dev/sda
```

In non-BIOS based (UEFI-based) systems the command to reinstall GRUB is as follows
```
yum reinstall grub2-efi shim
```

### Manage GRUB2 Defaults
`cat /etc/default/grub`

This file contains the default settings for GRUB. We edited this file earlier and we can also
tweak other settings such as GRUB_TIMEOUT for example.

The main GRUB file gets it's information from the default file above but also from a custom
directory.

To rebuild your config after changes to the default GRUB file
```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### Manage GRUB2 with grubby
If the GRUB config changes are complex or messy, we can use a utility called `grubby` to help
us.

To determine our default kernel in GRUB (the one that is selected by default when the GRUB menu is shown)
```
[root@server1 ~]# grubby --default-kernel
/boot/vmlinuz-3.10.0-1160.15.2.el7.x86_64
```

To set a different default kernel on GRUB menu
```
[root@server1 ~]# grubby --set-default /boot/vmlinuz-3.10.0-1160.el7.x86_64
[root@server1 ~]# grubby --default-kernel
/boot/vmlinuz-3.10.0-1160.el7.x86_64
```

To see Grubby settings for all our various kernels
```
[root@server1 ~]# grubby --info ALL
index=0
kernel=/boot/vmlinuz-3.10.0-1160.15.2.el7.x86_64
args="ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
root=/dev/mapper/centos-root
initrd=/boot/initramfs-3.10.0-1160.15.2.el7.x86_64.img
title=CentOS Linux (3.10.0-1160.15.2.el7.x86_64) 7 (Core)
index=1
kernel=/boot/vmlinuz-3.10.0-1160.el7.x86_64
args="ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
root=/dev/mapper/centos-root
initrd=/boot/initramfs-3.10.0-1160.el7.x86_64.img
title=CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
index=2
kernel=/boot/vmlinuz-0-rescue-66213c250ded4576a9120b120ad72e6c
args="ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
root=/dev/mapper/centos-root
initrd=/boot/initramfs-0-rescue-66213c250ded4576a9120b120ad72e6c.img
title=CentOS Linux (0-rescue-66213c250ded4576a9120b120ad72e6c) 7 (Core)
index=3
non linux entry
```

To see GRUB config for a specific kernel
```
[root@server1 ~]# grubby --info /boot/vmlinuz-3.10.0-1160.el7.x86_64
index=1
kernel=/boot/vmlinuz-3.10.0-1160.el7.x86_64
args="ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
root=/dev/mapper/centos-root
initrd=/boot/initramfs-3.10.0-1160.el7.x86_64.img
title=CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
```

We can change the args for a kernel using the following, such as remove rhgb and quiet
```
grubby --remove-args="rhgb quiet" --update-kernel /boot/vmlinuz-3.10.0-1160.el7.x86_64
```

### Password Protecting GRUB2
We can set passwords for our GRUB menu.  Passwords can either be encrypted or unencrypted

#### Setting a plain text password
Back up the existing GRUB users file
```
cp /etc/grub.d/01_users .
```

Then edit the users file and replace the contents the following.  The superusers setting here does not relate to anything in the linux users setup, it can be something completely arbitrary
```
vi /etc/grub.d/01_users

#!/bin/sh -e
cat << EOF
    set superusers="john"
    password john L1nux
EOF
```

We then need to rebuild our grub config and reboot the server for the changes to take effect.
If you try edit a kernel's settings from the GRUB menu, you will be asked for a username and password.
The username in this case being `john` and the password being `L1nux`
```
grub2-mkconfig -o /boot/grub2/grub.cfg

reboot
```

#### Setting an encrypted password
Plain text passwords held in configuration files are not ideal from a security standpoint.

To set an encryped password for GRUB editing We first need to generate an encrypted password,
which there is a utility for
```
[root@server1 ~]# grub2-mkpasswd-pbkdf2
Enter password:
Reenter password:
PBKDF2 hash of your password is grub.pbkdf2.sha512.10000.BFB67F3FD1ABD935FE90789403441064CCCAC7BCCC5DF99AA58C195995BE11579D28B03D1986E06AD3C9B8E0DDCCEA0761BF6954C2308D53CAC31EF9934C1D1F.BC4C351FEC8461A43EDC6C782062689FDECABFCBD7D3CB76ECD5EB940DDCD30C31C49571D7D0C3D6E1519A55A693370163F8B54012F2E75922F723E5EC4CBEBC
```

Taking the Hash output from the above command we edit the, change the password setting to encrypted and replace the plain text password with the Hashed password
```
vi /etc/grub.d/01_users

#!/bin/sh -e
cat << EOF
    set superusers="john"
    password_pbkdf2 john grub.pbkdf2.sha512.10000.BFB67F3FD1ABD935FE90789403441064CCCAC7BCCC5DF99AA58C195995BE11579D28B03D1986E06AD3C9B8E0DDCCEA0761BF6954C2308D53CAC31EF9934C1D1F.BC4C351FEC8461A43EDC6C782062689FDECABFCBD7D3CB76ECD5EB940DDCD30C31C49571D7D0C3D6E1519A55A693370163F8B54012F2E75922F723E5EC4CBEBC
EOF
```

We then need to rebuild our grub config and reboot the server for the changes to take effect
```
grub2-mkconfig -o /boot/grub2/grub.cfg

reboot
```

Finally re-instate the old users file to remove password protection from the GRUB menu
```
cp ~/01_users /etc/grub.d/
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

### Custom GRUB2 Entries

You can create custom GRUB menu entries using a similar syntax as follows
```
menuentry 'CentOS Custom' {
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'
        linux16 /vmlinuz-3.10.0-1160.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap
        initrd=/initramfs-3.10.0-1160.el7.x86_64.img
}
```
The above menuentry needs to be added to the bottom of `/etc/grub.d/40_custom` file
Then to enact the changes rebuild the GRUB config and reboot.  Your new menu entry will appear
on the Grub menu
```
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

## Managing Linux Processes

This section covers all things processes in Linux.  Remember in linux, everything is a file
or a process.

### Using ps (process status) command

The basic ps command shows us the process id, the terminal it's attached to, the CPU time amd the actual command
we are running
```
[root@server1 ~]# ps
  PID TTY          TIME CMD
 1521 pts/0    00:00:00 bash
 1536 pts/0    00:00:00 ps
 ```

To see **all** processes
 ```
 [root@server1 ~]# ps -e
  PID TTY          TIME CMD
    1 ?        00:00:01 systemd
    2 ?        00:00:00 kthreadd
    4 ?        00:00:00 kworker/0:0H
    5 ?        00:00:00 kworker/u2:0
    6 ?        00:00:00 ksoftirqd/0
    7 ?        00:00:00 migration/0
    8 ?        00:00:00 rcu_bh
    9 ?        00:00:00 rcu_sched
   10 ?        00:00:00 lru-add-drain
...
```

To get all user processes and processes not assigned to a terminal use `a` `u` and `x`
This gives us a bit more detail than the previous command
```
root@server1 ~]# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.6 128144  6780 ?        Ss   21:29   0:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root         2  0.0  0.0      0     0 ?        S    21:29   0:00 [kthreadd]
root         4  0.0  0.0      0     0 ?        S<   21:29   0:00 [kworker/0:0H]
root         5  0.0  0.0      0     0 ?        S    21:29   0:00 [kworker/u2:0]
root         6  0.0  0.0      0     0 ?        S    21:29   0:00 [ksoftirqd/0]
root         7  0.0  0.0      0     0 ?        S    21:29   0:00 [migration/0]
root         8  0.0  0.0      0     0 ?        S    21:29   0:00 [rcu_bh]
root         9  0.0  0.0      0     0 ?        R    21:29   0:00 [rcu_sched]
root        10  0.0  0.0      0     0 ?        S<   21:29   0:00 [lru-add-drain]
root        11  0.0  0.0      0     0 ?        S    21:29   0:00 [watchdog/0]
root        13  0.0  0.0      0     0 ?        S    21:29   0:00 [kdevtmpfs]
...
```

To see a process tree-like output
```
[root@server1 ~]# ps -e --forest
  PID TTY          TIME CMD
    2 ?        00:00:00 kthreadd
    4 ?        00:00:00  \_ kworker/0:0H
...
  730 ?        00:00:00 NetworkManager
  861 ?        00:00:00  \_ dhclient
  863 ?        00:00:00  \_ dhclient
 1102 ?        00:00:00 sshd
 1517 ?        00:00:00  \_ sshd
 1521 pts/0    00:00:00      \_ bash
 1601 pts/0    00:00:00          \_ ps
 1103 ?        00:00:00 tuned
 1105 ?        00:00:00 rsyslogd
 1115 tty1     00:00:00 agetty
 1118 ?        00:00:00 crond
 1120 ?        00:00:00 atd
 1351 ?        00:00:00 master
 1353 ?        00:00:00  \_ qmgr
 1458 ?        00:00:00  \_ pickup
 1444 ?        00:00:00 VBoxService
...
```
A better command for process trees
```
[root@server1 ~]# pstree
systemd─┬─NetworkManager─┬─2*[dhclient]
        │                └─2*[{NetworkManager}]
        ├─VBoxService───8*[{VBoxService}]
        ├─2*[abrt-watch-log]
        ├─abrtd
        ├─agetty
        ├─alsactl
        ├─atd
        ├─auditd─┬─audispd─┬─sedispatch
        │        │         └─{audispd}
        │        └─{auditd}
        ├─chronyd
        ├─crond
        ├─dbus-daemon───{dbus-daemon}
        ├─firewalld───{firewalld}
        ├─lvmetad
        ├─master─┬─pickup
        │        └─qmgr
        ├─polkitd───6*[{polkitd}]
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd───sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        └─tuned───4*[{tuned}]
```

To get a full listing, to get the User ID and the Parent ID of the process, including
when the process started (`STIME`)
```
[root@server1 ~]# ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      1521  1517  0 21:35 pts/0    00:00:00 -bash
root      1649  1521  0 21:45 pts/0    00:00:00 ps -f
```

To get an EXTRA-full listing, to get even more information
```
[root@server1 ~]# ps -F
UID        PID  PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
root      1521  1517  0 28887  2040   0 21:35 pts/0    00:00:00 -bash
root      1653  1521  0 38863  1840   0 21:47 pts/0    00:00:00 ps -F
```

Process Long listing, two different flavours
```
[root@server1 ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  1521  1517  0  80   0 - 28887 do_wai pts/0    00:00:00 bash
0 R     0  1700  1521  0  80   0 - 38332 -      pts/0    00:00:00 ps

[root@server1 ~]# ps -ly
S   UID   PID  PPID  C PRI  NI   RSS    SZ WCHAN  TTY          TIME CMD
S     0  1521  1517  0  80   0  2040 28887 do_wai pts/0    00:00:00 bash
R     0  1702  1521  0  80   0  1496 38332 -      pts/0    00:00:00 ps
```
To get a long listing and full Listing
```
[root@server1 ~]# ps -elf
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S root         1     0  0  80   0 - 32036 ep_pol 21:29 ?        00:00:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
1 S root         2     0  0  80   0 -     0 kthrea 21:29 ?        00:00:00 [kthreadd]
1 S root         4     2  0  60 -20 -     0 worker 21:29 ?        00:00:00 [kworker/0:0H]
1 S root         5     2  0  80   0 -     0 worker 21:29 ?        00:00:00 [kworker/u2:0]
1 S root         6     2  0  80   0 -     0 smpboo 21:29 ?        00:00:00 [ksoftirqd/0]
1 S root         7     2  0 -40   - -     0 smpboo 21:29 ?        00:00:00 [migration/0]
1 S root         8     2  0  80   0 -     0 rcu_gp 21:29 ?        00:00:00 [rcu_bh]
1 R root         9     2  0  80   0 -     0 -      21:29 ?        00:00:00 [rcu_sched]
1 S root        10     2  0  60 -20 -     0 rescue 21:29 ?        00:00:00 [lru-add-drain]
5 S root        11     2  0 -40   - -     0 smpboo 21:29 ?        00:00:00 [watchdog/0]
5 S root        13     2  0  80   0 -     0 devtmp 21:29 ?        00:00:00 [kdevtmpfs]
1 S root        14     2  0  60 -20 -     0 rescue 21:29 ?        00:00:00 [netns]
1 S root        15     2  0  80   0 -     0 watchd 21:29 ?        00:00:00 [khungtaskd]
...
```

Searching for a particular process
```
[root@server1 ~]# ps -elf | grep sshd
4 S root      1102     1  0  80   0 - 28234 poll_s 21:29 ?        00:00:00 /usr/sbin/sshd -D
4 S root      1517  1102  0  80   0 - 39735 poll_s 21:35 ?        00:00:00 sshd: root@pts/0
0 R root      1705  1521  0  80   0 - 28203 -      21:50 pts/0    00:00:00 grep --color=auto sshd
```

### The /prod Directory and the $$ Variable

The proc directory represents the running processes that you have on your system
as well as maintaining other files that have information about the system

The proc directory contains numbered directories, each number representing the process
id of a process;

```
root@server1 proc]# ls
1     1111  13    1449  16  20  25   284  291  33   391  396  400  45   504  594  601  647  674  716  9          bus       crypto     execdomains  iomem     keys        loadavg  modules       partitions   slabinfo  sysrq-trigger  uptime
10    1112  1357  1460  17  21  280  285  30   367  392  397  401  46   515  596  604  667  677  735  96         cgroups   devices    fb           ioports   key-users   locks    mounts        sched_debug  softirqs  sysvipc        version
102   1123  1358  1468  18  22  281  287  31   368  393  398  41   47   538  598  606  670  679  8    acpi       cmdline   diskstats  filesystems  irq       kmsg        mdstat   mtrr          schedstat    stat      timer_list     vmallocinfo
11    1125  1359  1484  19  23  282  288  32   377  394  399  43   482  589  6    643  672  686  866  asound     consoles  dma        fs           kallsyms  kpagecount  meminfo  net           scsi         swaps     timer_stats    vmstat
1109  1126  14    15    2   24  283  289  326  378  395  4    44   5    592  60   645  673  7    868  buddyinfo  cpuinfo   driver     interrupts   kcore     kpageflags  misc     pagetypeinfo  self         sys       tty            zoneinfo
```

You can get more indepth info for a process by passing in it's id to the ps command
```
[root@server1 proc]# ps -p1 -f
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 20:23 ?        00:00:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
```

To get the process id of the currently running process
```
[root@server1 proc]# echo $$
1468
```

To get extra full information for the current process use the following command
```
[root@server1 proc]# ps -p $$ -F
UID        PID  PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
root      1468  1460  0 28887  2072   0 20:24 pts/0    00:00:00 -bash
```

To get into the directoy of the current running process id
```
[root@server1 proc]# cd $$
[root@server1 1468]# pwd
/proc/1468
```

To interrogate even more information
```
[root@server1 1468]# ls -l cwd
lrwxrwxrwx. 1 root root 0 Mar  8 20:35 cwd -> /proc/1468
[root@server1 1468]# ls -l exe
lrwxrwxrwx. 1 root root 0 Mar  8 20:35 exe -> /usr/bin/bash
```

In the prod directory there is a loadavg file, that contains load averages regarding
running processes in the system.  The number of the right is the process id of the last command
that was run
```
[root@server1 proc]# ps
  PID TTY          TIME CMD
 1468 pts/0    00:00:00 bash
 1631 pts/0    00:00:00 ps
[root@server1 proc]# cat loadavg
0.00 0.01 0.05 2/129 1633
[root@server1 proc]#
[root@server1 proc]# cat loadavg
0.00 0.01 0.05 2/129 1634
[root@server1 proc]# cat loadavg
0.00 0.01 0.05 2/129 1635
```

The first three columns in the output of loadavg command shows the load average over the last minute, the last 5 mins, and
the last 15 mins.  The fourth column shows the number of processes running, out of a total.

### Send signals to processes with the command kill

To get a list of keyboard shortcuts
```
[root@server1 ~]# stty -a
speed 38400 baud; rows 56; columns 275; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = M-^?; eol2 = M-^?; swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; flush = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
-ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc ixany imaxbel iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke
```

To get a list of kill signals that can be sent
```
[root@server1 ~]# kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX
```

You can send kill signals using the number, the long word (SIGKILL) or the short
word (just take out the SIG and use the rest, eg TERM or KILL).  These are all
equivalent
```
[root@server1 ~]# kill -9 1731
[root@server1 ~]# kill -kill 1731
[root@server1 ~]# kill -sigkill 1731
```

When you remove a process, the directory for that process is removed from the `/proc`
directory.

The default signal is `15) SIGTERM` and that is basically a request to the process to shut
down gracefully.  If you use the kill command without a signal, eg `kill 1731`, it sends the
default signal

### Shortcuts with pgrep and pkill and the top command

To find processes running sshd
```
[root@server1 ~]# pgrep sshd
1111
1708
```

To get more indepth info
```
[root@server1 ~]# ps -F -p $(pgrep sshd)
UID        PID  PPID  C    SZ   RSS PSR STIME TTY      STAT   TIME CMD
root      1111     1  0 28234  4324   0 20:23 ?        Ss     0:00 /usr/sbin/sshd -D
root      1708  1111  0 39735  5756   0 20:45 ?        Ss     0:00 sshd: root@pts/0
```

To kill multiple processes for the same command (eg. sleep)
```
[root@server1 ~]# sleep 100&
[1] 1934
[root@server1 ~]# sleep 100&
[2] 1935
[root@server1 ~]# sleep 100&
[3] 1936
[root@server1 ~]# sleep 100&
[4] 1937
[root@server1 ~]# pgrep sleep
1934
1935
1936
1937
[root@server1 ~]# pkill sleep
[1]   Terminated              sleep 100
[2]   Terminated              sleep 100
[3]-  Terminated              sleep 100
[4]+  Terminated              sleep 100
```

The `top` command shows a list of running processes and sorts them based on processor
time by default
```
top - 21:05:17 up 42 min,  1 user,  load average: 0.00, 0.01, 0.05
Tasks: 104 total,   1 running, 103 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1014752 total,   619684 free,   185548 used,   209520 buff/cache
KiB Swap:   839676 total,   839676 free,        0 used.   680432 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    1 root      20   0  128144   6788   4188 S  0.0  0.7   0:01.37 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd
    4 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H
    5 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kworker/u2:0
    6 root      20   0       0      0      0 S  0.0  0.0   0:00.08 ksoftirqd/0
    7 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 migration/0
    8 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh
    9 root      20   0       0      0      0 S  0.0  0.0   0:00.68 rcu_sched
   10 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 lru-add-drain
   11 root      rt   0       0      0      0 S  0.0  0.0   0:00.02 watchdog/0
   13 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kdevtmpfs
   14 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 netns
   15 root      20   0       0      0      0 S  0.0  0.0   0:00.00 khungtaskd
   16 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 writeback
   17 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kintegrityd
   18 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 bioset
   19 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 bioset
   20 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 bioset
   21 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kblockd
```

`top` shows us things like the uptime, load averages, info on tasks, CPU and memory
utilization. To customize the `top` output, press the `F` key, and this will display a list of
fields that you can display with the top command.  To tell `top` what to sort on,
select a field and hit the `S` key.  Hit `esc` to return back to the main display, and
your changes will be apparent.

To get out of the `top` display hit the `q` key.

## Process Priority

### Backgrounding commands

A simple way to start a process and run it in the background is use `&`
Using `jobs` will show us the jobs we have running in the background
```
[root@server1 ~]# sleep 100&
[1] 2434
[root@server1 ~]# jobs
[1]+  Running                 sleep 100 &
```

You can suspend a process using CTRL-Z
```
[root@server1 ~]# sleep 500
^Z
[1]+  Stopped                 sleep 500
```

If you run the jobs command then, it will show you the command as being stopped.
It is still using memory but it is not taking CPU time. The `+` shows you which job
has focus.
```
[root@server1 ~]# jobs
[1]+  Stopped                 sleep 500
```

You can resume a job using `bg`, but it will resume in the background and allow you to
use the terminal. Using `bg` by itself resumes the job which has focus
```
[root@server1 ~]# bg
[1]+ sleep 500 &

[root@server1 ~]# jobs
[1]+  Running                 sleep 500 &
```

To bring a job into the foreground use the `fg` command.  This will mean that you have to
wait until the job completes before being able to use the terminal again
```
[root@server1 ~]# fg
sleep 500
```

If you want to bring a particular job to the foreground, use the `fg` command with the job number
(obtained from the jobs command)
```
[root@server1 ~]# fg 1
sleep 500
```

### Configuring process priority

You can see the CPU priority for a process from the PRI column in the output from
ps -l.  This is controlled by the nice value (NI column), which dictates the priority.
The default nice value is 0.  It can run from -20 to +19.  The highly the nice value
the nicer the application, or the less CPU time it's going to take.

The lower the PRI value, it means that the process is a higher priority.
```
[root@server1 ~]# sleep 1000&
[1] 1478
[root@server1 ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  1463  1459  0  80   0 - 28887 do_wai pts/0    00:00:00 bash
0 S     0  1478  1463  0  80   0 - 27014 hrtime pts/0    00:00:00 sleep
0 R     0  1479  1463  0  80   0 - 38332 -      pts/0    00:00:0
```

We can influence the `PRI` value by starting a process with the nice command and
passing in a nice value between -20 and +19. As we can see from the below the new
process has started with a nice value of 19,  and the PRI value is the default (80)
plus the nice value (+19) giving us 99.
```
root@server1 ~]# nice -n 19 sleep 1000&
[2] 1522
[root@server1 ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  1463  1459  0  80   0 - 28887 do_wai pts/0    00:00:00 bash
0 S     0  1478  1463  0  80   0 - 27014 hrtime pts/0    00:00:00 sleep
0 S     0  1522  1463  0  99  19 - 27014 hrtime pts/0    00:00:00 sleep
0 R     0  1523  1463  0  80   0 - 38332 -      pts/0    00:00:00 ps
```
NB To set a negative value you have to do it as an elevated user, such as root.

We can adjust the priority of a currently running process by using the `renice` command
NB An ordinary user cannot set the nice value at a lower nice value than it currently has, you can only
set it at a higher value.  Only the `root` user can do that
```
[root@server1 ~]# renice -n 10 -p 1463
1463 (process ID) old priority 0, new priority 10
[root@server1 ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0  1463  1459  0  90  10 - 28887 do_wai pts/0    00:00:00 bash
0 S     0  1478  1463  0  80   0 - 27014 hrtime pts/0    00:00:00 sleep
0 S     0  1522  1463  0  99  19 - 27014 hrtime pts/0    00:00:00 sleep
0 S     0  1571  1463  0  81   1 - 27014 hrtime pts/0    00:00:00 sleep
4 S     0  1573  1463  0  70 -10 - 27014 hrtime pts/0    00:00:00 sleep
0 R     0  1577  1463  0  90  10 - 38332 -      pts/0    00:00:00 ps
```

The root user can set the default nice value for users or groups.  You can add entries
to the limits.conf file.  The example shows you setting a nice value of +10 for
any process started by the centos user
```
[root@server1 ~]# vi /etc/security/limits.conf

centos - priority 10
```

## Monitoring Linux Performance

### Listing Standard Tools in procps-ng

To see what's included in the procps-ng package
```
[centos@server1 ~]$ rpm -ql procps-ng
/usr/bin/free
/usr/bin/pgrep
/usr/bin/pkill
/usr/bin/pmap
/usr/bin/ps
/usr/bin/pwdx
/usr/bin/skill
/usr/bin/slabtop
/usr/bin/snice
/usr/bin/tload
/usr/bin/top
/usr/bin/uptime
/usr/bin/vmstat
/usr/bin/w
/usr/bin/watch
/usr/lib64/libprocps.so.4
/usr/lib64/libprocps.so.4.0.0
/usr/sbin/sysctl
/usr/share/doc/procps-ng-3.3.10
/usr/share/doc/procps-ng-3.3.10/AUTHORS
/usr/share/doc/procps-ng-3.3.10/BUGS
/usr/share/doc/procps-ng-3.3.10/COPYING
/usr/share/doc/procps-ng-3.3.10/COPYING.LIB
/usr/share/doc/procps-ng-3.3.10/FAQ
/usr/share/doc/procps-ng-3.3.10/NEWS
/usr/share/doc/procps-ng-3.3.10/README
/usr/share/doc/procps-ng-3.3.10/README.top
/usr/share/doc/procps-ng-3.3.10/TODO
/usr/share/man/man1/free.1.gz
/usr/share/man/man1/pgrep.1.gz
/usr/share/man/man1/pkill.1.gz
/usr/share/man/man1/pmap.1.gz
/usr/share/man/man1/ps.1.gz
/usr/share/man/man1/pwdx.1.gz
/usr/share/man/man1/skill.1.gz
/usr/share/man/man1/slabtop.1.gz
/usr/share/man/man1/snice.1.gz
/usr/share/man/man1/tload.1.gz
/usr/share/man/man1/top.1.gz
/usr/share/man/man1/uptime.1.gz
/usr/share/man/man1/w.1.gz
/usr/share/man/man1/watch.1.gz
/usr/share/man/man5/sysctl.conf.5.gz
/usr/share/man/man8/sysctl.8.gz
/usr/share/man/man8/vmstat.8.gz
```

You can do it the other way around, by finding out what package a certain file
belongs to
```
[centos@server1 ~]$ rpm -qf /usr/bin/top
procps-ng-3.3.10-28.el7.x86_64
```
* - `rpm -ql procps-ng | grep '^/usr/bin/'` To show just the programs in a package
* - `rpm -qd procps-ng` To show just the documentation files in a package
* - `rpm -qc procps-ng` To show just the configuration files in a package

### Using pwdx and pmap

To see how much memory is free, and with options to express it in megabytes and gigabytes
```
[root@server1 ~]# free
              total        used        free      shared  buff/cache   available
Mem:        1014752      195892      594180        6924      224680      667988
Swap:        839676           0      839676
[root@server1 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:            990         191         580           6         219         652
Swap:           819           0         819
[root@server1 ~]# free -g
              total        used        free      shared  buff/cache   available
Mem:              0           0           0           0           0           0
Swap:             0           0           0
```
NB. Free only shows you complete units, and not partial units.  That's why when we
try to display free gigabytes, it comes back empty, because our free space is less than 1 gb

To get the memory map of a process.  You can see the size of the memory in use and
the breakdown by what is using it.  One of the useful things from the output is
that we can see the shared libraries that are being used by the process
```
[root@server1 ~]# pmap 7396
7396:   -bash
0000000000400000    888K r-x-- bash
00000000006dd000      4K r---- bash
00000000006de000     36K rw--- bash
00000000006e7000     24K rw---   [ anon ]
0000000000d63000    264K rw---   [ anon ]
00007f42de362000     48K r-x-- libnss_files-2.17.so
00007f42de36e000   2044K ----- libnss_files-2.17.so
00007f42de56d000      4K r---- libnss_files-2.17.so
00007f42de56e000      4K rw--- libnss_files-2.17.so
00007f42de56f000     24K rw---   [ anon ]
00007f42de575000 103692K r---- locale-archive
00007f42e4ab8000   1808K r-x-- libc-2.17.so
00007f42e4c7c000   2044K ----- libc-2.17.so
00007f42e4e7b000     16K r---- libc-2.17.so
00007f42e4e7f000      8K rw--- libc-2.17.so
00007f42e4e81000     20K rw---   [ anon ]
00007f42e4e86000      8K r-x-- libdl-2.17.so
00007f42e4e88000   2048K ----- libdl-2.17.so
00007f42e5088000      4K r---- libdl-2.17.so
00007f42e5089000      4K rw--- libdl-2.17.so
00007f42e508a000    148K r-x-- libtinfo.so.5.9
00007f42e50af000   2048K ----- libtinfo.so.5.9
00007f42e52af000     16K r---- libtinfo.so.5.9
00007f42e52b3000      4K rw--- libtinfo.so.5.9
00007f42e52b4000    136K r-x-- ld-2.17.so
00007f42e54c0000     12K rw---   [ anon ]
00007f42e54cb000      8K rw---   [ anon ]
00007f42e54cd000     28K r--s- gconv-modules.cache
00007f42e54d4000      4K rw---   [ anon ]
00007f42e54d5000      4K r---- ld-2.17.so
00007f42e54d6000      4K rw--- ld-2.17.so
00007f42e54d7000      4K rw---   [ anon ]
00007fff91b66000    132K rw---   [ stack ]
00007fff91b93000      8K r-x--   [ anon ]
ffffffffff600000      4K r-x--   [ anon ]
 total           115552K
```

To see the root file system the process is running with (or the working director
of a process)
```
root@server1 ~]# pwdx $$
7396: /root
[root@server1 ~]# pwdx $(pgrep sshd)
1105: /
7392: /
```

### Using uptime and tload

To see information how long the server has been up and running.  Shows you the
current time, the status, how long it's been up, the number of users logged on
and the system loads in the previous 1, 5 and 15 mins.
```
[root@server1 7396]# uptime
 20:35:15 up  9:01,  1 user,  load average: 0.00, 0.01, 0.05
 ```

 For load average values, it depends on the number of CPUs you have.  In our
 case, we have 1 CPU.  If the load value goes over 1.00 it means that the server is
 at capacity and that CPU requests will be queued.

 The w command comes from the procps package and is similar to the who command,
 but gives us a bit more information
 ```
 [root@server1 7396]# w
 20:39:37 up  9:05,  1 user,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    192.168.99.1     20:22    1.00s  0.03s  0.00s w
```

To find out how many CPUs we have and get information relating to it
```
[root@server1 7396]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                1
On-line CPU(s) list:   0
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 70
Model name:            Intel(R) Core(TM) i7-4770HQ CPU @ 2.20GHz
Stepping:              1
CPU MHz:               2194.918
BogoMIPS:              4389.83
Hypervisor vendor:     KVM
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              6144K
L4 cache:              131072K
NUMA node0 CPU(s):     0
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc eagerfpu pni pclmulqdq monitor ssse3 cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm invpcid_single fsgsbase avx2 invpcid md_clear flush_l1d
```

To get some more information about uptime.  The values are shown in seconds, with
the value on the left the seconds the system has been up, and the value on the right
how many seconds the CPU has been idle.  The idle time can be greater than your up time
but that's just an indication that you have more than 1 CPU
```
[root@server1 7396]# cat /proc/uptime
33003.90 32944.68
```

To get some more info about loadavg
```
[root@server1 7396]# cat /proc/loadavg.  Shows the load avgs for previous 1, 10, 15 mins
the number of active processes and the last process id that was used.
0.00 0.01 0.05 2/129 7704
```

Using watch to run commands such as uptime at intervals of your choosing.  This
command will run uptime every 4 seconds.  The default interval is 2 seconds.
```
[root@server1 7396]# watch -n 4 uptime

Every 4.0s: uptime                                                                                                                                                                                                 Wed Mar 10 20:49:41 2021

 20:49:41 up  9:15,  1 user,  load average: 0.02, 0.02, 0.05
```

To monitor your load averages in real time use tload.  The values will change if
the load averages change
```
[root@server1 7396]# tload

0.01, 0.02, 0.05 ---------
```

### Using top and vmstat

To run top in batch mode to capture information over a specified number of iterations,
in this case 1 interation (`-n1`)
```
[root@server1 7396]# top -b -n1

top - 20:59:07 up  9:25,  1 user,  load average: 0.00, 0.01, 0.05
Tasks: 102 total,   1 running, 101 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1014752 total,   593016 free,   195948 used,   225788 buff/cache
KiB Swap:   839676 total,   839676 free,        0 used.   667856 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    1 root      20   0  128144   6788   4188 S  0.0  0.7   0:02.24 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd
    4 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H
    5 root      20   0       0      0      0 S  0.0  0.0   0:00.28 kworker/u2:0
    6 root      20   0       0      0      0 S  0.0  0.0   0:00.40 ksoftirqd/0
    7 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 migration/0
    8 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh
...
```

To run top in batch mode to capture information over a specified number of iterations,
in this case 1 interation (`-n1`) and send to a file
```
[root@server1 7396]# top -b -n1 > file1
```

To run vmstat just once run vmstat.  Reading from left to right, it shows the number
of running processes, the number of blocked processes, memory information, swap in and
swap out, bytes in and bytes out, system info and CPU info
```
[root@server1 ~]# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 589192   2116 226280    0    0     6     3   20   39  0  0 100  0  0

 [root@server1 ~]# vmstat -S m
 procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
  r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
  2  0      0    604      2    231    0    0     6     3   20   39  0  0 100  0  0
 ```

 To run vmstat at an interval and for a number of iterations. In this example vmstat
 will be run every 5 seconds, for 3 iterations
 ```
 [root@server1 ~]# vmstat 5 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 3  0      0 590328   2116 226308    0    0     6     3   20   39  0  0 100  0  0
 0  0      0 590376   2116 226308    0    0     0    12   37   67  0  0 100  0  0
 0  0      0 590376   2116 226308    0    0     0     0   30   57  0  0 100  0  0
 ```

 ## Using sysstat to monitor performance

 ### Installing sysstat and initial configuration

 ```
[root@server1 ~]# yum install -y sysstat
```

As part of the install it will create a new cron job that will run sysstat every
10 mins and then a final summary wrap up at the end of the day.  Some additional
config is available at `etc/sysconfig/sysstat`
```
[root@server1 ~]#
[root@server1 ~]# cat /etc/cron.d/sysstat
# Run system activity accounting tool every 10 minutes
*/10 * * * * root /usr/lib64/sa/sa1 1 1
# 0 * * * * root /usr/lib64/sa/sa1 600 6 &
# Generate a daily summary of process accounting at 23:53
53 23 * * * root /usr/lib64/sa/sa2 -A
```

To start the sysstat service, enable it to ensure it starts when the system starts,
and to get it's status
```
[root@server1 ~]# systemctl start sysstat
[root@server1 ~]# systemctl enable sysstat
[root@server1 ~]# systemctl status sysstat
● sysstat.service - Resets System Activity Logs
   Loaded: loaded (/usr/lib/systemd/system/sysstat.service; enabled; vendor preset: enabled)
   Active: active (exited) since Wed 2021-03-10 21:18:23 GMT; 15s ago
 Main PID: 8279 (code=exited, status=0/SUCCESS)

Mar 10 21:18:23 server1.example.com systemd[1]: Starting Resets System Activity Logs...
Mar 10 21:18:23 server1.example.com systemd[1]: Started Resets System Activity Logs.
```

### Using additional sysstat tools

To look at disk activity, default is kbs, but you can use `iostat -m` to view in mbs
```
[root@server1 ~]# iostat
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	10/03/21 	_x86_64_	(1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.03    0.00    0.09    0.01    0.00   99.86

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.65        10.80         5.62     379680     197545
dm-0              0.72        10.44         5.81     367064     204043
dm-1              0.00         0.06         0.00       2204          0
```

To run iostat over a certain interval for a number of iterations.  The below is
using a 5 second interval and repeating iostat 3 times
```
[root@server1 ~]# iostat -m 5 3
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	10/03/21 	_x86_64_	(1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.03    0.00    0.09    0.01    0.00   99.86

Device:            tps    MB_read/s    MB_wrtn/s    MB_read    MB_wrtn
sda               0.65         0.01         0.01        370        193
dm-0              0.72         0.01         0.01        358        199
dm-1              0.00         0.00         0.00          2          0

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    0.20    0.00    0.00   99.80

Device:            tps    MB_read/s    MB_wrtn/s    MB_read    MB_wrtn
sda               0.00         0.00         0.00          0          0
dm-0              0.00         0.00         0.00          0          0
dm-1              0.00         0.00         0.00          0          0

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    0.00    0.00    0.00  100.00

Device:            tps    MB_read/s    MB_wrtn/s    MB_read    MB_wrtn
sda               0.00         0.00         0.00          0          0
dm-0              0.00         0.00         0.00          0          0
dm-1              0.00         0.00         0.00          0          0
```

To get stats for a particular process (this example using the current process),
for a certain interval and a certain number of iterations
```
[root@server1 ~]# pidstat -p $$ 5 3
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	10/03/21 	_x86_64_	(1 CPU)

21:25:41      UID       PID    %usr %system  %guest    %CPU   CPU  Command
21:25:46        0      7396    0.00    0.00    0.00    0.00     0  bash
21:25:51        0      7396    0.00    0.00    0.00    0.00     0  bash
21:25:56        0      7396    0.00    0.00    0.00    0.00     0  bash
Average:        0      7396    0.00    0.00    0.00    0.00     -  bash
```

To get more processor stat information, in this case for ALL processes
```
[root@server1 ~]# mpstat -P ALL 2 3
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	10/03/21 	_x86_64_	(1 CPU)

21:27:20     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
21:27:22     all    0.00    0.00    0.00    0.00    0.00    0.50    0.00    0.00    0.00   99.50
21:27:22       0    0.00    0.00    0.00    0.00    0.00    0.50    0.00    0.00    0.00   99.50

21:27:22     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
21:27:24     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
21:27:24       0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

21:27:24     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
21:27:26     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
21:27:26       0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

Average:     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
Average:     all    0.00    0.00    0.00    0.00    0.00    0.17    0.00    0.00    0.00   99.83
Average:       0    0.00    0.00    0.00    0.00    0.00    0.17    0.00    0.00    0.00   99.83
```

### Creating System activity reports using sar

Sar can help build up a picture of the system bottlenecks during the day

To report on CPU utilization (which is the default with sar), use `sar -u` or just
`sar`
```
[root@server1 cron.d]# sar -u
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	11/03/21 	_x86_64_	(1 CPU)

21:20:08          LINUX RESTART

21:25:26        CPU     %user     %nice   %system   %iowait    %steal     %idle
21:25:27        all      0.00      0.00      0.00      0.00      0.00    100.00
21:25:28        all      0.00      0.00      0.00      0.00      0.00    100.00
21:25:29        all      0.00      0.00      0.00      0.00      0.00    100.00
21:25:29        all      0.00      0.00      1.89      0.00      0.00     98.11
21:25:30        all      1.92      0.00      0.00      0.00      0.00     98.08
21:25:30        all      0.00      0.00      0.00      0.00      0.00    100.00
```

To report on memory utilization use `sar -r`
```
[root@server1 cron.d]# sar -r
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	11/03/21 	_x86_64_	(1 CPU)

21:20:08          LINUX RESTART

21:25:26    kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
21:25:27       621292    393460     38.77      2116    177920    332732     17.94    131424    122408         4
21:25:28       621300    393452     38.77      2116    177924    332732     17.94    131440    122396         8
21:25:29       621308    393444     38.77      2116    177928    332732     17.94    131408    122396        12
21:25:29       621348    393404     38.77      2116    177928    332732     17.94    131444    122396        12
21:25:30       621176    393576     38.79      2116    177932    332732     17.94    131412    122396        16
21:25:30       621308    393444     38.77      2116    177932    332732     17.94    131448    122396        16
21:25:31       621176    393576     38.79      2116    177936    332732     17.94    131452    122396        20
21:25:32       621176    393576     38.79      2116    177944    332732     17.94    131460    122396        28
21:25:33       621244    393508     38.78      2116    177948    332732     17.94    131452    122396        32
Average:       621259    393493     38.78      2116    177932    332732     17.94    131438    122397        16
```

To report on disk IO
```
[root@server1 cron.d]# sar -b
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	11/03/21 	_x86_64_	(1 CPU)

21:20:08          LINUX RESTART

21:25:26          tps      rtps      wtps   bread/s   bwrtn/s
21:25:27         0.00      0.00      0.00      0.00      0.00
21:25:28         0.00      0.00      0.00      0.00      0.00
21:25:29         0.00      0.00      0.00      0.00      0.00
21:25:29         0.00      0.00      0.00      0.00      0.00
21:25:30         0.00      0.00      0.00      0.00      0.00
21:25:30         0.00      0.00      0.00      0.00      0.00
21:25:31         0.00      0.00      0.00      0.00      0.00
21:25:32         0.00      0.00      0.00      0.00      0.00
21:25:33         0.00      0.00      0.00      0.00      0.00
Average:         0.00      0.00      0.00      0.00      0.00
```

To report on network activity
```
[root@server1 cron.d]# sar -n DEV
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	11/03/21 	_x86_64_	(1 CPU)

21:20:08          LINUX RESTART

21:25:26        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
21:25:27       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:27       enp0s8      3.45      2.07      0.28      0.26      0.00      0.00      0.00
21:25:27           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:28       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:28       enp0s8      6.76      4.05      0.54      0.51      0.00      0.00      0.00
21:25:28           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:29       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:29       enp0s8      9.26      5.56      0.74      0.70      0.00      0.00      0.00
21:25:29           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:29       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:29       enp0s8      9.43      5.66      0.76      0.71      0.00      0.00      0.00
21:25:29           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:30       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:30       enp0s8      9.62      5.77      0.77      0.72      0.00      0.00      0.00
21:25:30           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:30       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:30       enp0s8      8.93      5.36      0.71      0.67      0.00      0.00      0.00
21:25:30           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:31       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:31       enp0s8      9.80      5.88      0.79      0.74      0.00      0.00      0.00
21:25:31           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:32       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:32       enp0s8      9.49      5.70      0.76      0.72      0.00      0.00      0.00
21:25:32           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:33       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
21:25:33       enp0s8      8.20      4.92      0.66      0.62      0.00      0.00      0.00
21:25:33           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:       enp0s3      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:       enp0s8      7.81      4.69      0.63      0.59      0.00      0.00      0.00
Average:           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```

To report on load averages
```
[root@server1 cron.d]#
[root@server1 cron.d]# sar -q
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	11/03/21 	_x86_64_	(1 CPU)

21:20:08          LINUX RESTART

21:25:26      runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
21:25:27            1       128      0.00      0.04      0.04         0
21:25:28            1       128      0.00      0.04      0.04         0
21:25:29            1       128      0.00      0.04      0.04         0
21:25:29            1       128      0.00      0.04      0.04         0
21:25:30            2       128      0.00      0.04      0.04         0
21:25:30            1       128      0.00      0.04      0.04         0
21:25:31            1       128      0.00      0.04      0.04         0
21:25:32            0       128      0.00      0.04      0.04         0
21:25:33            1       128      0.00      0.04      0.04         0
21:30:01            2       130      0.00      0.03      0.04         0
Average:            1       128      0.00      0.04      0.04         0
```

To run sar queries to just show data between a start and end time
```
[root@server1 ~]# sar -s 21:25:00 -e 21:25:30
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	11/03/21 	_x86_64_	(1 CPU)

21:25:26        CPU     %user     %nice   %system   %iowait    %steal     %idle
21:25:27        all      0.00      0.00      0.00      0.00      0.00    100.00
21:25:28        all      0.00      0.00      0.00      0.00      0.00    100.00
21:25:29        all      0.00      0.00      0.00      0.00      0.00    100.00
21:25:29        all      0.00      0.00      1.89      0.00      0.00     98.11
21:25:30        all      1.92      0.00      0.00      0.00      0.00     98.08
21:25:30        all      0.00      0.00      0.00      0.00      0.00    100.00
Average:        all      0.23      0.00      0.23      0.00      0.00     99.54
```

To view the sar data for a particular day of the month (in this case the 10th)
then we specify the sar file we want to open, which will be called sa10 for the
10th of the month
```
[root@server1 ~]# sar -f /var/log/sa/sa10
Linux 3.10.0-1160.15.2.el7.x86_64 (server1.example.com) 	10/03/21 	_x86_64_	(1 CPU)

21:18:23          LINUX RESTART

21:20:01        CPU     %user     %nice   %system   %iowait    %steal     %idle
21:30:01        all      0.01      0.00      0.08      0.01      0.00     99.90
21:40:01        all      0.02      0.00      0.07      0.00      0.00     99.91
21:50:01        all      0.01      0.00      0.06      0.00      0.00     99.92
22:00:02        all      0.01      0.00      0.07      0.01      0.00     99.92
Average:        all      0.01      0.00      0.07      0.00      0.00     99.91
```

## Managing Shared Libraries

### Viewing Shared Libraries

Shared libraries are libraries of code that can be shared by multiple programs
Each program will reference a particular set of libraries that will help it fulfill
it's function.  Shared libraries have a `.so` suffix

To view shared libraries for a particular program
```
[root@server1 ~]# ldd /usr/bin/ls
	linux-vdso.so.1 =>  (0x00007ffc0538d000)
	libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f3e13ca8000)
	libcap.so.2 => /lib64/libcap.so.2 (0x00007f3e13aa3000)
	libacl.so.1 => /lib64/libacl.so.1 (0x00007f3e1389a000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f3e134cc000)
	libpcre.so.1 => /lib64/libpcre.so.1 (0x00007f3e1326a000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007f3e13066000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f3e13ecf000)
	libattr.so.1 => /lib64/libattr.so.1 (0x00007f3e12e61000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f3e12c45000)

[root@server1 ~]# ldd /usr/bin/grep
	linux-vdso.so.1 =>  (0x00007ffc620fd000)
	libpcre.so.1 => /lib64/libpcre.so.1 (0x00007fd8ca889000)
	libc.so.6 => /lib64/libc.so.6 (0x00007fd8ca4bb000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fd8ca29f000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fd8caaeb000)
```
You can see from the above that both `ls` and `grep` share some libraries

To see the library modules loaded for a particular process (in this case, shell)
```
[root@server1 ~]# pmap $$
1471:   -bash
0000000000400000    888K r-x-- bash
00000000006dd000      4K r---- bash
00000000006de000     36K rw--- bash
00000000006e7000     24K rw---   [ anon ]
00000000018f1000    264K rw---   [ anon ]
00007fed6da72000     48K r-x-- libnss_files-2.17.so
00007fed6da7e000   2044K ----- libnss_files-2.17.so
00007fed6dc7d000      4K r---- libnss_files-2.17.so
00007fed6dc7e000      4K rw--- libnss_files-2.17.so
00007fed6dc7f000     24K rw---   [ anon ]
00007fed6dc85000 103692K r---- locale-archive
00007fed741c8000   1808K r-x-- libc-2.17.so
00007fed7438c000   2044K ----- libc-2.17.so
00007fed7458b000     16K r---- libc-2.17.so
00007fed7458f000      8K rw--- libc-2.17.so
00007fed74591000     20K rw---   [ anon ]
00007fed74596000      8K r-x-- libdl-2.17.so
00007fed74598000   2048K ----- libdl-2.17.so
00007fed74798000      4K r---- libdl-2.17.so
00007fed74799000      4K rw--- libdl-2.17.so
00007fed7479a000    148K r-x-- libtinfo.so.5.9
00007fed747bf000   2048K ----- libtinfo.so.5.9
00007fed749bf000     16K r---- libtinfo.so.5.9
00007fed749c3000      4K rw--- libtinfo.so.5.9
00007fed749c4000    136K r-x-- ld-2.17.so
00007fed74bd0000     12K rw---   [ anon ]
00007fed74bdb000      8K rw---   [ anon ]
00007fed74bdd000     28K r--s- gconv-modules.cache
00007fed74be4000      4K rw---   [ anon ]
00007fed74be5000      4K r---- ld-2.17.so
00007fed74be6000      4K rw--- ld-2.17.so
00007fed74be7000      4K rw---   [ anon ]
00007ffeb9f20000    132K rw---   [ stack ]
00007ffeb9feb000      8K r-x--   [ anon ]
ffffffffff600000      4K r-x--   [ anon ]
 total           115552K
 ```

 ### Setting the location of Shared Libraries

The standard locations for the library modules are `/lib` or `/lib64`
However on Centos7 these directories are symbolic links to other locations in the file
system
```
[root@server1 /]# ls -l
total 20
lrwxrwxrwx.   1 root root    7 Feb 27 15:41 bin -> usr/bin
dr-xr-xr-x.   5 root root 4096 Mar  2 10:41 boot
drwxr-xr-x.  20 root root 3120 Mar 11 21:20 dev
drwxr-xr-x. 118 root root 8192 Mar 11 21:20 etc
drwxr-xr-x.   3 root root   20 Feb 27 15:45 home
lrwxrwxrwx.   1 root root    7 Feb 27 15:41 lib -> usr/lib
lrwxrwxrwx.   1 root root    9 Feb 27 15:41 lib64 -> usr/lib64
drwxr-xr-x.   2 root root    6 Apr 11  2018 media
drwxr-xr-x.   2 root root    6 Apr 11  2018 mnt
drwxr-xr-x.   3 root root   39 Mar  2 10:40 opt
dr-xr-xr-x. 115 root root    0 Mar 11 21:20 proc
dr-xr-x---.   5 root root  255 Mar 10 21:00 root
drwxr-xr-x.  38 root root 1080 Mar 11 21:20 run
lrwxrwxrwx.   1 root root    8 Feb 27 15:41 sbin -> usr/sbin
drwxr-xr-x.   2 root root    6 Apr 11  2018 srv
dr-xr-xr-x.  13 root root    0 Mar 11 21:20 sys
drwxrwxrwt.  14 root root 4096 Mar 12 03:06 tmp
drwxr-xr-x.  13 root root  155 Feb 27 15:41 usr
drwxr-xr-x.  19 root root  267 Feb 27 15:47 var
```

Please note that if you cd into a dir that is symlinked to another dir, a `pwd -P`
will show you the true location
```
[root@server1 /]# cd /lib
[root@server1 lib]# pwd
/lib
[root@server1 lib]# pwd -P
/usr/lib
[root@server1 lib]# ls
abrt-java-connector  binfmt.d  debug   firewalld  fontconfig  gcc   kbd    kernel  lsb         modules         mozilla         os-release  python2.7  rpm       sendmail.postfix  sysctl.d  tmpfiles.d  udev
alsa                 cpp       dracut  firmware   games       grub  kdump  locale  modprobe.d  modules-load.d  NetworkManager  polkit-1    python3.6  sendmail  sse2              systemd   tuned       yum-plugins
```

The configuration files for shared libraries are at this location
```
[root@server1 lib]# cd /etc/ld.so.conf.d/
[root@server1 ld.so.conf.d]# ls -al
total 32
drwxr-xr-x.   2 root root  180 Feb 27 16:07 .
drwxr-xr-x. 118 root root 8192 Mar 11 21:20 ..
-rw-r--r--.   1 root root   26 Dec 15 16:33 bind-export-x86_64.conf
-rw-r--r--.   1 root root   19 Aug  9  2019 dyninst-x86_64.conf
-r--r--r--.   1 root root   63 Feb  3 15:10 kernel-3.10.0-1160.15.2.el7.x86_64.conf
-r--r--r--.   1 root root   63 Oct 19 17:23 kernel-3.10.0-1160.el7.x86_64.conf
-rw-r--r--.   1 root root   17 Oct  1 17:55 mariadb-x86_64.conf
```

To create your own shared library you should create a directory under /usr/local/lib/
and copy your lib into that director
```
[root@server1 ld.so.conf.d]# mkdir /usr/local/lib/pluralsight
[root@server1 ld.so.conf.d]# cp/root/mysharedlib.so !$
```

Next we need to create a library configuration file in /etc/ld.so.conf.d/ and
give it the location of our library files
```
[root@server1 ld.so.conf.d]# vi pluralsight.conf
/usr/local/lib/pluralsight/
```

If you want to change the default location of libraries to something different (say if
were developing new libs and you wanted to point at those libs).  Unsetting the path
reverts to the default location of the libraries
```
[root@server1 ld.so.conf.d]# echo $LD_LIBRARY_PATH

[root@server1 ld.so.conf.d]#export LD_LIBRARY_PATH=path to your libs
```

### Shared Library cache

The library cache file is `/etc/ld.so.cache`

To look at all the shared libraries currently in the cache, use the `ldconfig` command
It needs to be run as `root` user
```
[root@server1 ld.so.conf.d]# ldconfig -p
846 libs found in cache `/etc/ld.so.cache'
	p11-kit-trust.so (libc6,x86-64) => /lib64/p11-kit-trust.so
	libz.so.1 (libc6,x86-64) => /lib64/libz.so.1
	libyelp.so.0 (libc6,x86-64) => /lib64/libyelp.so.0
	libxtables.so.10 (libc6,x86-64) => /lib64/libxtables.so.10
	libxslt.so.1 (libc6,x86-64) => /lib64/libxslt.so.1
	libxshmfence.so.1 (libc6,x86-64) => /lib64/libxshmfence.so.1
	libxml2.so.2 (libc6,x86-64) => /lib64/libxml2.so.2
	libxmlsec1.so.1 (libc6,x86-64) => /lib64/libxmlsec1.so.1
	libxmlrpc_util.so.3 (libc6,x86-64) => /lib64/libxmlrpc_util.so.3
	libxmlrpc_server_cgi.so.3 (libc6,x86-64) => /lib64/libxmlrpc_server_cgi.so.3
	libxmlrpc_server_abyss.so.3 (libc6,x86-64) => /lib64/libxmlrpc_server_abyss.so.3
	libxmlrpc_server.so.3 (libc6,x86-64) => /lib64/libxmlrpc_server.so.3
	libxmlrpc_client.so.3 (libc6,x86-64) => /lib64/libxmlrpc_client.so.3
	libxmlrpc_abyss.so.3 (libc6,x86-64) => /lib64/libxmlrpc_abyss.so.3
	libxmlrpc.so.3 (libc6,x86-64) => /lib64/libxmlrpc.so.3
	libxklavier.so.16 (libc6,x86-64) => /lib64/libxklavier.so.16
...
```

To update the cache run `ldconfig` by itself.  If you look at the `/etc/ld.so.cache` you can see
that the cache has been updated with the new date
```
[root@server1 ld.so.conf.d]# ls -l /etc/ld.so.cache
-rw-r--r--. 1 root root 66432 Mar 10 21:14 /etc/ld.so.cache
[root@server1 ld.so.conf.d]# ldconfig
[root@server1 ld.so.conf.d]# ls -l /etc/ld.so.cache
-rw-r--r--. 1 root root 66432 Mar 12 10:26 /etc/ld.so.cache
[root@server1 ld.so.conf.d]#
```

`ldconfig -v` will show you more verbose information when you are updating your cache, such as
what new libs are being added to the cache for example

## Scheduling Tasks in Linux

### Simple shell scripts

Create a simple bash script to see what the disk free space is and mail it to the centos user
```
[centos@server1 ~]$ vi df.sh

#!/usr/bin/bash
FILE=/tmp/df.txt
df -h > $FILE
mail -s "df $(date +%F)" centos < $FILE && rm $FILE
```

In the above file the first line is the shebang, the interpreter.  This needs to be in there
On the second line we are creating a variable called FILE and assigning it a value.  When we
reference a variable we put a $ symbol in front of it.

### Scheduling Jobs with crond

```
root@server1 ~]# ls /etc/cron*
/etc/cron.deny  /etc/crontab

/etc/cron.d:
0hourly  raid-check  sysstat

/etc/cron.daily:
logrotate  man-db.cron

/etc/cron.hourly:
0anacron

/etc/cron.monthly:

/etc/cron.weekly:
```

To create a system cron, log in as root and update the crontab file
```
[root@server1 ~]# vi /etc/crontab

HELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
3 8-18 * * 1-5 root df -h
```
Crontab legend for time intervals : `min hour d-o-m month d-o-w`

In the above we can add a line at the bottom for the new cronjob we want to run
The first 5 values are minute, hour, day of month, month, day of week.  `*` is a
wildcard and can mean `any`.  For these values we can put in ranges that make sense,
so in the example above for hour I have specified `8-18` which is from 8am to 6pm, and `1-5`
for day of week, which is Mon to Fri. 0 and 7 can both be used to represent Sunday.

To create a user cron job, log in as that user and run `crontab -e`, the contents of
which could be;
```
MAILTO=centos
*/5 * * * * ls /etc
0 15 * * 5 /home/centos/df.sh
```

These are two separate cronjobs, one that runs every five minutes and performs `ls /etc`
and mails the output to centos.  The second job runs at the top of the hour at 3 pm on a Friday,
and it runs the df.sh script in the centos home dir.

To list the cronjobs
```
[centos@server1 ~]$ crontab -l
MAILTO=centos
*/5 * * * * ls /etc
0 15 * * 5 /home/centos/df.sh
```

crontab -r will remove the entire crontab file and delete all the jobs in it.  To
change or remove individual jobs within the crontab file, use crontab -e

### Scheduling Jobs with anacron

`cron` is reliant on the system being turned on when the job is due to run, if you turn on a
system and there were jobs that were due to run during the time the system was turned off, cron
will not go back and run them.

With `anacron` you can schedule jobs to happen a certain number of minutes after a system has
powered on.  Ideal for desktops and laptops that get powered off regularly.

The contents of anacrontab
```
[root@server1 ~]# cat /etc/anacrontab
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1	5	cron.daily		nice run-parts /etc/cron.daily
7	25	cron.weekly		nice run-parts /etc/cron.weekly
@monthly 45	cron.monthly		nice run-parts /etc/cron.monthly
```

anacrontab legend : `period delay job-id command`

To create a new job in anacrontab
```
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1       5       cron.daily              nice run-parts /etc/cron.daily
7       25      cron.weekly             nice run-parts /etc/cron.weekly
@monthly 45     cron.monthly            nice run-parts /etc/cron.monthly
1 45 backup tar -cf /tmp/backup /etc
```

Anacrontab will also show you when the `cron.daily`, `cron.weekly` and `cron.monthly` jobs are
configured to run.  In the case of `cron.daily`, it is scheduled to run every 1 day, 5 mins
after the system startup.  `@monthly` is a special directive for the `cron.monthly` use case.

To examine the crond service.  This is a system that controls cron and it is started up when
on system startup
```
[root@server1 ~]# systemctl status crond
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2021-03-11 21:20:13 GMT; 14h ago
 Main PID: 1127 (crond)
   CGroup: /system.slice/crond.service
           └─1127 /usr/sbin/crond -n

Mar 11 21:20:13 server1.example.com systemd[1]: Started Command Scheduler.
Mar 11 21:20:13 server1.example.com crond[1127]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 23% if used.)
Mar 11 21:20:15 server1.example.com crond[1127]: (CRON) INFO (running with inotify support)
Mar 12 10:58:01 server1.example.com crond[1127]: (*system*) RELOAD (/etc/crontab)
```

If you look in the cron.daily folder you will see a script that runs anacron once a day
```
[root@server1 ~]# cd /etc/cron.hourly/
[root@server1 cron.hourly]# ls
0anacron
[root@server1 cron.hourly]# cat 0anacron
#!/bin/sh
# Check whether 0anacron was run today already
if test -r /var/spool/anacron/cron.daily; then
    day=`cat /var/spool/anacron/cron.daily`
fi
if [ `date +%Y%m%d` = "$day" ]; then
    exit 0;
fi

# Do not run jobs when on battery power
if test -x /usr/bin/on_ac_power; then
    /usr/bin/on_ac_power >/dev/null 2>&1
    if test $? -eq 1; then
    exit 0
    fi
fi
/usr/sbin/anacron -s
[root@server1 cron.hourly]# ls -al
total 16
drwxr-xr-x.   2 root root   22 Jun  9  2014 .
drwxr-xr-x. 118 root root 8192 Mar 12 12:10 ..
-rwxr-xr-x.   1 root root  392 Aug  9  2019 0anacron
[root@server1 cron.hourly]#
```

### Scheduling Jobs with At daemon

The `at` daemon is used to schedule once-off irregular jobs.  The at daemon runs as
a service, which starts on system startup.  To check if it is installed and running
```
[root@server1 ~]# systemctl status atd
● atd.service - Job spooling tools
   Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2021-03-11 21:20:13 GMT; 15h ago
 Main PID: 1129 (atd)
   CGroup: /system.slice/atd.service
           └─1129 /usr/sbin/atd -f

Mar 11 21:20:13 server1.example.com systemd[1]: Started Job spooling tools.
```

To schedule a job using `at`
```
[root@server1 ~]# at noon
at> ls /etc
at> ls /tmp
at> <EOT>
job 1 at Sat Mar 13 12:00:00 2021
```
Noon is the next occurrence of noon. You can enter multiple commands to be run
as part of your job.  To close out your job hit `CTRL-D`

To see what jobs are in the `at` queue
```
[root@server1 ~]# atq
1	Sat Mar 13 12:00:00 2021 a root
```

You can also specify days in your scheduling
```
[root@server1 ~]# at wednesday
at> ls /etc
at> <EOT>
job 2 at Wed Mar 17 12:30:00 2021
```
It will run on the next available wednesday

To remove an `at` job, use `atrm` with the number of the job you want to delete
```
[root@server1 ~]# atrm 2
[root@server1 ~]# atq
1	Sat Mar 13 12:00:00 2021 a root
```

You can delete multiple jobs at the same time `atrm 1 2 3`

To schedule jobs at specific times and dates.  The year token has to come last,
the month and date segments can be swapped around
```
[root@server1 ~]# at 13:23 jun 23 2021
at> ls /etc
at> <EOT>
job 3 at Wed Jun 23 13:23:00 2021
[root@server1 ~]# atq
1	Sat Mar 13 12:00:00 2021 a root
3	Wed Jun 23 13:23:00 2021 a root
```

By default every user is able to schedule and run `at` or `cron` jobs.  But you can
prevent specific users from running either using the `deny` files, which are located
under `/etc`.  For users you want to deny, you just enter the name of each user on a separate line.
```
[root@server1 ~]# ls /etc/*.deny
/etc/at.deny  /etc/cron.deny  /etc/hosts.deny
```

You can also create `allow` files for `at` and `cron`, which means that only the people listed
in the allow files are allowed to schedule and run jobs.  If you have users listed in both `allow`
and `deny` for either `at` or `cron`, the `allow` file will take precedence.

## Log Files and Logrotate

### Auditing Login Events

To see every user account whether they have logged in or not, plus the time they
last logged in, if ever
```
[centos@server1 ~]$ lastlog
Username         Port     From             Latest
root             pts/0                     Fri Mar 12 12:07:08 +0000 2021
bin                                        **Never logged in**
daemon                                     **Never logged in**
adm                                        **Never logged in**
lp                                         **Never logged in**
sync                                       **Never logged in**
shutdown                                   **Never logged in**
halt                                       **Never logged in**
mail                                       **Never logged in**
operator                                   **Never logged in**
games                                      **Never logged in**
ftp                                        **Never logged in**
nobody                                     **Never logged in**
systemd-network                            **Never logged in**
dbus                                       **Never logged in**
polkitd                                    **Never logged in**
sshd                                       **Never logged in**
postfix                                    **Never logged in**
chrony                                     **Never logged in**
centos           pts/0                     Fri Mar 12 20:57:58 +0000 2021
tss                                        **Never logged in**
unbound                                    **Never logged in**
geoclue                                    **Never logged in**
usbmuxd                                    **Never logged in**
openvpn                                    **Never logged in**
abrt                                       **Never logged in**
setroubleshoot                             **Never logged in**
lightdm                                    **Never logged in**
nm-openconnect                             **Never logged in**
nm-openvpn                                 **Never logged in**
vboxadd                                    **Never logged in**
```

To see just the user accounts that have logged in, use grep -v to invert the search
```
[centos@server1 ~]$ lastlog | grep -v "Never"
Username         Port     From             Latest
root             pts/0                     Fri Mar 12 12:07:08 +0000 2021
centos           pts/0                     Fri Mar 12 20:57:58 +0000 2021
```

The `last` command gives us a history going back a couple of weeks.  The `last`
command reads all it's information from the `/var/log/wtmp` file
```
[centos@server1 ~]$ last
root     pts/0        192.168.99.1     Thu Mar 11 21:21   still logged in
reboot   system boot  3.10.0-1160.15.2 Thu Mar 11 21:20 - 21:01  (23:41)
root     pts/0        192.168.99.1     Wed Mar 10 20:22 - crash (1+00:57)
root     pts/1        192.168.99.1     Tue Mar  9 21:20 - 21:28  (00:07)
root     pts/0        192.168.99.1     Tue Mar  9 21:02 - 10:54  (13:52)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  9 21:01 - 21:01 (3+00:00)
root     pts/1        192.168.99.1     Mon Mar  8 21:31 - 21:33  (00:01)
root     pts/1        192.168.99.1     Mon Mar  8 20:45 - 20:54  (00:08)
root     pts/0        192.168.99.1     Mon Mar  8 20:45 - crash (1+00:15)
root     pts/0        192.168.99.1     Mon Mar  8 20:24 - 20:45  (00:21)
reboot   system boot  3.10.0-1160.15.2 Mon Mar  8 20:23 - 21:01 (4+00:38)
root     pts/0        192.168.99.1     Tue Mar  2 21:35 - crash (5+22:47)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 19:24 - 21:01 (10+01:37)
root     pts/0        192.168.99.1     Tue Mar  2 21:12 - crash  (-1:-48)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 18:57 - 21:01 (10+02:03)
root     pts/0        192.168.99.1     Tue Mar  2 20:50 - down   (00:11)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 18:45 - 21:02  (02:17)
root     pts/1        192.168.99.1     Tue Mar  2 20:44 - down   (00:05)
root     pts/0        192.168.99.1     Tue Mar  2 17:57 - down   (02:52)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 17:54 - 20:49  (02:55)
root     pts/0        192.168.99.1     Tue Mar  2 17:13 - down   (00:40)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 17:05 - 17:53  (00:48)
root     tty1                          Tue Mar  2 17:04 - 17:04  (00:00)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 17:03 - 17:53  (00:50)
root     pts/0        192.168.99.1     Tue Mar  2 17:00 - down   (00:02)
root     tty1                          Tue Mar  2 16:54 - 16:55  (00:01)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 16:54 - 17:03  (00:09)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 16:52 - 17:03  (00:10)
root     pts/0        192.168.99.1     Tue Mar  2 16:40 - down   (00:08)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 16:38 - 16:48  (00:10)
root     pts/0        192.168.99.1     Tue Mar  2 10:58 - crash  (05:39)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 10:57 - 16:48  (05:51)
centos   tty1         :0               Tue Mar  2 10:43 - 10:44  (00:01)
centos   :0                            Tue Mar  2 10:43 - down   (00:01)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 10:43 - 10:44  (00:01)
centos   pts/0        :0               Tue Mar  2 10:39 - crash  (00:04)
centos   tty1         :0               Tue Mar  2 10:38 - crash  (00:04)
centos   :0                            Tue Mar  2 10:38 - crash  (00:04)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 10:38 - 10:44  (00:06)
root     pts/0        192.168.99.1     Sat Feb 27 16:51 - 16:53  (00:01)
centos   tty1         :0               Sat Feb 27 16:50 - 16:53  (00:02)
centos   :0                            Sat Feb 27 16:50 - down   (00:02)
reboot   system boot  3.10.0-1160.15.2 Sat Feb 27 16:50 - 16:53  (00:03)
root     pts/0        192.168.99.1     Sat Feb 27 16:46 - crash  (00:03)
centos   tty1         :0               Sat Feb 27 16:46 - 16:49  (00:03)
centos   :0                            Sat Feb 27 16:46 - crash  (00:03)
reboot   system boot  3.10.0-1160.15.2 Sat Feb 27 16:45 - 16:53  (00:07)
reboot   system boot  3.10.0-1160.15.2 Sat Feb 27 16:36 - 16:53  (00:16)
reboot   system boot  3.10.0-1160.15.2 Sat Feb 27 16:35 - 16:53  (00:18)
centos   tty1         :0               Sat Feb 27 16:27 - crash  (00:07)
centos   :0                            Sat Feb 27 16:27 - crash  (00:07)
centos   tty1         :0               Sat Feb 27 16:21 - 16:21  (00:00)
centos   :0                            Sat Feb 27 16:21 - 16:21  (00:00)
root     pts/0        192.168.99.1     Sat Feb 27 15:56 - crash  (00:39)
root     tty1                          Sat Feb 27 15:47 - 15:55  (00:08)
reboot   system boot  3.10.0-1160.el7. Sat Feb 27 15:47 - 16:53  (01:06)

wtmp begins Sat Feb 27 15:47:00 2021
```

To get just the last 10 lines of history
```
[centos@server1 ~]$
[centos@server1 ~]$ last -n 10
root     pts/0        192.168.99.1     Thu Mar 11 21:21   still logged in
reboot   system boot  3.10.0-1160.15.2 Thu Mar 11 21:20 - 21:02  (23:42)
root     pts/0        192.168.99.1     Wed Mar 10 20:22 - crash (1+00:57)
root     pts/1        192.168.99.1     Tue Mar  9 21:20 - 21:28  (00:07)
root     pts/0        192.168.99.1     Tue Mar  9 21:02 - 10:54  (13:52)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  9 21:01 - 21:02 (3+00:01)
root     pts/1        192.168.99.1     Mon Mar  8 21:31 - 21:33  (00:01)
root     pts/1        192.168.99.1     Mon Mar  8 20:45 - 20:54  (00:08)
root     pts/0        192.168.99.1     Mon Mar  8 20:45 - crash (1+00:15)
root     pts/0        192.168.99.1     Mon Mar  8 20:24 - 20:45  (00:21)

wtmp begins Sat Feb 27 15:47:00 2021
```

To see all users still logged in, and when they logged in
```
[centos@server1 ~]$ last | grep "still"
root     pts/0        192.168.99.1     Thu Mar 11 21:21   still logged in
```

To get a history of reboot events
```
[centos@server1 ~]$ last reboot
reboot   system boot  3.10.0-1160.15.2 Thu Mar 11 21:20 - 21:05  (23:45)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  9 21:01 - 21:05 (3+00:04)
reboot   system boot  3.10.0-1160.15.2 Mon Mar  8 20:23 - 21:05 (4+00:42)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 19:24 - 21:05 (10+01:41)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 18:57 - 21:05 (10+02:07)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 18:45 - 21:02  (02:17)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 17:54 - 20:49  (02:55)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 17:05 - 17:53  (00:48)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 17:03 - 17:53  (00:50)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 16:54 - 17:03  (00:09)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 16:52 - 17:03  (00:10)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 16:38 - 16:48  (00:10)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 10:57 - 16:48  (05:51)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 10:43 - 10:44  (00:01)
reboot   system boot  3.10.0-1160.15.2 Tue Mar  2 10:38 - 10:44  (00:06)
reboot   system boot  3.10.0-1160.15.2 Sat Feb 27 16:50 - 16:53  (00:03)
reboot   system boot  3.10.0-1160.15.2 Sat Feb 27 16:45 - 16:53  (00:07)
reboot   system boot  3.10.0-1160.15.2 Sat Feb 27 16:36 - 16:53  (00:16)
reboot   system boot  3.10.0-1160.15.2 Sat Feb 27 16:35 - 16:53  (00:18)
reboot   system boot  3.10.0-1160.el7. Sat Feb 27 15:47 - 16:53  (01:06)

wtmp begins Sat Feb 27 15:47:00 2021
```

To look at the history for the user `centos`
```
[centos@server1 ~]$ last centos
centos   tty1         :0               Tue Mar  2 10:43 - 10:44  (00:01)
centos   :0                            Tue Mar  2 10:43 - down   (00:01)
centos   pts/0        :0               Tue Mar  2 10:39 - crash  (00:04)
centos   tty1         :0               Tue Mar  2 10:38 - crash  (00:04)
centos   :0                            Tue Mar  2 10:38 - crash  (00:04)
centos   tty1         :0               Sat Feb 27 16:50 - 16:53  (00:02)
centos   :0                            Sat Feb 27 16:50 - down   (00:02)
centos   tty1         :0               Sat Feb 27 16:46 - 16:49  (00:03)
centos   :0                            Sat Feb 27 16:46 - crash  (00:03)
centos   tty1         :0               Sat Feb 27 16:27 - crash  (00:07)
centos   :0                            Sat Feb 27 16:27 - crash  (00:07)
centos   tty1         :0               Sat Feb 27 16:21 - 16:21  (00:00)
centos   :0                            Sat Feb 27 16:21 - 16:21  (00:00)

wtmp begins Sat Feb 27 15:47:00 2021
```

To see the last failed login attempts, use the `lastb` command.  This reads from
the `/var/log/btmp` file and needs elevated privileges to run.
```
[centos@server1 ~]$ sudo lastb
root     ssh:notty    192.168.99.1     Mon Mar  8 20:23 - 20:23  (00:00)
root     ssh:notty    192.168.99.1     Sat Feb 27 16:51 - 16:51  (00:00)

btmp begins Sat Feb 27 16:51:06 2021
```

### Auditing Root Access (use of the su and sudo commands)

In the `/var/log` directory there are `secure` files that contain usage of the `su`
and `sudo` commands.  You can grep these files for usages of su and sudo.  The sudo
lines also show you what command was run.
```
[root@server1 ~]# cd /var/log
[root@server1 log]# ls secure*
secure  secure-20210310

[root@server1 log]# grep sudo secure*
secure:Mar 12 21:09:03 server1 sudo:  centos : TTY=pts/0 ; PWD=/home/centos ; USER=root ; COMMAND=/bin/lastb
secure:Mar 12 21:09:03 server1 sudo: pam_unix(sudo:session): session opened for user root by root(uid=0)
secure:Mar 12 21:09:03 server1 sudo: pam_unix(sudo:session): session closed for user root
secure:Mar 12 21:09:08 server1 sudo:  centos : TTY=pts/0 ; PWD=/home/centos ; USER=root ; COMMAND=/bin/lastb
secure:Mar 12 21:09:08 server1 sudo: pam_unix(sudo:session): session opened for user root by root(uid=0)
secure:Mar 12 21:09:08 server1 sudo: pam_unix(sudo:session): session closed for user root

[root@server1 log]# grep su: secure*
secure:Mar 10 10:54:54 server1 su: pam_unix(su-l:session): session closed for user centos
secure:Mar 10 10:54:54 server1 su: pam_unix(su-l:session): session closed for user root
secure:Mar 12 10:38:26 server1 su: pam_unix(su-l:session): session opened for user root by root(uid=0)
secure:Mar 12 10:38:35 server1 su: pam_unix(su-l:session): session opened for user centos by root(uid=0)
secure:Mar 12 10:52:11 server1 su: pam_unix(su-l:session): session opened for user root by root(uid=1000)
secure:Mar 12 11:02:00 server1 su: pam_unix(su-l:session): session closed for user root
secure:Mar 12 11:08:55 server1 su: pam_unix(su-l:session): session opened for user root by root(uid=1000)
secure:Mar 12 11:10:02 server1 su: pam_unix(su-l:session): session opened for user centos by root(uid=0)
```

### Scripting awk to analyse log files

You can use the `awk` utility to parse out the log lines in the `secure` files
and print out certain fields of lines that match against a certain search string

This is searching for lines containing the string `sudo` and the print $0 is going
to output the entire line
```
[root@server1 log]# awk '/sudo/ { print $0 } ' secure
Mar 12 21:09:03 server1 sudo:  centos : TTY=pts/0 ; PWD=/home/centos ; USER=root ; COMMAND=/bin/lastb
Mar 12 21:09:03 server1 sudo: pam_unix(sudo:session): session opened for user root by root(uid=0)
Mar 12 21:09:03 server1 sudo: pam_unix(sudo:session): session closed for user root
Mar 12 21:09:08 server1 sudo:  centos : TTY=pts/0 ; PWD=/home/centos ; USER=root ; COMMAND=/bin/lastb
Mar 12 21:09:08 server1 sudo: pam_unix(sudo:session): session opened for user root by root(uid=0)
Mar 12 21:09:08 server1 sudo: pam_unix(sudo:session): session closed for user root
```

To print out just the command, which is column 5;
```
[root@server1 log]# awk '/sudo/ { print $5 } ' secure
sudo:
sudo:
sudo:
sudo:
sudo:
sudo:
```

To print out the command, the user and the actual end command that was executed;
```
[root@server1 log]# awk '/sudo/ { print $5, $6, $14 } ' secure
sudo: centos COMMAND=/bin/lastb
sudo: pam_unix(sudo:session):
sudo: pam_unix(sudo:session):
sudo: centos COMMAND=/bin/lastb
sudo: pam_unix(sudo:session):
sudo: pam_unix(sudo:session):
```

You can put this command into a bash script that you can re-run
```
root@server1 ~]# vi secure.sh

#!/usr/bin/bash
awk "/$1/ { print \$5, \$6, \$14 }" $2
```

```
[root@server1 ~]# chmod +x secure.sh
[root@server1 ~]# ./secure.sh sudo: /var/log/secure
sudo: centos COMMAND=/bin/lastb
sudo: pam_unix(sudo:session):
sudo: pam_unix(sudo:session):
sudo: centos COMMAND=/bin/lastb
sudo: pam_unix(sudo:session):
sudo: pam_unix(sudo:session):
```

### Configuring System Logging

`rsyslog` is a rocket-fast syslog daemon.  It improves on the standard syslog daemon,
it is much quicker and it can integrate with various database systems, ie. you
can write through to databases instead of just flat files.
```
[root@server1 etc]# systemctl status rsyslog
● rsyslog.service - System Logging Service
   Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2021-03-13 15:02:57 GMT; 7s ago
     Docs: man:rsyslogd(8)
           http://www.rsyslog.com/doc/
 Main PID: 1688 (rsyslogd)
   CGroup: /system.slice/rsyslog.service
           └─1688 /usr/sbin/rsyslogd -n

Mar 13 15:02:57 server1.example.com systemd[1]: Stopped System Logging Service.
Mar 13 15:02:57 server1.example.com systemd[1]: Starting System Logging Service...
Mar 13 15:02:57 server1.example.com rsyslogd[1688]:  [origin software="rsyslogd" swVersion="8.24.0-57.el7_9" x-pid="1688" x-info="http://www.rsyslog.com"] start
Mar 13 15:02:57 server1.example.com systemd[1]: Started System Logging Service.
```

The rsyslog config files are located in the `/etc` directory
```
[root@server1 etc]# ls rsyslog*
rsyslog.conf

rsyslog.d:
listen.conf
```

To look at the config for rsyslog, run `vi rsyslog.conf`

The first thing you see are the modules that can be loaded.
```
#### MODULES ####

# The imjournal module bellow is now used as a message source instead of imuxsock.
$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imjournal # provides access to the systemd journal
#$ModLoad imklog # reads kernel messages (the same are read from journald)
#$ModLoad immark  # provides --MARK-- message capability

# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514

# Provides TCP syslog reception
#$ModLoad imtcp
#$InputTCPServerRun 514
```


It also shows the logging rules.  The `.*`` means that all logging levels are enabled
(such as `INFO`, `DEBUG`, `ERROR` etc.).  If you look at the rule for `/var/log/messages`
it stipulates that everything with .info level debugging gets written there APART
from `mail`, `authpriv` and `cron`
```
#### RULES ####

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure

# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog


# Log cron stuff
cron.*                                                  /var/log/cron

# Everybody gets emergency messages
*.emerg                                                 :omusrmsg:*

# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log


# ### begin forwarding rule ###
# The statement between the begin ... end define a SINGLE forwarding
# rule. They belong together, do NOT split them. If you create multiple
# forwarding rules, duplicate the whole block!
# Remote Logging (we use TCP for reliable delivery)
#
# An on-disk queue is created for this action. If the remote host is
# down, messages are spooled to disk and sent when it is up again.
#$ActionQueueFileName fwdRule1 # unique name prefix for spool files
#$ActionQueueMaxDiskSpace 1g   # 1gb space limit (use as much as possible)
#$ActionQueueSaveOnShutdown on # save messages to disk on shutdown
#$ActionQueueType LinkedList   # run asynchronously
#$ActionResumeRetryCount -1    # infinite retries if host is down
# remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
#*.* @@remote-host:514
# ### end of the forwarding rule ###
```

You can add in your own logging rule, add a line like the following to the conf file
The local1 facility will log info logging and higher to the /var/log/john file, which
will be created when logs entries are generated.  You need to restart the `rsyslog`
daemon for the new rule to come into effect with `systemctl restart rsyslog`
```
local1.info /var/log/john
```

We can then generate a log message using the `logger` program, and it will show up
in both `/var/log/messages` and `/var/log/john`
```
root@server1 etc]# logger -p local1.warn "Test WARNING"
[root@server1 etc]# tail /var/log/messages
Mar 13 15:03:03 server1 NetworkManager[739]: <info>  [1615647783.0023] dhcp4 (enp0s3):   domain name 'home'
Mar 13 15:03:03 server1 NetworkManager[739]: <info>  [1615647783.0023] dhcp4 (enp0s3): state changed bound -> bound
Mar 13 15:03:03 server1 dbus[679]: [system] Activating via systemd: service name='org.freedesktop.nm_dispatcher' unit='dbus-org.freedesktop.nm-dispatcher.service'
Mar 13 15:03:03 server1 systemd: Starting Network Manager Script Dispatcher Service...
Mar 13 15:03:03 server1 dhclient[870]: bound to 10.0.2.9 -- renewal in 252 seconds.
Mar 13 15:03:03 server1 dbus[679]: [system] Successfully activated service 'org.freedesktop.nm_dispatcher'
Mar 13 15:03:03 server1 systemd: Started Network Manager Script Dispatcher Service.
Mar 13 15:03:03 server1 nm-dispatcher: req:1 'dhcp4-change' [enp0s3]: new request (4 scripts)
Mar 13 15:03:03 server1 nm-dispatcher: req:1 'dhcp4-change' [enp0s3]: start running ordered scripts...
Mar 13 15:04:35 server1 root: Test WARNING
[root@server1 etc]# tail /var/log/john
Mar 13 15:04:35 server1 root: Test WARNING
```

### Rotating log files

If log files are left to grow and accumulate unchecked, they will eventually fill
the hard disk of our system. We can prevent this from happening by clearing down
old logs based on how old they are or based on how big the log files get.  This can
be controlled by the `logrotate` utility.

`logrotate` is run daily as part of a daily cron job
```
[root@server1 etc]# cd /etc/cron.daily/
[root@server1 cron.daily]# pwd
/etc/cron.daily
[root@server1 cron.daily]# ls
logrotate  man-db.cron
```

The configuration for logrotate is `/etc/logrotate.conf`.
```
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
        minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```

We can add a simple rule for the log file we created above.  Adding the following
will ensure our `/var/log/john` log file will get rotated.
* `missingok` - means that if the log file is not there, then no error will be thrown
* `notifempty` - means that it will not attempt to rotate any empty log files
* `size 10` - it will rotate the logfile if it exceeds 10 bytes in size.  This overrides the weekly setting above, as you can rotate either on a schedule or on log file size.
* `compress` - it will compress the rotated log file

It will also use some global defaults specified in the conf file such as
* `create` - a new empty log file will be created after the old one is rotated.
* `rotate 4` - it will maintain 4 old rotated logs at any one time.  Older log files get deleted
```
/var/log/john {
    missingok
    notifempty
    size 10
    compress
}
```

Once you have updated the logrotate.conf file then you can manually run logrotate
and you can see that your log file has been rotated.
```
[root@server1 etc]# logrotate /etc/logrotate.conf
[root@server1 etc]# ls /var/log/john*
/var/log/john  /var/log/john-20210313.gz
```

### Using Journalctl

Centos7 is `systemd` based, and being `systemd` based means we have access to
`journalctl` to read from a unified log.

The configuration for journalctl is located here : `/etc/systemd/journald.conf`

To look at journalctl.  It uses the same paging and search system as `less`
```
[root@server1 etc]# journalctl
-- Logs begin at Sat 2021-03-13 14:43:42 GMT, end at Sat 2021-03-13 20:56:03 GMT. --
Mar 13 14:43:42 server1.example.com systemd-journal[89]: Runtime journal is using 6.1M (max allowed 49.5M, trying to leave 74.3M free of 489.2M available → current limit 49.5M).
Mar 13 14:43:42 server1.example.com kernel: Initializing cgroup subsys cpuset
Mar 13 14:43:42 server1.example.com kernel: Initializing cgroup subsys cpu
Mar 13 14:43:42 server1.example.com kernel: Initializing cgroup subsys cpuacct
Mar 13 14:43:42 server1.example.com kernel: Linux version 3.10.0-1160.15.2.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) ) #1 SMP Wed Feb 3 15:06:38 UTC 2021
Mar 13 14:43:42 server1.example.com kernel: Command line: BOOT_IMAGE=/vmlinuz-3.10.0-1160.15.2.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet
Mar 13 14:43:42 server1.example.com kernel: e820: BIOS-provided physical RAM map:
Mar 13 14:43:42 server1.example.com kernel: BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
Mar 13 14:43:42 server1.example.com kernel: BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
Mar 13 14:43:42 server1.example.com kernel: BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
Mar 13 14:43:42 server1.example.com kernel: BIOS-e820: [mem 0x0000000000100000-0x000000003ffeffff] usable
Mar 13 14:43:42 server1.example.com kernel: BIOS-e820: [mem 0x000000003fff0000-0x000000003fffffff] ACPI data
Mar 13 14:43:42 server1.example.com kernel: BIOS-e820: [mem 0x00000000fec00000-0x00000000fec00fff] reserved
Mar 13 14:43:42 server1.example.com kernel: BIOS-e820: [mem 0x00000000fee00000-0x00000000fee00fff] reserved
Mar 13 14:43:42 server1.example.com kernel: BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
Mar 13 14:43:42 server1.example.com kernel: NX (Execute Disable) protection: active
Mar 13 14:43:42 server1.example.com kernel: SMBIOS 2.5 present.
Mar 13 14:43:42 server1.example.com kernel: DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 12/01/2006
Mar 13 14:43:42 server1.example.com kernel: Hypervisor detected: KVM
Mar 13 14:43:42 server1.example.com kernel: e820: update [mem 0x00000000-0x00000fff] usable ==> reserved
Mar 13 14:43:42 server1.example.com kernel: e820: remove [mem 0x000a0000-0x000fffff] usable
Mar 13 14:43:42 server1.example.com kernel: e820: last_pfn = 0x3fff0 max_arch_pfn = 0x400000000
Mar 13 14:43:42 server1.example.com kernel: MTRR default type: uncachable
Mar 13 14:43:42 server1.example.com kernel: MTRR variable ranges disabled:
Mar 13 14:43:42 server1.example.com kernel: PAT configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- UC
Mar 13 14:43:42 server1.example.com kernel: CPU MTRRs all blank - virtualized system.
Mar 13 14:43:42 server1.example.com kernel: found SMP MP-table at [mem 0x0009fff0-0x0009ffff] mapped at [ffffffffff200ff0]
Mar 13 14:43:42 server1.example.com kernel: Base memory trampoline at [ffff918c00099000] 99000 size 24576
Mar 13 14:43:42 server1.example.com kernel: BRK [0x20474000, 0x20474fff] PGTABLE
Mar 13 14:43:42 server1.example.com kernel: BRK [0x20475000, 0x20475fff] PGTABLE
Mar 13 14:43:42 server1.example.com kernel: BRK [0x20476000, 0x20476fff] PGTABLE
Mar 13 14:43:42 server1.example.com kernel: BRK [0x20477000, 0x20477fff] PGTABLE
Mar 13 14:43:42 server1.example.com kernel: BRK [0x20478000, 0x20478fff] PGTABLE
Mar 13 14:43:42 server1.example.com kernel: RAMDISK: [mem 0x343cb000-0x361ddfff]
Mar 13 14:43:42 server1.example.com kernel: Early table checksum verification disabled
...
```

To see the most recent events, `journalctl -n 15` will show the last 15 entries.
When `-n` is used by itself, the default number of entries shown is 10.
To follow or tail the journalctl: `journalctl -f`.  `journalctl -b` will show the
entries since the most recent boot, but that's the default behaviour in any case.
Journal doesn't persist old entries, it get's wiped when the machine is restarted

If you want to just look at entries being produced by a specific service or "unit"
you can run something like the following
```
[root@server1 etc]# journalctl -u sshd.service
-- Logs begin at Sat 2021-03-13 14:43:42 GMT, end at Sat 2021-03-13 21:08:59 GMT. --
Mar 13 14:43:54 server1.example.com systemd[1]: Starting OpenSSH server daemon...
Mar 13 14:43:54 server1.example.com sshd[1114]: Server listening on 0.0.0.0 port 22.
Mar 13 14:43:54 server1.example.com sshd[1114]: Server listening on :: port 22.
Mar 13 14:43:54 server1.example.com systemd[1]: Started OpenSSH server daemon.
Mar 13 14:49:05 server1.example.com sshd[1513]: Accepted password for root from 192.168.99.1 port 50469 ssh2
```

You can narrow down your search with time ranges, using the `--since` and `--until`
flags
```
[root@server1 etc]# journalctl --since "2021-01-11 10:00:00"
-- Logs begin at Sat 2021-03-13 14:43:42 GMT, end at Sat 2021-03-13 21:11:33 GMT. --
Mar 13 14:43:42 server1.example.com systemd-journal[89]: Runtime journal is using 6.1M (max allowed 49.5M, trying to leave 74.3M free of 489.2M available → current limit 49.5M).
Mar 13 14:43:42 server1.example.com kernel: Initializing cgroup subsys cpuset
Mar 13 14:43:42 server1.example.com kernel: Initializing cgroup subsys cpu
Mar 13 14:43:42 server1.example.com kernel: Initializing cgroup subsys cpuacct
...
```

Time queries like ```[root@server1 etc]# journalctl --since "10 minutes ago"```
are also possible

To make journalctl reboot persistent there are two approaches. You can do both to
really ensure that journalctl is persistent.  You need to reboot the system for
these measures to take effect.
1) Create a journal directory under /var/log
```
[root@server1 etc]# mkdir /var/log/journal
```

2) Edit the `journald.conf` file, make sure the `storage` setting is enabled and the
value is `persistent`
```
[root@server1 etc]# vi /etc/systemd/journald.conf

#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See journald.conf(5) for details.

[Journal]
Storage=persistent
#Compress=yes
#Seal=yes
#SplitMode=uid
#SyncIntervalSec=5m
#RateLimitInterval=30s
...
```

To see reboots from a journalctl perspective
```
[root@server1 ~]# journalctl --list-boots
-1 d843037478c948198af11336312c32a4 Sat 2021-03-13 14:43:42 GMT—Sat 2021-03-13 21:15:21 GMT
 0 56d4be0a3ecb47c7929957c5a75488d2 Sat 2021-03-13 21:15:28 GMT—Sat 2021-03-13 21:15:57 GMT
 ```
 The current file that journal is writing to is file 0, the previous one is -1.
 To view previous journal files, you pass in the index of the file you want
 ```
 [root@server1 ~]# journalctl -b -1
 ```

 ## Introducing SELinux

SELinux is the mandatory access control system for linux.

### View SELinux Modes and Context

To get the current SELinux mode
```
[root@server1 ~]# getenforce
Enforcing
```

To get the SELinux status
```
[root@server1 ~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31
```

With `enforcing` SELinux is checking the rules and applying the rules.  With `permissive`
mode, it's reading the rules but not enforcing them, but it will log that those things
are happening. `disabled` means that no SELinux policy is loaded.

The `Mode from config file` is also enforcing.  The config file is located here
```
[root@server1 ~]# cat /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

If you disable selinux, perform some actions and then re-enable, you have to change
the context of the files to have the correct value.

`setenforce 0` or `setenforce permissive` will change mode to `permissive`.  `setenforce 1`
will set it back to `enforcing`.

To disable SELinux, you need to change the mode in the config file and then reboot the
system for that to take effect.

To get context labels
1) For users (has to be the current user)
```
[root@server1 ~]# id -Z
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

2) For processes
```
[root@server1 ~]# ps -Z
LABEL                             PID TTY          TIME CMD
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 1467 pts/0 00:00:00 bash
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 1724 pts/0 00:00:00 ps
```

3) For files and directories
```
[root@server1 etc]# ls -Z /etc/shadow
----------. root root system_u:object_r:shadow_t:s0    /etc/shadow

[root@server1 etc]# ls -Z /etc/xml
-rw-r--r--. root root system_u:object_r:etc_t:s0       catalog
```

### View SELinux Logs

The `audit.log` file is the log file that SELinux outputs to.
```
[root@server1 log]# cd /var/log/audit/audit.log
type=CRED_DISP msg=audit(1615739401.908:2111): pid=14643 uid=0 auid=0 ses=138 subj=system_u:system_r:crond_t:s0-s0:c0.c1023 msg='op=PAM:setcred grantors=pam_env,pam_unix acct="root" exe="/usr/sbin/crond" hostname=? addr=? terminal=cron res=success'
type=USER_END msg=audit(1615739401.908:2112): pid=14643 uid=0 auid=0 ses=138 subj=system_u:system_r:crond_t:s0-s0:c0.c1023 msg='op=PAM:session_close grantors=pam_loginuid,pam_keyinit,pam_limits,pam_systemd acct="root" exe="/usr/sbin/crond" hostname=? addr=? terminal=cron res=success'
type=SERVICE_START msg=audit(1615739428.020:2113): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=NetworkManager-dispatcher comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=SERVICE_STOP msg=audit(1615739437.845:2114): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=NetworkManager-dispatcher comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=SERVICE_START msg=audit(1615739549.549:2115): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=NetworkManager-dispatcher comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=SERVICE_STOP msg=audit(1615739559.850:2116): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=NetworkManager-dispatcher comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=SERVICE_START msg=audit(1615739684.752:2117): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=NetworkManager-dispatcher comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=SERVICE_STOP msg=audit(1615739694.851:2118): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=NetworkManager-dispatcher comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=SERVICE_START msg=audit(1615739783.656:2119): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=NetworkManager-dispatcher comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
type=SERVICE_STOP msg=audit(1615739793.851:2120): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=NetworkManager-dispatcher comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
```

If we are in permissive mode we log the error but still allow the action to take place.
The command `ausearch -m avc` can be used to search the SELinux logs and see what
actions were done while SELinux was in `permissive` mode.  Adding `ts recent` will
limit entries to the last 10 minutes
```
[root@server1 audit]# ls -Z /etc/shadow
----------. root root system_u:object_r:shadow_t:s0    /etc/shadow
[root@server1 audit]#
[root@server1 audit]# chcon -t unlabeled_t /etc/shadow
[root@server1 audit]# ls -Z /etc/shadow
----------. root root system_u:object_r:unlabeled_t:s0 /etc/shadow
[root@server1 audit]# chcon -t unlabeled_t /etc/shadow
[root@server1 audit]# ausearch -m avc -ts recent
----
time->Sun Mar 14 16:44:53 2021
type=PROCTITLE msg=audit(1615740293.564:2141): proctitle=2F7573722F7362696E2F756E69785F63686B70776400726F6F74006E6F6E756C6C
type=SYSCALL msg=audit(1615740293.564:2141): arch=c000003e syscall=2 success=no exit=-13 a0=7fbad18c0573 a1=80000 a2=1b6 a3=24 items=0 ppid=14831 pid=14856 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="unix_chkpwd" exe="/usr/sbin/unix_chkpwd" subj=system_u:system_r:chkpwd_t:s0-s0:c0.c1023 key=(null)
type=AVC msg=audit(1615740293.564:2141): avc:  denied  { read } for  pid=14856 comm="unix_chkpwd" name="shadow" dev="dm-0" ino=5155781 scontext=system_u:system_r:chkpwd_t:s0-s0:c0.c1023 tcontext=system_u:object_r:unlabeled_t:s0 tclass=file permissive=0
```
In the above example we are checking the context of the `/etc/shadow file`, and it
currently has the correct type.  If we change it's context type to `unlabeled_t`
using `chcon` and then try and log in we will see an entry in the `audit.log` file.

To restore the correct context info for the `/etc/shadaow` file use the following.  SELinux
has a master file of correct contexts that should be set, so it can restore from that
```
[root@server1 audit]# restorecon /etc/shadow
[root@server1 audit]# ls -Z /etc/shadow
----------. root root system_u:object_r:shadow_t:s0    /etc/shadow
```

To see the master list of contexts for the system files run `semanage fcontext -l`
That output is very large so we can grep for what we are intested in, in this case
the `shadow` file
```
[root@server1 audit]# semanage fcontext -l | grep shadow
/etc/shadow.*                                      regular file       system_u:object_r:shadow_t:s0
/etc/gshadow.*                                     regular file       system_u:object_r:shadow_t:s0
/etc/nshadow.*                                     regular file       system_u:object_r:shadow_t:s0
/var/db/shadow.*                                   regular file       system_u:object_r:shadow_t:s0
/etc/security/opasswd                              regular file       system_u:object_r:shadow_t:s0
/etc/security/opasswd\.old                         regular file       system_u:object_r:shadow_t:s0
```

The file context information is actually held in files that hold default context info for files and
directories.  The location is
```
[root@server1 files]# cd /etc/selinux/targeted/contexts/files
[root@server1 files]# ls
file_contexts  file_contexts.bin  file_contexts.homedirs  file_contexts.homedirs.bin  file_contexts.local  file_contexts.local.bin  file_contexts.subs  file_contexts.subs_dist  media
```

### Working with SELinux Booleans and Ports

You can adjust SELinux policies by using the SELinux booleans.  Booleans allow policies to the
changed using simple "on" and "off" settings.

To list the current list of SELinux booleans in teh system
```
[root@server1 files]# getsebool -a
abrt_anon_write --> off
abrt_handle_event --> off
abrt_upload_watch_anon_write --> on
antivirus_can_scan_system --> off
antivirus_use_jit --> off
auditadm_exec_content --> on
authlogin_nsswitch_use_ldap --> off
authlogin_radius --> off
authlogin_yubikey --> off
awstats_purge_apache_log_files --> off
boinc_execmem --> on
cdrecord_read_content --> off
cluster_can_network_connect --> off
cluster_manage_all_files --> off
cluster_use_execmem --> off
cobbler_anon_write --> off
...
```

To get more information regarding each boolean use the semanage command.  This gives
a list of booleans, it's current state, the default state and the description for
the boolean
```
[root@server1 files]# semanage boolean -l
SELinux boolean                State  Default Description

privoxy_connect_any            (on   ,   on)  Allow privoxy to connect any
smartmon_3ware                 (off  ,  off)  Allow smartmon to 3ware
mpd_enable_homedirs            (off  ,  off)  Allow mpd to enable homedirs
xdm_sysadm_login               (off  ,  off)  Allow xdm to sysadm login
xen_use_nfs                    (off  ,  off)  Allow xen to use nfs
mozilla_read_content           (off  ,  off)  Allow mozilla to read content
ssh_chroot_rw_homedirs         (off  ,  off)  Allow ssh to chroot rw homedirs
mount_anyfile                  (on   ,   on)  Allow mount to anyfile
cron_userdomain_transition     (on   ,   on)  Allow cron to userdomain transition
xdm_write_home                 (off  ,  off)  Allow xdm to write home
openvpn_can_network_connect    (on   ,   on)  Allow openvpn to can network connect
...
```

To see the status of a particular se boolean
```
[root@server1 files]# getsebool httpd_read_user_content
httpd_read_user_content --> off
```

To turn a boolean "on", in this case the httpd_read_user_content boolean.  However
this will only persist for the current session, if you reboot the boolean will be
set back to it's previous state.
```
[root@server1 files]# setsebool !$ on
setsebool httpd_read_user_content on
[root@server1 files]# getsebool httpd_read_user_content
httpd_read_user_content --> on
```

However the above command will only persist for the current session, if you reboot
the boolean will be set back to it's previous state.  To permanently set the boolean
use the following
```
[root@server1 files]# setsebool -P httpd_read_user_content on
```

To get a list of SELinux ports types
```
[root@server1 files]# semanage port -l
SELinux Port Type              Proto    Port Number

afs3_callback_port_t           tcp      7001
afs3_callback_port_t           udp      7001
afs_bos_port_t                 udp      7007
afs_fs_port_t                  tcp      2040
afs_fs_port_t                  udp      7000, 7005
afs_ka_port_t                  udp      7004
afs_pt_port_t                  tcp      7002
afs_pt_port_t                  udp      7002
afs_vl_port_t                  udp      7003
agentx_port_t                  tcp      705
agentx_port_t                  udp      705
amanda_port_t                  tcp      10080-10083
amanda_port_t                  udp      10080-10082
...
```

To see the SELinux port info for `ssh`.
```
[root@server1 files]# semanage port -l | grep ssh
ssh_port_t                     tcp      22
```

To change the SELinux port setting.  If we reconfigured SSH to use a new port then
we would also need to change the SELinux port policy to ensure that it would allow
traffic over the new port
```
[root@server1 files]# semanage port -a -t ssh_port_t -p tcp 2222
```

## Managing Software on Centos7

### Software Management with RPM

RPM is the basic package manager tool used in Redhat (RHEL) and Centos systems.

To install software using `yum`.  For this example when we are prompted to install
or download it, we will download it. This step is required in order to demo how
we can use the `rpm` command on downloaded packages later
```
[root@server1 ~]# yum install httpd
```

To see the httpd packages in the yum cache (that is where they are downloaded to).
```
[root@server1 ~]# tree /var/cache/yum
/var/cache/yum
└── x86_64
    └── 7
        ├── base
        │   ├── 6d0c3a488c282fe537794b5946b01e28c7f44db79097bb06826e1c0c88bad5ef-primary.sqlite.bz2
        │   ├── a4e2b46586aa556c3b6f814dad5b16db5a669984d66b68e873586cd7c7253301-c7-x86_64-comps.xml.gz
        │   ├── cachecookie
        │   ├── d6d94c7d406fe7ad4902a97104b39a0d8299451832a97f31d71653ba982c955b-filelists.sqlite.bz2
        │   ├── gen
        │   │   ├── comps.xml
        │   │   ├── filelists_db.sqlite
        │   │   └── primary_db.sqlite
        │   ├── mirrorlist.txt
        │   ├── packages
        │   │   └── mailcap-2.1.41-2.el7.noarch.rpm
        │   └── repomd.xml
        ├── epel
        │   ├── 02940900d5b6b6d28608cec00729e979a5523fdc67996ba6e6e2c33d5acfd32c-primary.sqlite.bz2
        │   ├── 16395c89edfbd0e0915c06ef2df4573aea1c42b8e27e7e18b32329ee4fb3b2cb-comps-Everything.x86_64.xml.gz
        │   ├── 1f894bb4020fd90db9a87134fd22465f8a3c1ead4ac85c32afbf222142289a3f-updateinfo.xml.bz2
        │   ├── cachecookie
        │   ├── gen
        │   │   ├── comps.xml
        │   │   ├── filelists_db.sqlite
        │   │   └── primary_db.sqlite
        │   ├── metalink.xml
        │   ├── packages
        │   └── repomd.xml
        ├── extras
        │   ├── cachecookie
        │   ├── f566f80ea90a80473260b93311d51f669dedd898c7a25836b121b0e44fdc9e75-primary.sqlite.bz2
        │   ├── gen
        │   │   └── primary_db.sqlite
        │   ├── mirrorlist.txt
        │   ├── packages
        │   └── repomd.xml
        ├── timedhosts
        ├── timedhosts.txt
        └── updates
            ├── 7ef1bb9522c9f351c89eca2ff7335901b182519e3080d06341ffe5cc75a5f53e-primary.sqlite.bz2
            ├── cachecookie
            ├── gen
            │   ├── filelists_db.sqlite
            │   └── primary_db.sqlite
            ├── mirrorlist.txt
            ├── packages
            │   ├── httpd-2.4.6-97.el7.centos.x86_64.rpm
            │   └── httpd-tools-2.4.6-97.el7.centos.x86_64.rpm
            └── repomd.xml
```

To list all the rpm packages currently installed
```
[root@server1 packages]# rpm -qa
python-ntplib-0.3.2-1.el7.noarch
openssh-server-7.4p1-21.el7.x86_64
setroubleshoot-server-3.2.30-8.el7.x86_64
libvorbis-1.3.3-8.el7.1.x86_64
ncurses-base-5.9-14.20130511.el7_4.noarch
sgpio-1.2.0.10-13.el7.x86_64
gnome-abrt-0.3.4-9.el7.x86_64
libtdb-1.3.18-1.el7.x86_64
libstdc++-4.8.5-44.el7.x86_64
python-nss-0.16.0-3.el7.x86_64
libwacom-0.30-1.el7.x86_64
libwayland-client-1.15.0-1.el7.x86_64
...
```

To get info on a particular package that is installed
```
[root@server1 packages]# rpm -qi nmap
Name        : nmap
Epoch       : 2
Version     : 6.40
Release     : 19.el7
Architecture: x86_64
Install Date: Sun 14 Mar 2021 21:09:12 GMT
Group       : Applications/System
Size        : 16916966
License     : GPLv2 and LGPLv2+ and GPLv2+ and BSD
Signature   : RSA/SHA256, Thu 22 Aug 2019 22:36:20 IST, Key ID 24c6a8a7f4a80eb5
Source RPM  : nmap-6.40-19.el7.src.rpm
Build Date  : Fri 09 Aug 2019 02:16:48 IST
Build Host  : x86-01.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://nmap.org/
Summary     : Network exploration tool and security scanner
Description :
Nmap is a utility for network exploration or security auditing.  It supports
ping scanning (determine which hosts are up), many port scanning techniques
(determine what services the hosts are offering), and TCP/IP fingerprinting
(remote host operating system identification). Nmap also offers flexible target
and port specification, decoy scanning, determination of TCP sequence
predictability characteristics, reverse-identd scanning, and more. In addition
to the classic command-line nmap executable, the Nmap suite includes a flexible
data transfer, redirection, and debugging tool (netcat utility ncat), a utility
for comparing scan results (ndiff), and a packet generation and response analysis
tool (nping).
```

To list the contents of a package
```
root@server1 packages]# rpm -ql nmap
/usr/bin/ndiff
/usr/bin/nmap
/usr/bin/nping
/usr/share/doc/nmap-6.40
/usr/share/doc/nmap-6.40/COPYING
/usr/share/doc/nmap-6.40/README
/usr/share/doc/nmap-6.40/nmap.usage.txt
/usr/share/man/de/man1/nmap.1.gz
/usr/share/man/es/man1/nmap.1.gz
/usr/share/man/fr/man1/nmap.1.gz
...
```

To get info for a package that has not yet been installed, and to get a list of
what is included in ot
```
[root@server1 packages]# rpm -qpi httpd-2.4.6-97.el7.centos.x86_64.rpm
[root@server1 packages]# rpm -qpi httpd-2.4.6-97.el7.centos.x86_64.rpm
Name        : httpd
Version     : 2.4.6
Release     : 97.el7.centos
Architecture: x86_64
Install Date: (not installed)
Group       : System Environment/Daemons
Size        : 9821064
License     : ASL 2.0
Signature   : RSA/SHA256, Wed 18 Nov 2020 14:17:43 GMT, Key ID 24c6a8a7f4a80eb5
Source RPM  : httpd-2.4.6-97.el7.centos.src.rpm
Build Date  : Mon 16 Nov 2020 16:21:17 GMT
Build Host  : x86-02.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://httpd.apache.org/
Summary     : Apache HTTP Server
Description :
The Apache HTTP Server is a powerful, efficient, and extensible
web server.

[root@server1 packages]# rpm -qpl httpd-2.4.6-97.el7.centos.x86_64.rpm
[root@server1 packages]# rpm -qpl httpd-2.4.6-97.el7.centos.x86_64.rpm
/etc/httpd
/etc/httpd/conf
/etc/httpd/conf.d
/etc/httpd/conf.d/README
/etc/httpd/conf.d/autoindex.conf
/etc/httpd/conf.d/userdir.conf
/etc/httpd/conf.d/welcome.conf
/etc/httpd/conf.modules.d
/etc/httpd/conf.modules.d/00-base.conf
/etc/httpd/conf.modules.d/00-dav.conf
/etc/httpd/conf.modules.d/00-lua.conf
/etc/httpd/conf.modules.d/00-mpm.conf
/etc/httpd/conf.modules.d/00-proxy.conf
/etc/httpd/conf.modules.d/00-systemd.conf
/etc/httpd/conf.modules.d/01-cgi.conf
/etc/httpd/conf/httpd.conf
/etc/httpd/conf/magic
/etc/httpd/logs
/etc/httpd/modules
...
```

If we try to install the package directly using `rpm -i` then we see some potential
issues with installing each package individually.  In the below case we see that
we get errors on install because there are missing dependencies
```
[root@server1 packages]# rpm -i httpd-2.4.6-97.el7.centos.x86_64.rpm
error: Failed dependencies:
	/etc/mime.types is needed by httpd-2.4.6-97.el7.centos.x86_64
	httpd-tools = 2.4.6-97.el7.centos is needed by httpd-2.4.6-97.el7.centos.x86_64
```

To remove an installed package
```
[root@server1 packages]# rpm -e nmap
```

To see which package a file belongs to.  Once you have a package name you can then
get more info about it using `rpm -qi`
```
[root@server1 packages]# rpm -qf /etc/hosts
setup-2.8.71-11.el7.noarch

[root@server1 packages]# rpm -qi setup
Name        : setup
Version     : 2.8.71
Release     : 11.el7
Architecture: noarch
Install Date: Sat 27 Feb 2021 15:41:49 GMT
Group       : System Environment/Base
Size        : 697141
License     : Public Domain
Signature   : RSA/SHA256, Fri 03 Apr 2020 22:21:58 IST, Key ID 24c6a8a7f4a80eb5
Source RPM  : setup-2.8.71-11.el7.src.rpm
Build Date  : Wed 01 Apr 2020 05:29:34 IST
Build Host  : x86-02.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : https://pagure.io/setup/
Summary     : A set of system configuration and setup files
Description :
The setup package contains a set of important system configuration and
setup files, such as passwd, group, and profile.
```

To verify a package run the following.  If nothing is output it means the package
is correct and nothing has changed.  If any files have been changed then the command
will show you what files
```
[root@server1 packages]# rpm -V setup
```

### Using YUM

Yum is a package manager that removes a lot of the potential issues with making sure
you are installing all the dependencies required for a package to be installed
successfully.

To install a package.  You just need to specify the package name, you don't need
a full path. Yum will look up it's local cache and will be able to look up the
package based just on the name. The second command is to agree to install upfront,
without it prompting you to enter "yes"
```
[root@server1 packages]# yum install bash-completion
[root@server1 packages]# yum install -y bash-completion
```

**Useful Aside!** : The bash-completion package is very useful. You can auto-complete on
commands by hitting tab, and if you hit tab-tab after a command it will show you
the available sub-commands and options available.

To use yum to get package info
```
[root@server1 ~]# yum info bash-completion.noarch
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.strencom.net
 * epel: fedora.cu.be
 * extras: mirror.strencom.net
 * updates: mirror.strencom.net
Installed Packages
Name        : bash-completion
Arch        : noarch
Epoch       : 1
Version     : 2.1
Release     : 8.el7
Size        : 263 k
Repo        : installed
From repo   : base
Summary     : Programmable completion for Bash
URL         : http://bash-completion.alioth.debian.org/
License     : GPLv2+
Description : bash-completion is a collection of shell functions that take advantage
            : of the programmable completion feature of bash.
```

To get the yum version
```
[root@server1 ~]# yum version
Loaded plugins: fastestmirror
Installed: 7/x86_64                                                                                                                                                                       1148:1bd7dda4d476fea3c26ed509e39b81c07e9f38d1
Group-Installed: yum                                                                                                                                                                        14:7894245620e388e6cc2b147e584eb2b9511d7ca8
version
```

To remove a package
```
[root@server1 ~]# yum remove nmap
[root@server1 ~]# yum remove -y nmap
```

To set the dependency list for a package
```
[root@server1 ~]# yum deplist nmap
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.strencom.net
 * epel: fedora.cu.be
 * extras: mirror.strencom.net
 * updates: mirror.strencom.net
package: nmap.x86_64 2:6.40-19.el7
  dependency: /usr/bin/python
   provider: python.x86_64 2.7.5-90.el7
  dependency: libc.so.6(GLIBC_2.15)(64bit)
   provider: glibc.x86_64 2.17-323.el7_9
  dependency: libcrypto.so.10()(64bit)
   provider: openssl-libs.x86_64 1:1.0.2k-21.el7_9
  dependency: libcrypto.so.10(libcrypto.so.10)(64bit)
   provider: openssl-libs.x86_64 1:1.0.2k-21.el7_9
  dependency: libdl.so.2()(64bit)
   provider: glibc.x86_64 2.17-323.el7_9
  dependency: libdl.so.2(GLIBC_2.2.5)(64bit)
...
```

To get a list of installed packages.  This will show the yum repo they were installed
from
```
[root@server1 ~]# yum list installed
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.strencom.net
 * epel: fedora.cu.be
 * extras: mirror.strencom.net
 * updates: mirror.strencom.net
Installed Packages
GConf2.x86_64                                                                                                       3.2.6-8.el7                                                                                               @base
ImageMagick.x86_64                                                                                                  6.9.10.68-5.el7_9                                                                                         @updates
ModemManager-glib.x86_64                                                                                            1.6.10-4.el7                                                                                              @base
NetworkManager.x86_64                                                                                               1:1.18.8-2.el7_9                                                                                          @updates
NetworkManager-glib.x86_64                                                                                          1:1.18.8-2.el7_9                                                                                          @updates
...
```

To get a list of all available packages
```
[root@server1 ~]# yum list installed
```

### Configuring YUM Repositories

The yum repo files are located locally at `/etc/yum.repos.d`

The anatomy of a yum repo file
```
[root@server1 yum.repos.d]# less CentOS-Base.repo

# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```
As you can see from the above there are 4 repos contained in this repo file, but
one of them, `centosplus` is disabled (`enabled=0`)

To list the repositories
```
[root@server1 yum.repos.d]# yum repolist
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.strencom.net
 * epel: fedora.cu.be
 * extras: mirror.strencom.net
 * updates: mirror.strencom.net
repo id                                                                                           repo name                                                                                                                      status
!base/7/x86_64                                                                                    CentOS-7 - Base                                                                                                                10,072
!epel/x86_64                                                                                      Extra Packages for Enterprise Linux 7 - x86_64                                                                                 13,552
!extras/7/x86_64                                                                                  CentOS-7 - Extras                                                                                                                 453
!updates/7/x86_64                                                                                 CentOS-7 - Updates                                                                                                              1,729
repolist: 25,806
```

To see ALL yum repos, enabled and disabled
```
[root@server1 yum.repos.d]# yum repolist all
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.strencom.net
 * epel: fedora.cu.be
 * extras: mirror.strencom.net
 * updates: mirror.strencom.net
repo id                                                                                      repo name                                                                                                                  status
C7.0.1406-base/x86_64                                                                        CentOS-7.0.1406 - Base                                                                                                     disabled
C7.0.1406-centosplus/x86_64                                                                  CentOS-7.0.1406 - CentOSPlus                                                                                               disabled
C7.0.1406-extras/x86_64                                                                      CentOS-7.0.1406 - Extras                                                                                                   disabled
C7.0.1406-fasttrack/x86_64                                                                   CentOS-7.0.1406 - Fasttrack                                                                                                disabled
C7.0.1406-updates/x86_64                                                                     CentOS-7.0.1406 - Updates                                                                                                  disabled
C7.1.1503-base/x86_64                                                                        CentOS-7.1.1503 - Base                                                                                                     disabled
C7.1.1503-centosplus/x86_64                                                                  CentOS-7.1.1503 - CentOSPlus                                                                                               disabled
C7.1.1503-extras/x86_64                                                                      CentOS-7.1.1503 - Extras                                                                                                   disabled
C7.1.1503-fasttrack/x86_64                                                                   CentOS-7.1.1503 - Fasttrack                                                                                                disabled
C7.1.1503-updates/x86_64                                                                     CentOS-7.1.1503 - Updates                                                                                                  disabled
C7.2.1511-base/x86_64                                                                        CentOS-7.2.1511 - Base                                                                                                     disabled
C7.2.1511-centosplus/x86_64                                                                  CentOS-7.2.1511 - CentOSPlus                                                                                               disabled
C7.2.1511-extras/x86_64                                                                      CentOS-7.2.1511 - Extras                                                                                                   disabled
C7.2.1511-fasttrack/x86_64                                                                   CentOS-7.2.1511 - Fasttrack                                                                                                disabled
C7.2.1511-updates/x86_64                                                                     CentOS-7.2.1511 - Updates                                                                                                  disabled
C7.3.1611-base/x86_64                                                                        CentOS-7.3.1611 - Base                                                                                                     disabled
...
```

To make your own repo file in the `/etc/yum.repos.d` directory
```
[root@server1 yum.repos.d]# vi local.repo

[CentOS7]
name=CentOS 7.2 Local Network
baseurk=http://192.168.56.220/centos7/
enabled=1
gpgcheck=0
```

After you have created your repo file you can see it when you run `yum repolist`
```
[root@server1 yum.repos.d]# yum repolist
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Loading mirror speeds from cached hostfile
Loading mirror speeds from cached hostfile
epel/x86_64/metalink                                                                                                                                                                                            |  12 kB  00:00:00
epel/x86_64                                                                                                                                                                                                     | 4.7 kB  00:00:00
epel/x86_64/updateinfo                                                                                                                                                                                          | 1.0 MB  00:00:00
epel/x86_64/primary_db                                                                                                                                                                                          | 6.9 MB  00:00:01
repo id                                                                                        repo name                                                                                                                         status
CentOS7                                                                                        CentOS 7.2 Local Network                                                                                                          0
!epel/x86_64                                                                                   Extra Packages for Enterprise Linux 7 - x86_64                                                                                    0
repolist: 0
```

### Working with the YUM Cache

The YUM cache contains metadata of all yum packages in a local cache.  When you install
a package using `yum install`, it will get package information from the local yum cache
rather than go to an online resource. So it should be quicker.

The yum cache is located in `/var/yum/cache`
```
[root@server1 yum.repos.d]# tree /var/cache/yum
/var/cache/yum
└── x86_64
    └── 7
        ├── base
        │   ├── 6d0c3a488c282fe537794b5946b01e28c7f44db79097bb06826e1c0c88bad5ef-primary.sqlite.bz2
        │   ├── a4e2b46586aa556c3b6f814dad5b16db5a669984d66b68e873586cd7c7253301-c7-x86_64-comps.xml.gz
        │   ├── cachecookie
        │   ├── d6d94c7d406fe7ad4902a97104b39a0d8299451832a97f31d71653ba982c955b-filelists.sqlite.bz2
        │   ├── gen
        │   │   ├── comps.xml
        │   │   ├── filelists_db.sqlite
        │   │   └── primary_db.sqlite
        │   ├── mirrorlist.txt
        │   ├── packages
        │   └── repomd.xml
        ├── base-debuginfo
        │   ├── gen
        │   └── packages
        ├── base-source
        │   ├── gen
        │   └── packages
        ├── C7.0.1406-base
        │   ├── gen
        │   └── packages
        ├── C7.0.1406-centosplus
        │   ├── gen
...
```

To build metadate from our repositories and make sure we have the latest repo data
there
```
[root@server1 yum.repos.d]# yum makecache
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
epel/x86_64/metalink                                                                                                                                                                                            |  12 kB  00:00:00
 * base: mirror.strencom.net
 * epel: mirror.seas.harvard.edu
 * extras: mirror.strencom.net
 * updates: mirror.strencom.net
base                                                                                                                                                                                                            | 3.6 kB  00:00:00
extras                                                                                                                                                                                                          | 2.9 kB  00:00:00
updates                                                                                                                                                                                                         | 2.9 kB  00:00:00
(1/8): epel/x86_64/prestodelta                                                                                                                                                                                  |  616 B  00:00:00
(2/8): extras/7/x86_64/other_db                                                                                                                                                                                 | 134 kB  00:00:01
(3/8): extras/7/x86_64/filelists_db                                                                                                                                                                             | 226 kB  00:00:02
(4/8): epel/x86_64/other_db                                                                                                                                                                                     | 3.3 MB  00:00:02
(5/8): updates/7/x86_64/other_db                                                                                                                                                                                | 476 kB  00:00:02
(6/8): base/7/x86_64/other_db                                                                                                                                                                                   | 2.6 MB  00:00:05
(7/8): updates/7/x86_64/filelists_db                                                                                                                                                                            | 3.4 MB  00:00:07
(8/8): epel/x86_64/filelists_db                                                                                                                                                                                 |  12 MB  00:00:11
Metadata Cache Created
```

To remove any packages you have downloaded (pending installation) and the cache data
```
[root@server1 yum.repos.d]# yum clean all
Loaded plugins: fastestmirror
Cleaning repos: base epel extras updates
Cleaning up list of fastest mirrors
```
You can rebuild your cache again using `yum makecache`

PRO Tip : If you want to make virtual machine images from your system, it is a good
idea to clear out your yum cache to save space, and then when you have spun up your
VM you can rebuild your cache then.

### Controlling Kernel Updates

It is simple to update your kernal using `yum`, the command is simple `yum update kernel`.
However one thing to remember when updating your kernel is that the existing kernel is
not touched.  yum will install a new kernal alongside the existing ones.  When you boot
it will boot to the new kernel from that point. That might break your virtualization
environment.

However if you want to exclude any kernel updates when running `yum update` you can
add an entry to the `yum.conf` file
```
[root@server1 yum.repos.d]# vi /etc/yum.conf

// Add in the following entry
exclude=kernel*
```
You can also do the above by running `yum --exclude=kernel* update`

### Working with Source RPMS

As an alternative to installing pre-built and pre-compiled packages, you can also download
the source for packages, if you need to change them for some reason. The steps for
installing the source repositories and building the source packages are as follows;

1) If you run a yum repolist all you will see that the source repositories for Centos7
are disabled.
```
[root@server1 yum.repos.d]# yum repolist all
centosplus/7/x86_64                                                                          CentOS-7 - Plus                                                                                                            disabled
centosplus-source/7                                                                          CentOS-7 - Plus Sources                                                                                                    disabled
cr/7/x86_64                                                                                  CentOS-7 - cr                                                                                                              disabled
epel/x86_64                                                                                  Extra Packages for Enterprise Linux 7 - x86_64                                                                             enabled: 13,552
```
These repos need to be enabled before we can install the src packages.

2) If we grep the CentOS-Sources.repo file and check the enabled field, we can
see that file does contain repo information but all of them are disabled.
```
[root@server1 yum.repos.d]# grep ^enabled CentOS-Sources.repo
enabled=0
enabled=0
enabled=0
enabled=0
```

3) To enable the repos we need to do a search and replace on the CentOS-Sources.repo
file
```
[root@server1 yum.repos.d]# sed -i 's/^enabled=0/enabled=1/' CentOS-Sources.repo
[root@server1 yum.repos.d]# grep ^enabled CentOS-Sources.repo
enabled=1
enabled=1
enabled=1
enabled=1
```

4) Then when we do a yum repolist we can see the source repos in our list of enabled repos.
```
[root@server1 yum.repos.d]# yum repolist
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.facebook.net
 * epel: mirror.dst.ca
 * extras: centos.mirror.constant.com
 * updates: mirrors.seas.harvard.edu
base-source                                                                                                                                                                                                     | 2.9 kB  00:00:00
centosplus-source                                                                                                                                                                                               | 2.9 kB  00:00:00
extras-source                                                                                                                                                                                                   | 2.9 kB  00:00:00
updates-source                                                                                                                                                                                                  | 2.9 kB  00:00:00
(1/4): updates-source/7/primary_db                                                                                                                                                                              | 106 kB  00:00:00
(2/4): centosplus-source/7/primary_db                                                                                                                                                                           |  11 kB  00:00:02
(3/4): base-source/7/primary_db                                                                                                                                                                                 | 974 kB  00:00:03
(4/4): extras-source/7/primary_db                                                                                                                                                                               |  27 kB  00:00:04
repo id                                                                                            repo name                                                                                                                     status
base/7/x86_64                                                                                      CentOS-7 - Base                                                                                                               10,072
base-source/7                                                                                      CentOS-7 - Base Sources                                                                                                            0
centosplus-source/7                                                                                CentOS-7 - Plus Sources                                                                                                            0
epel/x86_64                                                                                        Extra Packages for Enterprise Linux 7 - x86_64                                                                                13,552
extras/7/x86_64                                                                                    CentOS-7 - Extras                                                                                                                453
extras-source/7                                                                                    CentOS-7 - Extras Sources                                                                                                          0
updates/7/x86_64                                                                                   CentOS-7 - Updates                                                                                                             1,729
updates-source/7                                                                                   CentOS-7 - Updates Sources                                                                                                         0
repolist: 25,806
```

5) For the next Step we need to ensure that the yum-utils package is installed
```
[root@server1 yum.repos.d]# yum install -y yum-utils
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.facebook.net
 * epel: mirror.dst.ca
 * extras: centos.mirror.constant.com
 * updates: mirrors.seas.harvard.edu
Package yum-utils-1.1.31-54.el7_8.noarch already installed and latest version
```

6) Now we can download the source for the zsh shell
```
[root@server1 yum.repos.d]# yumdownloader --source zsh
Loaded plugins: fastestmirror
Enabling epel-source repository
Loading mirror speeds from cached hostfile
epel-source/x86_64/metalink                                                                                                                                                                                     |  23 kB  00:00:00
 * base: mirror.facebook.net
 * epel: mirror.dst.ca
 * epel-source: mirrors.ukfast.co.uk
 * extras: centos.mirror.constant.com
 * updates: mirrors.seas.harvard.edu
epel-source                                                                                                                                                                                                     | 3.5 kB  00:00:00
(1/2): epel-source/x86_64/updateinfo                                                                                                                                                                            | 1.0 MB  00:00:00
(2/2): epel-source/x86_64/primary_db                                                                                                                                                                            | 2.4 MB  00:00:01
zsh-5.0.2-34.el7_8.2.src.rpm

[root@server1 yum.repos.d]# ls
CentOS-Base.repo  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-fasttrack.repo  CentOS-Media.repo  CentOS-Sources.repo  CentOS-Vault.repo  CentOS-x86_64-kernel.repo  epel.repo  epel-testing.repo  zsh-5.0.2-34.el7_8.2.src.rpm
```

7) To help us make the source files we need to have the `ncurses-devel` package
```
[root@server1 yum.repos.d]# yum install ncurses-devel
```

8) We can install the sources package using the `rpm -i` command.  This will create
a directory call `rpmbuild/SOURCES` in our home directoy
```
[root@server1 yum.repos.d]# rpm -i zsh-5.0.2-34.el7_8.2.src.rpm
arning: user mockbuild does not exist - using root
warning: group mockbuild does not exist - using root
warning: user mockbuild does not exist - using root
warning: group mockbuild does not exist - using root
...

[root@server1 yum.repos.d]# cd ~/rpmbuild/SOURCES/
```

9) One of the files in the SOURCES folder is the `zsh-5.0.2.tar.bz2` file. We need
to untar it to work with it
```
-rw-rw-r--. 1 root root 3025767 Mar 31  2020 zsh-5.0.2.tar.bz2

[root@server1 SOURCES]# tar -xjf zsh-5.0.2.tar.bz2
```

10) A directory called `zsh-5.0.2/` will be extracted from the tar file, we need to
cd to that directoy and run the `configure` script there in order to create a `make file`
```
[root@server1 SOURCES]# cd zsh-5.0.2/
[root@server1 zsh-5.0.2]# ls
aclocal.m4  ChangeLog   Config        config.h.in  configure     Doc  FEATURES   INSTALL     LICENCE   Makefile.in  Misc           NEWS    Scripts  stamp-h.in    Test
aczsh.m4    Completion  config.guess  config.sub   configure.ac  Etc  Functions  install-sh  MACHINES  META-FAQ     mkinstalldirs  README  Src      StartupFiles  Util
[root@server1 zsh-5.0.2]# ./configure
configuring for zsh 5.0.2
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables...
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
...
```

11) This uses the `ncurses-devel` package to generate the make file.  We can now run
the `make` script and then `make install` and that will compile the sources for us
and put the `zsh` command on the path
```
[root@server1 zsh-5.0.2]# make
...
...
[root@server1 zsh-5.0.2]# make install
...
...

[root@server1]~/rpmbuild/SOURCES/zsh-5.0.2# which zsh
/usr/local/bin/zsh
```

### Enabling Services

If you are installing services such as a web server or samba service, you also
need to enable the service so that it starts up automatically on reboot.

For this demo we are going to use the httpd service.  In order to see if the correct package is
installed (which it is)
```
[root@server1 ~]# yum install httpd
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.facebook.net
 * epel: mirror.dst.ca
 * extras: centos.mirror.constant.com
 * updates: mirrors.seas.harvard.edu
Package httpd-2.4.6-97.el7.centos.x86_64 already installed and latest version
Nothing to do
```

Then we will use `nmap` to see if the http ports are listening (they are not)
```
[root@server1 ~]# nmap localhost

Starting Nmap 6.40 ( http://nmap.org ) at 2021-03-15 20:35 GMT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000015s latency).
Other addresses for localhost (not scanned): 127.0.0.1
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp

Nmap done: 1 IP address (1 host up) scanned in 1.60 seconds
[root@server1 ~]# nmap localhost

Starting Nmap 6.40 ( http://nmap.org ) at 2021-03-15 20:37 GMT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000012s latency).
Other addresses for localhost (not scanned): 127.0.0.1
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp

Nmap done: 1 IP address (1 host up) scanned in 1.60 seconds
```

Let's check the status of the httpd service to see if it's up and running and if
the service is enabled (it's not)
```
[root@server1 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
```

To enable the `httpd` service.  When we check the status we see it's still not running
but the service status is now `enabled`
```
[root@server1 ~]# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.

[root@server1 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
```

To start the `httpd` service
```
[root@server1 ~]# systemctl start httpd
[root@server1 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2021-03-15 20:42:04 GMT; 4s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 14365 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/httpd.service
           ├─14365 /usr/sbin/httpd -DFOREGROUND
           ├─14366 /usr/sbin/httpd -DFOREGROUND
           ├─14367 /usr/sbin/httpd -DFOREGROUND
           ├─14368 /usr/sbin/httpd -DFOREGROUND
           ├─14369 /usr/sbin/httpd -DFOREGROUND
           └─14370 /usr/sbin/httpd -DFOREGROUND

Mar 15 20:41:31 server1.example.com systemd[1]: Starting The Apache HTTP Server...
Mar 15 20:42:04 server1.example.com systemd[1]: Started The Apache HTTP Server.
```

So now when we run `nmap` on localhost again we can see that we are listening on the
http port now
```
[root@server1 ~]# nmap localhost

Starting Nmap 6.40 ( http://nmap.org ) at 2021-03-15 20:43 GMT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000090s latency).
Other addresses for localhost (not scanned): 127.0.0.1
Not shown: 997 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.60 seconds
```

If you want to employ a shortcut and enable and start a service at the same time
you can use the following command instead.  This enables the service and starts it
immediately.
```
[root@server1 ~]# systemctl enable httpd --now
```

You can stop a service using `systemctl stop httpd`.  You can disable a service using
`systemctl disable httpd`.  `systemctl disable httpd --now` will do both in one command

## Configuration Management Tools

### Installing puppet

`Puppet` is a ruby based configuration management tool like `chef`.  To install it run the following.
It comes from the `epel` yum repository, so you need to have the `epel-release` package installed first
We can do a version check after installation to ensure it is installed correctly
```
[root@server1 yum.repos.d]# yum install -y puppet
...
...
[root@server1 yum.repos.d]# puppet --version
3.6.2
```

### Checking the Hostname and Domain

Because CentOS7 is systemd based, we can use `hostnamectl`.  We can see from the output
that we have a FQDN as well as just a hostname.
```
[root@server1 yum.repos.d]# hostnamectl
   Static hostname: server1.example.com
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 66213c250ded4576a9120b120ad72e6c
           Boot ID: 56d4be0a3ecb47c7929957c5a75488d2
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1160.15.2.el7.x86_64
      Architecture: x86-64
```

One of the tools that is installed with `Puppet` is `facter` which when run will
output a lot system information such as hardware details, network settings, kernel/OS
information etc. Facter was create for Puppet to gather system information, but can
also be used as a standalone command
```
[root@server1 yum.repos.d]# facter
architecture => x86_64
augeasversion => 1.4.0
bios_release_date => 12/01/2006
bios_vendor => innotek GmbH
bios_version => VirtualBox
blockdevice_sda_model => VBOX HARDDISK
blockdevice_sda_size => 8589934592
blockdevice_sda_vendor => ATA
blockdevice_sr0_model => CD-ROM
blockdevice_sr0_size => 1073741312
blockdevice_sr0_vendor => VBOX
blockdevices => sda,sr0
boardmanufacturer => Oracle Corporation
boardproductname => VirtualBox
boardserialnumber => 0
dhcp_servers => {"system"=>"10.0.2.3", "enp0s3"=>"10.0.2.3", "enp0s8"=>"192.168.99.10"}
domain => example.com
facterversion => 2.4.1
filesystems => xfs
fqdn => server1.example.com
gid => root
hardwareisa => x86_64
hardwaremodel => x86_64
hostname => server1
...
...
...
```

You can `grep` `facter` to get all sorts of info
```
[root@server1 yum.repos.d]# facter | grep hostname
hostname => server1
[root@server1 yum.repos.d]# facter | grep domain
domain => example.com
[root@server1 yum.repos.d]# facter | grep kernel
kernel => Linux
kernelmajversion => 3.10
kernelrelease => 3.10.0-1160.15.2.el7.x86_64
kernelversion => 3.10.0
```

### Puppet Manifests
`Puppet` has "manifests", these are like a shipping list of what needs to be created
on a node or a group of nodes. Manifests are `.pp` files.  Normally we would be
configuring manifest files on a central puppet server.  But for the purposes of
this demo we will create them locally and run them using the puppet client.
Puppet and it's config files are located in `/etc/puppet`
```
[root@server1 yum.repos.d]# cd /etc/puppet
[root@server1 puppet]# mkdir manifests
[root@server1 puppet]# ls -al
total 24
drwxr-xr-x.   4 root root   74 Mar 15 21:05 .
drwxr-xr-x. 121 root root 8192 Mar 15 20:51 ..
-rw-r--r--.   1 root root 4178 Jun  9  2014 auth.conf
drwxr-xr-x.   2 root root    6 Mar 15 21:05 manifests
drwxr-xr-x.   2 root root    6 Aug 19  2014 modules
-rw-r--r--.   1 root root  853 Jun  9  2014 puppet.conf
[root@server1 puppet]# cd manifests
```

In the manifests directory we create a file called site.pp and in that file we enter
the following.  This is the node definition for a named node and we have a file rule
to ensure a file is in place with the attributes we have specified in the manifest
```
[root@server1 manifests]# vi site.pp
node "server1.example.com" {
    file { '/etc/yum.repos.d/local.repo':
            ensure => "file",
            owner => "root",
            group => "wheel",
            mode => "644",
            content => "[localc7]
name=CentOS 7 Local
baseurl=http://192.168.56.220/centos7
gpgcheck=0
enabled=1
"}
}
```

To use the puppet client to apply the manifest
```
[root@server1 manifests]# puppet apply site.pp
Notice: Compiled catalog for server1.example.com in environment production in 0.08 seconds
Notice: /Stage[main]/Main/Node[server1.example.com]/File[/etc/yum.repos.d/local.repo]/ensure: defined content as '{md5}c685fe7171168871d26edc5d4af08f87'
Notice: Finished catalog run in 0.04 seconds
```

If we go the /etc/yum.repos.d/ dir, we can see that our new file has been created
with the correct contents and attributes
```
[root@server1 manifests]# ls -l /etc/yum.repos.d/local.repo
-rw-r--r--. 1 root wheel 89 Mar 15 21:16 /etc/yum.repos.d/local.repo
[root@server1 manifests]# cat !$
cat /etc/yum.repos.d/local.repo
[localc7]
name=CentOS 7 Local
baseurl=http://192.168.56.220/centos7
gpgcheck=0
enabled=1
```

If you were to change anything about the file, such as it's contents or ownership,
if you then use puppet to reapply the manifest, it will ensure the file conforms with the
policy as stated in the manifest.
