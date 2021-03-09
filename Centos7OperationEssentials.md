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
