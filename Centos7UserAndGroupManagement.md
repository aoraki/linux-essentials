# CentOS Enterprise Linux 7 User and Group Management - Course Notes

Path to pluralsight course : https://app.pluralsight.com/library/courses/lfcs-linux-user-group-management/table-of-contents

## Introduction

To look at the local user account DB, open up `passwd`
```
[centos@server1 ~]$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
centos:x:1000:1000:centos:/home/centos:/bin/bash
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
unbound:x:997:994:Unbound DNS resolver:/etc/unbound:/sbin/nologin
geoclue:x:996:993:User for geoclue:/var/lib/geoclue:/sbin/nologin
usbmuxd:x:113:113:usbmuxd user:/:/sbin/nologin
openvpn:x:995:992:OpenVPN:/etc/openvpn:/sbin/nologin
abrt:x:173:173::/etc/abrt:/sbin/nologin
setroubleshoot:x:994:990::/var/lib/setroubleshoot:/sbin/nologin
lightdm:x:993:989::/var/lib/lightdm:/sbin/nologin
nm-openconnect:x:992:988:NetworkManager user for OpenConnect:/:/sbin/nologin
nm-openvpn:x:991:986:Default user for running openvpn spawned by NetworkManager:/:/sbin/nologin
vboxadd:x:990:1::/var/run/vboxadd:/bin/false
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
puppet:x:52:52:Puppet:/var/lib/puppet:/sbin/nologin
```

### Using getent

getent is a utility that can be used to get entries from a number of important
text files in a linux system called databases.  Examples of this are passwd, services,
networks, hosts, group

The name services switch mechanism is the mechanism that is used by getent to search in mulitple locations
for users.  If we look in the nssswitch.conf file we can see the places where passwd will check
(files for local file system, and sss for system security services subsystem which covers Active directory)
```
[centos@server1 ~]$ grep passwd /etc/nsswitch.conf
#passwd:    db files nisplus nis
passwd:     files sss
```

The `cat /etc/passwd` command only shows you local users.  If you are integrated with an AD domain, you can
use getent, which will return the local users as well as the users in the domain.
In our case there are only local users so the output is the same as the command above
```
[centos@server1 ~]$ getent passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
...
```

To get an individual entry from a database, in this case getting a user called
centos from the `passwd` database
```
[centos@server1 ~]$ getent passwd centos
centos:x:1000:1000:centos:/home/centos:/bin/bash
```

To get a list of groups, can be used for local groups and domain-based groups
```
[centos@server1 ~]$ getent group
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:centos
...
```

To use getent to get a list of networks
```
[centos@server1 ~]$ getent networks
default               0.0.0.0
loopback              127.0.0.0
link-local            169.254.0.0
```

To get a list of services/port names.  This is basically querying the /etc/services file
If we are integrated with LDAP, we can use this command to get a list of services on the LDAP
servers
```
[centos@server1 ~]$ getent services
tcpmux                1/tcp
tcpmux                1/udp
rje                   5/tcp
rje                   5/udp
echo                  7/tcp
echo                  7/udp
discard               9/tcp sink null
discard               9/udp sink null
systat                11/tcp users
systat                11/udp users
daytime               13/tcp
daytime               13/udp
...
```


### Login vs Non-Login shells

There is a login shell and a non-login shell. When we ssh to a machine or login on
the console itself, that starts up a new login shell.  If we also use a `su -`or `su -l`
that will initiate a new login shell.

The login shell is going to first look for the `/etc/profile` login script, and thats
going to include it's extension directory `/etc/profile.d`.

When the profile login script runs it wll look for the `~/.bash_profile` file and that
exists by default on a CentOS7 system. If the `.bash_profile` doesn't exist we can default to
looking for the `~/.profile` file.

The `.bash_profile` script is going to call the `~/.bashrc` and the first thing that that is going
to do is check the `/etc/bashrc` script to see if it exists and execute it.

So when we get a login shell, we get everything, our full environment is set.

If we use a non-login shell (`su` by itself), or we just run the `bash` command
then we only run the `~/.bashrc` which in turn runs `/etc/bashrc`, but we miss out on
the whole profile structure in this case

### Modifying the System Login Scripts

As explained above, in your `/etc` directory there is a profile script that get's
executed along with any scrips in the /etc/profile.d directory.  If you wanted a
script to be executed at login time, you don't need to edit the `profile` script,
you can just drop the new script into the /etc/profile.d directory. As you install
new software on your system you will need additional scripts being added to this
directory.
```
centos@server1 etc]$ ls profile*
profile

profile.d:
256term.csh  256term.sh  bash_completion.sh  colorgrep.csh  colorgrep.sh  colorls.csh  colorls.sh  csh.local  flatpak.sh  lang.csh  lang.sh  less.csh  less.sh  sh.local  vim.csh  vim.sh  vte.sh  which2.csh  which2.sh
```

You can also change your bash behavior by editing the `/etc/bashrc` file.  For example
you can change the value of the PS1 variable (which controls the bash prompt), and
have it display the full current working directory path rather than just the current
directory you are in.

### Home Directory Templates

The `/etc/skel` directory is the default directory template that is used to create
home directories for **new** users.  Modifying it will not affect existing users.
The contents of this directory becomes the contents of the home directory for new
users
```
[root@server1 /var/tmp]# cd /etc/skel/
[root@server1 /etc/skel]# ls -al
total 28
drwxr-xr-x.   3 root root   92 Mar  2 11:41 .
drwxr-xr-x. 121 root root 8192 Mar 21 14:06 ..
-rw-r--r--.   1 root root   18 Apr  1  2020 .bash_logout
-rw-r--r--.   1 root root  193 Apr  1  2020 .bash_profile
-rw-r--r--.   1 root root  231 Apr  1  2020 .bashrc
drwxr-xr-x.   4 root root   39 Feb 27 16:13 .mozilla
-rw-r--r--.   1 root root  658 Apr  7  2020 .zshrc
```

**Useful side note!** In a bash script the `.` and `source` commands mean the same thing -
they both are used to execute a script.

## Creating and Managing Local Users in CentOS7
### Introducing accounts and the id command

To get info for the current user.  This shows the user id and the group id, which in
this case are the same, but they don't always have to be the same.  CentOS and Redhat based
systems use private groups so each user account is going to be a member of their own group.
The idea around private groups is that it keeps your user data private unless you choose to
share it out.

This command will also show us the `groups` info, which show us which groups the user is a
member of, and these groups can be used as an indicator as to what resources can be accessed.
So when you are accessing resources you can access resources that have access permissions to
either of these groups.

The selinux context of the user is also shown.
```
[centos@server1 ~]$ id
uid=1000(centos) gid=1000(centos) groups=1000(centos),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

To get info for another user
```
[centos@server1 ~]$ id root
uid=0(root) gid=0(root) groups=0(root)
```

To see the primary group id - `ig -g`
To see the primary group id - `ig -G`
To see the group names      - `id -gn`

### Creating local user accounts

You can have local accounts or domain-based accounts.

To create a local user (you need elevated privileges and you will be prompted for
your password).  The -m flag is to create a home directory, but this is default
behaviour, so it can be left out and the home directory will still be created.
```
[centos@server1 ~]$ sudo useradd -m user1
[sudo] password for centos:
```

`passwd` is where local users are stored, to verify that our new user has been added
We can see that the password field is an `x` which means it is stored in the `shadow` file.
The comments field is blank in this case `::`
```
[centos@server1 ~]$ tail -n 1 /etc/passwd
user1:x:1001:1001::/home/user1:/bin/bash
```

The contents of the new user's home directory will have been derived from the `/etc/skel`
directory template
```
[centos@server1 ~]$ sudo ls -al /home/user1
[sudo] password for centos:
total 16
drwx------. 3 user1 user1  92 Mar 21 16:40 .
drwxr-xr-x. 4 root  root   33 Mar 21 16:40 ..
-rw-r--r--. 1 user1 user1  18 Apr  1  2020 .bash_logout
-rw-r--r--. 1 user1 user1 199 Mar 21 14:18 .bash_profile
-rw-r--r--. 1 user1 user1 236 Mar 21 14:16 .bashrc
drwxr-xr-x. 4 user1 user1  39 Feb 27 16:13 .mozilla
-rw-r--r--. 1 user1 user1 658 Apr  7  2020 .zshrc
```

To create a new user with no private group to be created (`-N`), with a certain primary group
(`-g users`) and a certain secondary group (`-G adm`).  We verify that the user's home
directory has also been created.
```
[centos@server1 ~]$ sudo useradd -N user2 -g users -G adm
[centos@server1 ~]$ !t
tail -n 1 /etc/passwd
user2:x:1002:100::/home/user2:/bin/bash
[centos@server1 ~]$ ls /home/
centos  user1  user2
```

To create a user with a custom shell (the ordinary bourne shell)
```
[centos@server1 ~]$ sudo useradd user3 -G adm -s /bin/sh
[centos@server1 ~]$ !t
tail -n 1 /etc/passwd
user3:x:1003:1003::/home/user3:/bin/sh
```

On a Ubuntu/Debian based system there is a utility called `/sbin/adduser` where you can
also specify the user's password during user creation.  But on a CentOS7/RHEL system
this is symlinked to the `useradd` command
```
[centos@server1 ~]$ ls -l /sbin/adduser
lrwxrwxrwx. 1 root root 7 Feb 27 15:42 /sbin/adduser -> useradd
```

### Managing user passwords
When you create a new user you don't specify their passwords at creation time.

To set a user's password.  You will need to enter your own password and then enter
the user's password and retype to confirm it
```
[centos@server1 ~]$ sudo passwd user1
[sudo] password for centos:
Changing password for user user1.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

User passwords are stored in the /etc/shadow file.  They are encrypted using a
SHA 512 algorithm
```
[centos@server1 ~]$ sudo grep user1 /etc/shadow
user1:$6$4IN6vMEf$Lod7W.r2XLZecVudYmKw.FCoD.B4odS9NcmM/El8CScfkaXmfrLKCwGR01W/0T6iE2xNrr4Wd1RNuT36op2x5/:18707:0:99999:7:::
```

To see an example of shadow data for users and their passwords.  user2 and user3
have invalid passwords, denoted by the `!!` in the second field.  The third field
is the date the password was changed (in number of days since 1st Jan 1970).  The
0 is the number of days the user has to keep the password before changing, it's not
set in this case.  The 99999 is the how often in days the password needs to be changed,
you get a warning 7 days before it expires.
```
[centos@server1 ~]$ sudo grep user. /etc/shadow
user1:$6$4IN6vMEf$Lod7W.r2XLZecVudYmKw.FCoD.B4odS9NcmM/El8CScfkaXmfrLKCwGR01W/0T6iE2xNrr4Wd1RNuT36op2x5/:18707:0:99999:7:::
user2:!!:18707:0:99999:7:::
user3:!!:18707:0:99999:7:::
```

To change the password for a particular user.  In this example we are passing the
name of the user and it's new password to the `chpasswd` command (which requires
elevated privileges to run)
```
centos@server1 ~]$ echo 'user2:Password1' | sudo chpasswd
[sudo] password for centos:
[centos@server1 ~]$ sudo grep user. /etc/shadow
user1:$6$4IN6vMEf$Lod7W.r2XLZecVudYmKw.FCoD.B4odS9NcmM/El8CScfkaXmfrLKCwGR01W/0T6iE2xNrr4Wd1RNuT36op2x5/:18707:0:99999:7:::
user2:$6$aj.y/yeVvQWj6Un$bZaJATQtkdIxSNi.pVV9oeCwyKNh0S/Cr7p.kS.eb5afkEz7cfW1jipb5hHUQ0ZlfL4ppKLpB6R64J8OvHP4e.:18707:0:99999:7:::
user3:!!:18707:0:99999:7:::
```

A different way to change the password.  But this is specific to RHEL/CentOS based
systems, so better to learn the above approach which can also be used on Ubuntu
systems
```
[centos@server1 ~]$ echo Password1 | sudo passwd user3 --stdin
Changing password for user user3.
passwd: all authentication tokens updated successfully.
[centos@server1 ~]$ sudo grep user. /etc/shadow
user1:$6$4IN6vMEf$Lod7W.r2XLZecVudYmKw.FCoD.B4odS9NcmM/El8CScfkaXmfrLKCwGR01W/0T6iE2xNrr4Wd1RNuT36op2x5/:18707:0:99999:7:::
user2:$6$aj.y/yeVvQWj6Un$bZaJATQtkdIxSNi.pVV9oeCwyKNh0S/Cr7p.kS.eb5afkEz7cfW1jipb5hHUQ0ZlfL4ppKLpB6R64J8OvHP4e.:18707:0:99999:7:::
user3:$6$j75ocy9g$A2eplLGmiSkn1/6UaZUbYPdkHIepKz5ZQiyUNjWvENBv4db/slf5xle0zQwmQTRkLqsLg3PRrMMnSdyJpla1s.:18707:0:99999:7:::
```

### Password age data

The `shadow` data shown above shows the age data for each user and their passwords,
albeit in a not too human friendly way. Using the `chage` command will display this
information in a human readable format.  If you query a user other than your own you
need elevated privileges.  This info comes from the shadow file.
```
[centos@server1 ~]$ sudo chage -l centos
Last password change					: never
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
```

Password data is normally kept in the shadow file for security reasons.  If you
want your passwords to be stored in the /etc/passwd file, you can run `pwunconv`.
However doing it this way you lose the password aging info from the shadow file.
```
centos@server1 ~]$ sudo pwunconv
[centos@server1 ~]$ grep user1 /etc/passwd
user1:$6$4IN6vMEf$Lod7W.r2XLZecVudYmKw.FCoD.B4odS9NcmM/El8CScfkaXmfrLKCwGR01W/0T6iE2xNrr4Wd1RNuT36op2x5/:1001:1001::/home/user1:/bin/bash
```

To put the password data back into `shadow` run
```
[centos@server1 ~]$ sudo pwconv
[centos@server1 ~]$ grep user1 /etc/passwd
user1:x:1001:1001::/home/user1:/bin/bash
```

To see what is possible with the `chage` command
```
[centos@server1 ~]$ chage --help
Usage: chage [options] LOGIN

Options:
  -d, --lastday LAST_DAY        set date of last password change to LAST_DAY
  -E, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE
  -h, --help                    display this help message and exit
  -I, --inactive INACTIVE       set password inactive after expiration
                                to INACTIVE
  -l, --list                    show account aging information
  -m, --mindays MIN_DAYS        set minimum number of days before password
                                change to MIN_DAYS
  -M, --maxdays MAX_DAYS        set maximum number of days before password
                                change to MAX_DAYS
  -R, --root CHROOT_DIR         directory to chroot into
  -W, --warndays WARN_DAYS      set expiration warning days to WARN_DAYS
```

We will use the above knowledge to force a user password change every 40 days
```
[centos@server1 ~]$ sudo chage -M 40 user1
[centos@server1 ~]$ chage -l user1
chage: Permission denied.
[centos@server1 ~]$ sudo chage -l user1
Last password change					: Mar 21, 2021
Password expires					: Apr 30, 2021
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 40
Number of days of warning before password expires	: 7
```

To lock a password for a user.  This command really just pre-pends the `!!` in front
of the password rendering it invalid. Requires elevated privileges
```
[centos@server1 ~]$ sudo passwd -l user1
Locking password for user user1.
passwd: Success
[centos@server1 ~]$ sudo grep user1 /etc/shadow
user1:!!$6$4IN6vMEf$Lod7W.r2XLZecVudYmKw.FCoD.B4odS9NcmM/El8CScfkaXmfrLKCwGR01W/0T6iE2xNrr4Wd1RNuT36op2x5/:18707:0:40:7:::
```

To unlock the password for a user. You can see the password is no longer invalid.
Requires elevated privileges
```
[centos@server1 ~]$ sudo passwd -u user1
Unlocking password for user user1.
passwd: Success
[centos@server1 ~]$ sudo grep user1 /etc/shadow
user1:$6$4IN6vMEf$Lod7W.r2XLZecVudYmKw.FCoD.B4odS9NcmM/El8CScfkaXmfrLKCwGR01W/0T6iE2xNrr4Wd1RNuT36op2x5/:18707:0:40:7:::
```

### User account defaults
