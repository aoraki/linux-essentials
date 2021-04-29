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

We can use default info to be applied when we create new users.  The default information
is pulled from a couple of locations

The first file is the `login.defs` file.  This contains things like password aging
default values, id ranges for users and groups, home directory creation default value,
the `umask` value, the password encryption algorithm, etc.
```
[centos@server1 ~]$ cat /etc/login.defs
#
# Please note that the parameters in this configuration file control the
# behavior of the tools from the shadow-utils component. None of these
# tools uses the PAM mechanism, and the utilities that use PAM (such as the
# passwd command) should therefore be configured elsewhere. Refer to
# /etc/pam.d/system-auth for more information.
#

# *REQUIRED*
#   Directory where mailboxes reside, _or_ name of file, relative to the
#   home directory.  If you _do_ define both, MAIL_DIR takes precedence.
#   QMAIL_DIR is for Qmail
#
#QMAIL_DIR	Maildir
MAIL_DIR	/var/spool/mail
#MAIL_FILE	.mail

# Password aging controls:
#
#	PASS_MAX_DAYS	Maximum number of days a password may be used.
#	PASS_MIN_DAYS	Minimum number of days allowed between password changes.
#	PASS_MIN_LEN	Minimum acceptable password length.
#	PASS_WARN_AGE	Number of days warning given before a password expires.
#
PASS_MAX_DAYS	99999
PASS_MIN_DAYS	0
PASS_MIN_LEN	5
PASS_WARN_AGE	7

#
# Min/max values for automatic uid selection in useradd
#
UID_MIN                  1000
UID_MAX                 60000
# System accounts
SYS_UID_MIN               201
SYS_UID_MAX               999

#
# Min/max values for automatic gid selection in groupadd
#
GID_MIN                  1000
GID_MAX                 60000
# System accounts
SYS_GID_MIN               201
SYS_GID_MAX               999

#
# If defined, this command is run when removing a user.
# It should remove any at/cron/print jobs etc. owned by
# the user to be removed (passed as the first argument).
#
#USERDEL_CMD	/usr/sbin/userdel_local

#
# If useradd should create home directories for users by default
# On RH systems, we do. This option is overridden with the -m flag on
# useradd command line.
#
CREATE_HOME	yes

# The permission mask is initialized to this value. If not specified,
# the permission mask will be initialized to 022.
UMASK           077

# This enables userdel to remove user groups if no members exist.
#
USERGROUPS_ENAB yes

# Use SHA512 to encrypt password.
ENCRYPT_METHOD SHA512
```

To see additional default info given to users on creation
```
[centos@server1 ~]$ useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

To change one of the defaults we can add an extra switch to the command, in this
example use a different default shell
```
[centos@server1 ~]$ sudo useradd -Ds /bin/sh
[sudo] password for centos:
[centos@server1 ~]$ useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/sbin/nologin
SKEL=/etc/skel
CREATE_MAIL_SPOOL=no
```

The above information is held in a file and you can edit this file directly as well
```
[centos@server1 ~]$ sudo cat /etc/default/useradd
# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/sh
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

### Modifying and deleting user accounts

In the `/etc/passwd` there is a field that we can use to store the user's full name.
The field is the comments field but it is commonly used to store the user's full name
To modify a user;
```
[centos@server1 ~]$ sudo usermod -c "User One" user1
[centos@server1 ~]$ grep user1 /etc/passwd
user1:x:1001:1001:User One:/home/user1:/bin/bash
```

Other things we can modify for a user is the default shell.  To see a list of
available shells use `chsh`
```
[centos@server1 ~]$ chsh -l
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
/bin/zsh
```

We can use chsh to modify the shell of a user.  If it's for another user we need
elevated privileges
```
[centos@server1 ~]$ chsh -s /bin/sh centos
Changing shell for centos.
Password:
Shell changed.
[centos@server1 ~]$ grep centos /etc/passwd
centos:x:1000:1000:centos:/home/centos:/bin/sh
```

But it's better to use the `usermod` command
```
[centos@server1 ~]$ sudo usermod -s /bin/bash centos
[sudo] password for centos:
[centos@server1 ~]$ grep centos /etc/passwd
centos:x:1000:1000:centos:/home/centos:/bin/bash
```

To delete a user. Using `-r` will do a thorough deletion, including removing the
users home directory, mail spool files and cron jobs.
```
[centos@server1 ~]$ sudo userdel -r user2
[centos@server1 ~]$ ls /home
centos  user1  user3
```

To leave the home directory in place when deleting a user, omit `-r`.  We can also
see that we can't resolve the user name from the uid anymore, because the user is
deleted.
```
[centos@server1 ~]$ sudo userdel user3
[centos@server1 ~]$ ls /home
centos  user1  user3
[centos@server1 ~]$ ls -l /home
total 4
drwx------. 14 centos centos 4096 Mar 12 10:47 centos
drwx------.  3 user1  user1    92 Mar 21 16:40 user1
drwx------.  3   1003   1003   92 Mar 21 16:52 user3
```

To remove any residual files owned by the user that was deleted, we can delete them
using a find and the uid
```
[centos@server1 ~]$ sudo find /home -uid 1003 -delete
[centos@server1 ~]$ ls /home
centos  user1
```

## Managing local groups in CentOS7

### Creating local groups
CentOS7 has several flat files or "databases" that contain things like user info,
group info, services etc.  The group database is located in `/etc/group`

You can grep the group for a group name or a user name.  Below we are searching for a
username, and it shows which groups the user belongs to.  The second group is the user's
private group that gets created when the user is created.
```
[centos@server1 ~]$ grep centos /etc/group
wheel:x:10:centos
centos:x:1000:centos
```

This info is confirmed when we run `id`
```
[centos@server1 ~]$ id
uid=1000(centos) gid=1000(centos) groups=1000(centos),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

To change your primary group id in the current session.  Then if we create a new
file we can see that the group ownership is the wheel group
```
[centos@server1 ~]$ newgrp wheel
[centos@server1 ~]$ id
uid=1000(centos) gid=10(wheel) groups=10(wheel),1000(centos) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[centos@server1 ~]$ touch g1
[centos@server1 ~]$ ls -l g1
-rw-r--r--. 1 centos wheel 0 Mar 22 21:22 g1
```

If we then exit out of hte current session, the primary group id will revert to it's
normal setting
```
centos@server1 ~]$ exit
exit
[centos@server1 ~]$ touch g2
[centos@server1 ~]$ ls -l g2
-rw-rw-r--. 1 centos centos 0 Mar 22 21:24 g2
```

To create a new group, use `groupadd`.  When we grep the group database we see the
new group.  The `x` in the second field denotes that the group password is held in the
`gshadow` file
```
[centos@server1 ~]$ sudo groupadd sales
[sudo] password for centos:
[centos@server1 ~]$ grep sales /etc/group
sales:x:1002:
```

When we look at the gshadow file we can see an entry for our group, and it has an
invalid password.  Elevated privileges are required to look at the `gshadow` file
```
[centos@server1 ~]$ sudo grep sales /etc/gshadow
sales:!::
```

`groupadd`, `groupmod` and `groupdel` are the commands used to add, edit and delete a group

### Managing group membership

To add a user to another secondary group, "sales" use this. You need to include the existing secondary group if you want to keep it. If you were to just list the sales group the user would be removed from the wheel group.
This is the method you would use if you wanted to do it from a user perspective
```
centos@server1 ~]$ sudo usermod -G sales,wheel centos
[sudo] password for centos:
Sorry, try again.
[sudo] password for centos:
[centos@server1 ~]$ id -Gn centos
centos wheel sales
[centos@server1 ~]$ id -G centos
1000 10 1002
```

If you wanted to do it from a group perspective, ie. add a load of users to a group, then use something like this;
```
[centos@server1 ~]$ sudo gpasswd -a centos sales
Adding user centos to group sales
```

To add in a list of members at the same time
```
[centos@server1 ~]$ sudo gpasswd -M centos,root,user1 sales
[centos@server1 ~]$ grep sales /etc/group
sales:x:1002:centos,root,user1
```

If you run `id` for your own user, you will not see the group settings updated, you need to start a new session before you will see that
```
[centos@server1 ~]$ id
uid=1000(centos) gid=1000(centos) groups=1000(centos),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

[centos@server1 ~]$ su -l centos
Last login: Wed Mar 24 20:40:09 GMT 2021 from 192.168.99.1 on pts/0
[centos@server1 ~]$ id
uid=1000(centos) gid=1000(centos) groups=1000(centos),10(wheel),1002(sales) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[centos@server1 ~]$ id -Gn
centos wheel sales
```

### Setting the SGID Permission

For this we are going to use the files associated with the httpd service, `/var/www/html`
When you install httpd initially it is owned by the root group, but the "others" section
will have read and execute privileges
```
[centos@server1 ~]$ ls -ld /var/www/html
drwxr-xr-x. 2 root root 6 Nov 16 16:19 /var/www/html
```

We should recursively change the group ownership of the `/var/www` directory to the apache group
```
[centos@server1 ~]$ sudo chgrp -R apache /var/www
[centos@server1 ~]$ ls -ld /var/www
drwxr-xr-x. 4 root apache 33 Mar 14 21:36 /var/www
```

However we should remove the "others" privileges, they are no longer needed
```
[centos@server1 ~]$ sudo chmod -R o= /var/www
[centos@server1 ~]$ ls -ld /var/www
drwxr-x---. 4 root apache 33 Mar 14 21:36 /var/www
```

In the html directory we want to set the SGID bit, which will change the group `x`
to an `s` to denote the sticky bit has been set.  So this means that when any new file
is created in the html directory, its going to be group owned by the directory's owner.
```
[centos@server1 ~]$ sudo -i
[root@server1 ~]# cd /var/www/html
[root@server1 /var/www/html]# ls -ld
drwxr-x---. 2 root apache 24 Mar 24 21:11 .
[root@server1 /var/www/html]# chmod g+s .
[root@server1 /var/www/html]# ls -ld
drwxr-s---. 2 root apache 24 Mar 24 21:11 .
```

We set `umask 027` to keep privileges away from the others group
```
[root@server1 /var/www/html]# umask 027
```

If we, as the root user, now create a new file under the `/var/www/html` dir, instead of it
being owned by the root group, it will belong to the apache group because the sticky bit has
been set.
```
[root@server1 /var/www/html]# vi test.html
[root@server1 /var/www/html]# ls -l
total 8
-rw-r-----. 1 root apache  9 Mar 24 21:11 index.html
-rw-r-----. 1 root apache 15 Mar 24 21:26 test.html
[root@server1 /var/www/html]# id
uid=0(root) gid=0(root) groups=0(root),1002(sales) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

### Group passwords

Group passwords are stored in the `/etc/gshadow` file.

You need to enter the password of a group when you want to change the primary group
associated with a user.  But first the group must have a valid password.  You can check that by
grepping the `/etc/group` directory
```
[centos@server1 ~]$ grep adm /etc/group
adm:!:4:
```

To set the password for a group.  You need to enter your own password and then the groups password
```
[centos@server1 ~]$ sudo gpasswd adm
[sudo] password for centos:
Changing the password for group adm
New Password:
Re-enter new password:
```

Then if you check the `/etc/group` file again you will see that the password is now in the `gshadow`
file
```
[centos@server1 ~]$ grep adm /etc/group
adm:x:4:
```

Then to change the primary group for your user, the group now has a valid password that
can be entered
```
[centos@server1 ~]$ newgrp adm
Password:
[centos@server1 ~]$ id -gn
adm
[centos@server1 ~]$ id -Gn
adm wheel centos sales
```

**Side note:** Group passwords actually lessens security.  If you have a group's password you can change your
primary group ownership and possibly gain more access than you currently have

## Using PAM to Control User Access

`PAM` are pluggable authentication modules.  When you login it accesses the PAM
and that will give you access to a session (assuming you have the permissions to
do so).

User logins can come from SSH, through the server console or through a GUI.

PAM modules can tie in to our login program and provide methods for Authentication
(which can be password based, biometric based etc).  There could be limits applied
such as when we're allowed to login, how many times, etc.  We can create our home
directory during the login process and that's going to be affected by a PAM module.

When you have gained access to our session PAM can continue to monitor things such
as our access to resources, such as how many processes we're allowed to run, how much memory
can be used, how many files we can open.  This can be continually monitored by PAM modules.

Password policy is another thing that can be monitored by PAM, it can ensure that we are
applying complexity rules etc.


### Create home directories at Login

PAM-related files and config are located in the following three locations;

* `/etc/security/` - Contains config files for the PAM modules
* `/etc/pam.d/` - Contains config files for the authentication programs and programs
that can use PAM.
* `/lib64/security/`

To get a list of the programs that can be tied into PAM, and their PAM configuration
files
```
[centos@server1 ~]$ ls -al /etc/pam.d/
total 160
drwxr-xr-x.   2 root root 4096 Mar  2 11:39 .
drwxr-xr-x. 122 root root 8192 Mar 25 21:25 ..
-rw-r--r--.   1 root root  272 Oct 30  2018 atd
-rw-r--r--.   1 root root  192 Feb  2 16:31 chfn
-rw-r--r--.   1 root root  192 Feb  2 16:31 chsh
-rw-r--r--.   1 root root  232 Apr  1  2020 config-util
-rw-r--r--.   1 root root  287 Aug  9  2019 crond
lrwxrwxrwx.   1 root root   19 Feb 27 15:45 fingerprint-auth -> fingerprint-auth-ac
-rw-r--r--.   1 root root  702 Feb 27 16:12 fingerprint-auth-ac
-rw-r--r--.   1 root root  963 Nov 28  2017 lightdm
-rw-r--r--.   1 root root  698 Nov 28  2017 lightdm-autologin
-rw-r--r--.   1 root root  409 Nov 28  2017 lightdm-greeter
-rw-r--r--.   1 root root   97 Oct  1 17:29 liveinst
-rw-r--r--.   1 root root  796 Feb  2 16:31 login
-rw-r--r--.   1 root root  413 Feb  5  2017 mate-screensaver
-rw-r--r--.   1 root root  121 Nov  3  2018 mate-system-log
-rw-r--r--.   1 root root  154 Apr  1  2020 other
-rw-r--r--.   1 root root  188 Apr  1  2020 passwd
lrwxrwxrwx.   1 root root   16 Feb 27 15:45 password-auth -> password-auth-ac
-rw-r--r--.   1 root root 1033 Feb 27 16:12 password-auth-ac
-rw-r--r--.   1 root root  510 Aug  6  2020 pluto
-rw-r--r--.   1 root root  155 Apr  1  2020 polkit-1
lrwxrwxrwx.   1 root root   12 Feb 27 15:45 postlogin -> postlogin-ac
-rw-r--r--.   1 root root  330 Feb 27 15:45 postlogin-ac
-rw-r--r--.   1 root root  144 Feb 27  2020 ppp
-rw-r--r--.   1 root root  681 Feb  2 16:31 remote
-rw-r--r--.   1 root root  143 Feb  2 16:31 runuser
-rw-r--r--.   1 root root  138 Feb  2 16:31 runuser-l
-rw-r--r--.   1 root root   36 Sep 30 14:19 screen
lrwxrwxrwx.   1 root root   17 Feb 27 15:45 smartcard-auth -> smartcard-auth-ac
-rw-r--r--.   1 root root  752 Feb 27 16:12 smartcard-auth-ac
lrwxrwxrwx.   1 root root   25 Feb 27 15:42 smtp -> /etc/alternatives/mta-pam
-rw-r--r--.   1 root root   76 Apr  1  2020 smtp.postfix
-rw-r--r--.   1 root root  904 Aug  9  2019 sshd
-rw-r--r--.   1 root root  540 Feb  2 16:31 su
-rw-r--r--.   1 root root  200 Jan 26 21:56 sudo
-rw-r--r--.   1 root root  178 Jan 26 21:56 sudo-i
-rw-r--r--.   1 root root  137 Feb  2 16:31 su-l
lrwxrwxrwx.   1 root root   14 Feb 27 15:45 system-auth -> system-auth-ac
-rw-r--r--.   1 root root 1031 Feb 27 16:12 system-auth-ac
-rw-r--r--.   1 root root   97 Aug  3  2017 system-config-language
-rw-r--r--.   1 root root  129 Feb  2 16:34 systemd-user
-rw-r--r--.   1 root root   84 Oct 30  2018 vlock
-rw-r--r--.   1 root root  163 Feb 24 21:10 xserver
```

The shared libraries that PAM uses are all located at `/lib64/security/`

We can configure some of the PAM modules by accessing the following files
```
[centos@server1 ~]$ ls -al /etc/security/
total 68
drwxr-xr-x.   6 root root 4096 Mar  9 21:24 .
drwxr-xr-x. 122 root root 8192 Mar 25 21:25 ..
-rw-r--r--.   1 root root 4564 Apr  1  2020 access.conf
-rw-r--r--.   1 root root   82 Apr  1  2020 chroot.conf
drwxr-xr-x.   2 root root  109 Feb 27 16:13 console.apps
-rw-r--r--.   1 root root  604 Apr  1  2020 console.handlers
-rw-r--r--.   1 root root  939 Apr  1  2020 console.perms
drwxr-xr-x.   2 root root    6 Apr  1  2020 console.perms.d
-rw-r--r--.   1 root root 3635 Apr  1  2020 group.conf
-rw-r--r--.   1 root root 2442 Mar  9 21:20 limits.conf
drwxr-xr-x.   2 root root   27 Feb 27 15:42 limits.d
-rw-r--r--.   1 root root 1440 Apr  1  2020 namespace.conf
drwxr-xr-x.   2 root root    6 Apr  1  2020 namespace.d
-rwxr-xr-x.   1 root root 1019 Apr  1  2020 namespace.init
-rw-------.   1 root root    0 Apr  1  2020 opasswd
-rw-r--r--.   1 root root 2972 Apr  1  2020 pam_env.conf
-rw-r--r--.   1 root root 1718 Dec  6  2011 pwquality.conf
-rw-r--r--.   1 root root  419 Apr  1  2020 sepermit.conf
-rw-r--r--.   1 root root 2179 Apr  1  2020 time.conf
```

The default behaviour when creating users using the useradd command is for the user's
home directory to be created.  However if you are batch creating hundreds of users, you
may not want to create all the home directories at user creation time, so you can turn off
the home directory creation by editing the `/etc/login.defs` file and change the
`CREATE_HOME` flag to no.
```
[centos@server1 ~]$ sudo vi /etc/login.defs

#
# If useradd should create home directories for users by default
# On RH systems, we do. This option is overridden with the -m flag on
# useradd command line.
#
CREATE_HOME     no
```

Then if we create a user we will see that his home directory has not been created, but
if we grep the passwd file we can see what his home directory will be called when it's
created.  We can also see the user's entry in the shadow file but it's showing as
an invalid password because it hasn't been set yet
```
[centos@server1 ~]$ sudo useradd bob
[centos@server1 ~]$ ls /home
centos  user1
[centos@server1 ~]$ grep bob /etc/passwd
bob:x:1002:1003::/home/bob:/bin/bash
[centos@server1 ~]$ sudo grep bob /etc/shadow
bob:!!:18711:0:99999:7:::
```

To set a password for the user bob, we just used the `passwd` command
```
[centos@server1 ~]$ sudo passwd bob
Changing password for user bob.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

Now the new user has a password so should be able to log.  But the user doesn't
have a home directory yet.  We can use PAM to automate the creation of home directories
First we check to see if we have the correct module and service installed, the `oddjob`
service
```
[centos@server1 ~]$ rpm -qa | grep oddjob
oddjob-mkhomedir-0.31.5-4.el7.x86_64
oddjob-0.31.5-4.el7.x86_64
```

Looks like we have the oddjob-mkhomedir module, and the service.  The mkhomedir mechanism
is the preferred mechanism for creating home directories in Red Hat distros.  To use it,
we need to enable and start the `oddjob` service.
```
[centos@server1 ~]$ systemctl enable oddjobd
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ===
Authentication is required to manage system service or unit files.
Authenticating as: centos
Password:
==== AUTHENTICATION COMPLETE ===
Created symlink from /etc/systemd/system/multi-user.target.wants/oddjobd.service to /usr/lib/systemd/system/oddjobd.service.
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Authenticating as: centos
Password:
==== AUTHENTICATION COMPLETE ===
[centos@server1 ~]$ systemctl start oddjobd
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to manage system services or units.
Authenticating as: centos
Password:
==== AUTHENTICATION COMPLETE ===
[centos@server1 ~]$ sudo systemctl status oddjobd
● oddjobd.service - privileged operations for unprivileged applications
   Loaded: loaded (/usr/lib/systemd/system/oddjobd.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2021-03-25 21:52:07 GMT; 15s ago
 Main PID: 2013 (oddjobd)
   CGroup: /system.slice/oddjobd.service
           └─2013 /usr/sbin/oddjobd -n -p /var/run/oddjobd.pid -t 300

Mar 25 21:52:07 server1.example.com systemd[1]: Started privileged operations for unprivileged applications.
```

To enable the automatic home dir creation across all the PAM modules.
```
[centos@server1 ~]$ sudo authconfig --enablemkhomedir --update
```

Let's gain root privileges and look at the contents of the /etc/pam.d directory,
grepping for mkhomedir.  There are references to the `pam_oddjob_mkhomedir` are in
a lot of PAM modules.  Using the `authconfig` command enables it for us in all the modules.
Without that we'd have to manually configure all these modules.
```
[centos@server1 ~]$ sudo -i
[root@server1 ~]# cd /etc/pam.d/
[root@server1 /etc/pam.d]# grep mkhomedir *
fingerprint-auth:session     optional      pam_oddjob_mkhomedir.so umask=0077
fingerprint-auth-ac:session     optional      pam_oddjob_mkhomedir.so umask=0077
password-auth:session     optional      pam_oddjob_mkhomedir.so umask=0077
password-auth-ac:session     optional      pam_oddjob_mkhomedir.so umask=0077
smartcard-auth:session     optional      pam_oddjob_mkhomedir.so umask=0077
smartcard-auth-ac:session     optional      pam_oddjob_mkhomedir.so umask=0077
system-auth:session     optional      pam_oddjob_mkhomedir.so umask=0077
system-auth-ac:session     optional      pam_oddjob_mkhomedir.so umask=0077
```

Having done that, if we now log in as the new user bob, we can see that his home
dir is created at login time.
```
[root@server1 /etc/pam.d]# su - bob
Creating home directory for bob.
[bob@server1 ~]$ pwd
/home/bob
[bob@server1 ~]$ ls -al
total 16
drwx------. 3 bob  bob   92 Mar 25 21:54 .
drwxr-xr-x. 5 root root  44 Mar 25 21:54 ..
-rw-------. 1 bob  bob   18 Mar 25 21:54 .bash_logout
-rw-------. 1 bob  bob  199 Mar 25 21:54 .bash_profile
-rw-------. 1 bob  bob  236 Mar 25 21:54 .bashrc
drwx------. 4 bob  bob   39 Mar 25 21:54 .mozilla
-rw-------. 1 bob  bob  658 Mar 25 21:54 .zshrc
```

### Configure Password Policies

PAM modules can be used to control the quality/complexity of user passwords

The pam.d directory holds configuration files for programs that can use PAM modules.

There is a shared file that is used by many programs, `system-auth`

To look at the PAM configuration that controls password changing
```
[centos@server1 ~]$ cat /etc/pam.d/system-auth
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth        required      pam_env.so
auth        required      pam_faildelay.so delay=2000000
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        required      pam_deny.so

account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     optional      pam_oddjob_mkhomedir.so umask=0077
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
```

The settings in this file relevant to passwords are all of `password` event.  So
password changes fall under `password` type events.
```
password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
```
`requisite` means it has to pass success for anything else to run.  Then we have the name of
the module `pam_pwquality.so`.  These shared libraries are stored in `/lib64/security`.
This is followed by some options for the module such as `try_first_pass` etc. The
`pam_pwquality.so` is where we are setting our password quality, and the configuration for this
is outlined below.


To look at password quality config file , look at the `/etc/security/pwquality.conf` file.
It contains policies for things such as max and min number of characters and lots of other
constraints that can be placed on password changes.
```
[centos@server1 ~]$ less /etc/security/pwquality.conf
Configuration for systemwide password quality limits
# Defaults:
#
# Number of characters in the new password that must not be present in the
# old password.
# difok = 5
#
# Minimum acceptable size for the new password (plus one if
# credits are not disabled which is the default). (See pam_cracklib manual.)
# Cannot be set to lower value than 6.
# minlen = 9
#
# The maximum credit for having digits in the new password. If less than 0
# it is the minimum number of digits in the new password.
# dcredit = 1
#
# The maximum credit for having uppercase characters in the new password.
# If less than 0 it is the minimum number of uppercase characters in the new
# password.
# ucredit = 1
#
# The maximum credit for having lowercase characters in the new password.
# If less than 0 it is the minimum number of lowercase characters in the new
# password.
# lcredit = 1
#
# The maximum credit for having other characters in the new password.
# If less than 0 it is the minimum number of other characters in the new
# password.
# ocredit = 1
...
```

There is a handy program that gives your password a score based on the policies set
When you type the command `pwscore` you then enter a password and it gives it a
quality rating
```
[centos@server1 ~]$ pwscore
Password1
Password quality check failed:
 The password fails the dictionary check - it is based on a dictionary word

[centos@server1 ~]$ pwscore
Aorangi&1922
78
```

It even detects if your password is based on a dictionary word, even if you are using
symbols that resemble letters;
```
[centos@server1 ~]$ pwscore
P@$$w0rd1
Password quality check failed:
 The password fails the dictionary check - it is based on a dictionary word
 ```

 `pwscore` values run from 0 to 100, where 0 is a very weak password and 100 is a very
 strong one

 ### Restrict Access to Resources

 It is quite likely that you do not want to allow your users unfettered access to
 all of the resources on your linux system, as it is most likely shared with other
 users and services.  We can use PAM to control the level of access that users have through
 to the resources.

 To see the restrictions that are in place use the `ulimit` command.
 We can actually change some of these settings ourselves and don't need PAM to do so
 ```
 [centos@server1 ~]$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 3832
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 3832
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

To view a particular limit use the switch that you can identify from the above
output.  For example, to view the `max user processes (-u)`;
```
[centos@server1 ~]$ ulimit -u
3832
```

To change a particular value, you use the flag of the limit you want to change,
with the new value, such as;
```
[centos@server1 ~]$ ulimit -u 10
[centos@server1 ~]$ ulimit -u
10
```

To demonstrate this new limit we can create a script that will call itself and therefore
create a new process each time it does it.  The `$0` is what calls the current script.
However after 10 concurrent processes are reached the limit should kick in and prevent
a new process from being created.
```
[centos@server1 ~]$ vi test.sh
#!/bin/bash
echo "test test"
$0
```

Then when we make it executable and run it we get the following;
```
[centos@server1 ~]$ chmod +x test.sh
[centos@server1 ~]$ ./test.sh
test test
test test
test test
test test
test test
test test
test test
test test
./test.sh: fork: retry: No child processes
./test.sh: fork: retry: No child processes
./test.sh: fork: retry: No child processes
./test.sh: fork: retry: No child processes
```

The above describes how you could impose a limit on your own user.  However the more
likely scenario would be an admin imposing limits across users and groups in the organization.
The settings for this is controlled by the `/etc/security/limits.conf` file.
This limits policy file is laid out as follows;
```
#<domain>        <type>  <item>  <value>
```

`domain` can be a user name, or a group name (with "@group" syntax)
`type` can be `soft` or `hard`.  `soft` is the current effective limit.  `hard` is the absolute
ceiling that cannot be exceeded.  An admin can adjust the soft limit but it can never exceed
the hard limit.
`item` - describes where the limits can be implemented on.  The list of items are as follows;
```
#<item> can be one of the following:
#        - core - limits the core file size (KB)
#        - data - max data size (KB)
#        - fsize - maximum filesize (KB)
#        - memlock - max locked-in-memory address space (KB)
#        - nofile - max number of open file descriptors
#        - rss - max resident set size (KB)
#        - stack - max stack size (KB)
#        - cpu - max CPU time (MIN)
#        - nproc - max number of processes
#        - as - address space limit (KB)
#        - maxlogins - max number of logins for this user
#        - maxsyslogins - max number of logins on the system
#        - priority - the priority to run user process with
#        - locks - max number of file locks the user can hold
#        - sigpending - max number of pending signals
#        - msgqueue - max memory used by POSIX message queues (bytes)
#        - nice - max nice priority allowed to raise to values: [-20, 19]
#        - rtprio - max realtime priority
```

So for example if you wanted to set a limit for all users to have 4 concurrent logins;
```
#<domain>        <type>  <item>     <value>

*                -       maxlogins  4
```

To make some limits for the users group for number of processes.  This is setting
a soft limit (the effective limit) of 50 and a hard limit of 75, which cannot be
exceeded.
```
#<domain>        <type>  <item>     <value>
@users           soft    nproc      50
@users           hard    nproc      75
```

### Control Access Times

We can in effect put a virtual "lock" on our servers, ensuring users can only
log in at certain times of day using PAM.

To impose restrictions around ssh access we need to add a new restriction policy
to the `/etc/pam.d/sshd` file.  Add a new entry to the account section of that file,
above the other account entries
```
[centos@server1 /etc/pam.d]$ sudo vi sshd
...
account    required     pam_time.so
account    required     pam_nologin.so
account    include      password-auth
...
```
The new restriction is an `account` restriction, it is `required` which means we need to
pass but can continue. That's why we need to put this restriction in first above the others.
`pam_time.so` is the library file that is going to be checked when this restriction is
enacted.  That module is located in the `/lib64/security`

The configuration for this module is located in `/etc/security/time.conf`

To create a new login time restriction in this file, edit it and add in the shown
line at the bottom of the file.
```
[centos@server1 /etc/pam.d]$ sudo vi /etc/security/time.conf
*;*;centos|bob;Wk0800-1800
```

* The first `*` is for services, we have specified all services although we could have
specified `sshd`.
* The second `*` is for terminals, again we have specified all terminals.
* The third section is a user or list of users, in this case we have specified a pipe-delimited
list of 2 users, "centos" and "bob"
* The last section is the actual time window those users are allowed to log in under. In
this case it is week days (`Wk`) between the hours of 0800 and 1800.

If one of those users attempts to login outside of those hours then they will be blocked.

## Implementing OpenLDAP Directories on CentOS7

### Installing OpenLDAP

First of all we need to check if our hostname has been set
```
centos@server1 ~]$ hostname
server1.example.com
```

Then we need to gain root privileges
```
[centos@server1 ~]$ su -
Password:
Last login: Wed Mar 24 20:53:01 GMT 2021 on pts/0
```

Then we need to add an entry for our machine and hostname into our `/etc/hosts` file.
We can confirm our ip address by running `ip a s`.  We can confirm that our hosts
file has the entry by pinging the hostname FQDN.
```
[root@server1 ~]# echo "192.168.99.107 server1.example.com" >> /etc/hosts
[root@server1 ~]# ping server1.example.com
PING server1.example.com (192.168.99.107) 56(84) bytes of data.
64 bytes from server1.example.com (192.168.99.107): icmp_seq=1 ttl=64 time=0.064 ms
64 bytes from server1.example.com (192.168.99.107): icmp_seq=2 ttl=64 time=0.055 ms
```

We can check to make sure no ldap server is listening
```
[root@server1 ~]# netstat -ltn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 ::1:25                  :::*                    LISTEN
```

Next we need to add a firewall rule to allow ldap traffic through. The firewall
settings need to be reloaded for it to take effect in the current session (otherwise
we need to create a new session)
```
[root@server1 ~]# firewall-cmd --permanent --add-service=ldap
success
[root@server1 ~]# firewall-cmd --reload
success
```

Finally we install the various ldap packages and tools using `yum`
```
[root@server1 ~]# yum install -y openldap openldap-clients openldap-servers migrationtools.noarch
```

### Configure OpenLDAP Server

To get a DB Config example in place;
```
[root@server1 ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
[root@server1 ~]# ls -l /var/lib/ldap/
total 4
-rw-r--r--. 1 root root 845 Mar 27 21:22 DB_CONFIG
```

Next we run the `slaptest` command to generate the db files (it looks like a failure
but it actually generates the necessary files we need).  We need to change the ownership
to the `ldap` user and `ldap` group as well
```
[root@server1 ~]# slaptest
605fa268 hdb_db_open: database "dc=my-domain,dc=com": db_open(/var/lib/ldap/id2entry.bdb) failed: No such file or directory (2).
605fa268 backend_startup_one (type=hdb, suffix="dc=my-domain,dc=com"): bi_db_open failed! (2)
slap_startup failed (test would succeed using the -u switch)
[root@server1 ~]# chown ldap.ldap /var/lib/ldap/*
[root@server1 ~]# ls -l /var/lib/ldap/
total 18968
-rw-r--r--. 1 ldap ldap     2048 Mar 27 21:23 alock
-rw-------. 1 ldap ldap  2326528 Mar 27 21:23 __db.001
-rw-------. 1 ldap ldap 17448960 Mar 27 21:23 __db.002
-rw-------. 1 ldap ldap  1884160 Mar 27 21:23 __db.003
-rw-r--r--. 1 ldap ldap      845 Mar 27 21:22 DB_CONFIG
```

Next we enable and start the OpenLDAP service
```
[root@server1 ~]# systemctl start slapd
[root@server1 ~]# systemctl enable slapd
Created symlink from /etc/systemd/system/multi-user.target.wants/slapd.service to /usr/lib/systemd/system/slapd.service.
[root@server1 ~]# systemctl status slapd
● slapd.service - OpenLDAP Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/slapd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2021-03-27 21:27:36 GMT; 11s ago
     Docs: man:slapd
           man:slapd-config
           man:slapd-hdb
           man:slapd-mdb
           file:///usr/share/doc/openldap-servers/guide.html
 Main PID: 5331 (slapd)
   CGroup: /system.slice/slapd.service
           └─5331 /usr/sbin/slapd -u ldap -h ldapi:/// ldap:///
```

We check the ports being listened on to see if ldap is running using `netstat`
```
[root@server1 ~]# netstat -ltn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
tcp6       0      0 :::389                  :::*                    LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 ::1:25                  :::*                    LISTEN
[root@server1 ~]# netstat -lt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:ldap            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN
tcp6       0      0 [::]:ldap               [::]:*                  LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN
```

To configure the ldap schema files.  The schema defines what can be created in our
directory service.
```
[root@server1 /etc/openldap/schema]# ldapadd -Y EXTERNAL -H ldapi:/// -D "cn=config" -f cosine.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"

[root@server1 /etc/openldap/schema]# ldapadd -Y EXTERNAL -H ldapi:/// -D "cn=config" -f nis.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"
```

To create an encrypted password for our directory administrator, passing in the secret
(`-s`) and removing the trailing carraige return (`-n`) and sending it through to the
rootpwd file, which is just a temporary file that we need to store this password.
```
[root@server1 ~]# slappasswd -s Password1 -n > rootpwd
[root@server1 ~]# cat rootpwd
{SSHA}t2GXHIRTx7bRxiuf5ucCxs35fTA1YTAP
```

We are now going to import this password into another configuration file.
The contents of this file are provided as part of the course notes, we are just
inserting the SSHA of the password created above.
```
[root@server1 ~]# vi config.ldif

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=example,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=example,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
#Password is Password1 or add your own
olcRootPW: {SSHA}t2GXHIRTx7bRxiuf5ucCxs35fTA1YTAP

dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: 0

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=example,dc=com" read by * none
```

Finally, we can import this config file using the `ldapmodify` command.  This step
completes the configuration of the ldap server
```
[root@server1 ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -D "cn=config" -f config.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "cn=config"

modifying entry "olcDatabase={1}monitor,cn=config"
```

### Create directory structure

The previous step was about configuring the LDAP server. Now we need to create the
hierarchal structure, which is going to look after our directory tree. We are now
going to create the top-level containers that are going to organize our directory
entries.

```
[root@server1 ~]# vi structure.ldif
dn: dc=example,dc=com
dc: example
objectClass: top
objectClass: domain

dn: ou=people,dc=example,dc=com
ou: people
objectClass: top
objectClass: organizationalUnit

dn: ou=group,dc=example,dc=com
ou: group
objectClass: top
objectClass: organizationalUnit
```

Next we are going to import that ldif file using ldapadd command.  We are going
to authenticate as the admin user we created in the `config.ldif` file (cn=Manager),
so we need to enter that password that we created for that user.  This will create
the upper level containers
```
[root@server1 ~]# ldapadd -x -W -D "cn=Manager,dc=example,dc=com" -f structure.ldif
Enter LDAP Password:
adding new entry "dc=example,dc=com"

adding new entry "ou=people,dc=example,dc=com"

adding new entry "ou=group,dc=example,dc=com"
```

If we want to check if we have the entries in correctly we can do an ldapsearch
```
[root@server1 ~]# ldapsearch -x -W -D "cn=Manager,dc=example,dc=com" -b "dc=example,dc=com" -s sub "(objectclass=organizationalUnit)"
Enter LDAP Password:
# extended LDIF
#
# LDAPv3
# base <dc=example,dc=com> with scope subtree
# filter: (objectclass=organizationalUnit)
# requesting: ALL
#

# people, example.com
dn: ou=people,dc=example,dc=com
ou: people
objectClass: top
objectClass: organizationalUnit

# group, example.com
dn: ou=group,dc=example,dc=com
ou: group
objectClass: top
objectClass: organizationalUnit

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
```

### Creating Groups and Users in OpenLDAP

Now that we have our top level structures of our directory in place we can proceed with creating
groups and users.  We are going to create the groups first so that they are in place
and it is easy for us to add new users to those groups

We need to create a group.ldif file with the following contents (obtained from course files)
```
[root@server1 ~]# vi group.ldif
dn: cn=ldapusers,ou=group,dc=example,dc=com
objectClass: posixGroup
cn: ldapusers
gidNumber: 4000
```
In the above config we are creating a group called `ldapusers` (`cn=ldapusers`).
It is being created in the `group` container (`ou=group`) within domain `dc=example`.
The object class is posixGroup and this time because everything does inherit from "top"
it is unneccessary to add that in, it's implied.
The common name (`cn`) is ldapusers.
We have to have a `gidNumber`.  We are starting with 4000 as the starting group id number
just to be pretty clear and distinct from the other group ids that we created on our local system.

To create the group specified above, we run this command, entering the password of the
ldap admin user (`cn=Manager`)
```
[root@server1 ~]# ldapadd -x -W -D "cn=Manager,dc=example,dc=com" -f group.ldif
Enter LDAP Password:
adding new entry "cn=ldapusers,ou=group,dc=example,dc=com"
```

Having created our group now we can go ahead and create our users.  Users have a little
bit more properties than groups.  We can use migration tools to held add in new users on our
system into LDAP, but based on the structure of existing users and they will add in the
structure that we need, but we just have to make sure that is set correctly with
the configuration file.

First, edit the `/etc/share/migrationtools/migratio_common.ph` file and change `DEFAULT_MAIL_DOMAIN`
and `DEFAULT_BASE` to the following values;
```
# Default DNS domain
$DEFAULT_MAIL_DOMAIN = "example.com";

# Default base
$DEFAULT_BASE = "dc=example,dc=com";
```

Next we are going to grab an existing user, `centos` from our `/etc/passwd` and just
copy the details of that user into a new file `~/passwd`
```
[root@server1 ~]# grep centos /etc/passwd > passwd
[root@server1 ~]# cat passwd
centos:x:1000:1000:centos:/home/centos:/bin/bash
```

Then we can create a user.ldif template file based off the centos user using the
migrate_passwd.pl perl script
```
[root@server1 ~]# /usr/share/migrationtools/migrate_passwd.pl passwd user.ldif
```

Now that we have a user.ldif template we can use that to add in new users.  We just
need to edit the users.ldif file and replace the references to the centos user to the new
user.  Fields such as `dn`, `uid`, `cn`, `uidNumber`, `gidNumber`, `homeDirectory` and `gecos`
need to be amended to reflect the new user.

Then we can use the ldapadd command to add the user based of the user.ldif template.
```
[root@server1 ~]# ldapadd -x -W -D "cn=Manager,dc=example,dc=com" -f user.ldif
Enter LDAP Password:
adding new entry "uid=fred,ou=People,dc=example,dc=com"
```

We have our directory structure up and running.  Now that we our LDAP server
set up we can move on and demonstrate how to set up authentication against an OpenLDAP
server.

## Implementing OpenLDAP authentication in CentOS7

### Install and Configure the OpenLDAP client

We are going to use a separate server to the one that we installed the LDAP Server
on.  Log on to server2

First we are going to append an entry into our /etc/hosts file for the server where
the LDAP Server is installed, which in our case is server1.example.com.  Ping it
by it's hostname to ensure we can hit it.
```
[root@server2 ~]# echo "192.168.99.107 server1.example.com" >> /etc/hosts
[root@server2 ~]# ping server1.example.com
PING server1.example.com (192.168.99.107) 56(84) bytes of data.
64 bytes from server1.example.com (192.168.99.107): icmp_seq=1 ttl=64 time=0.984 ms
64 bytes from server1.example.com (192.168.99.107): icmp_seq=2 ttl=64 time=0.637 ms
```

Next we are going to install oddjob-mkhomedir to ensure our home directories get created at
login time.  When it is installed we enable and start the oddjobd service
```
[root@server2 ~]# yum install oddjob-mkhomedir
root@server2 ~]# systemctl start oddjobd
[root@server2 ~]# systemctl enable oddjobd
Created symlink from /etc/systemd/system/multi-user.target.wants/oddjobd.service to /usr/lib/systemd/system/oddjobd.service.
[root@server2 ~]# systemctl status oddjobd
● oddjobd.service - privileged operations for unprivileged applications
   Loaded: loaded (/usr/lib/systemd/system/oddjobd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2021-03-28 12:23:54 IST; 12s ago
 Main PID: 1502 (oddjobd)
   CGroup: /system.slice/oddjobd.service
           └─1502 /usr/sbin/oddjobd -n -p /var/run/oddjobd.pid -t 300
```

Then we are going to install some openldap client tools
```
[root@server2 ~]# yum install openldap-clients nss-pam-ldapd
```

Once installed we need to configure the client.  There are two ways
METHOD 1 : `authconfig-tui`
This is a terminal menu-system tool that allows you to configure your ldap client.
On the first screen you need to enable "Use LDAP" and "Use LDAP Authentication" in
addition to what is already enabled on that screen.
On the second screen, don't enable "Use TLS" but change the Server to "ldap://server1.example.com"
Keep the Base DN to what it is "dc=example,dc=com".  Click on Ok to save the settings and
exit the `authconfig-tui` tool
```
[root@server2 ~]# authconfig-tui
```

METHOD 2 : `authconfig`
This is a command line tool that does the same as the tool above, but without the
nice menu system
```
[root@server2 ~]# authconfig --enableldap --ldapserver=server1.example.com --ldapbasedn="dc=example,dc=com" --enablemkhomedir --update
```

If we grep the `/etc/nsswitch.conf` file we will see that `passwd` (the user DB) has been
enabled for ldap
```
[root@server2 ~]# grep passwd /etc/nsswitch.conf
#passwd:    db files nisplus nis
passwd:     files sss ldap
```

If we look at passwd using the `getent` tool, we'll see the user we created via ldap at
the bottom of the list
```
[root@server2 ~]# getent passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
...
...
fred:x:4000:4000:fred bloggs:/home/fred:/bin/bash
```

Finally, if we login as this new user, fred, we will see that it's home directory
will be created and if we run the `id` command we will see that it is part of the
ldapusers group we created earlier.
```
[root@server2 ~]# su - fred
Creating home directory for fred.
[fred@server2 ~]$ id
uid=4000(fred) gid=4000(ldapusers) groups=4000(ldapusers) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

### Listing Users and Groups

We've set up the LDAP client on server2, but not on server1 (which is where the LDAP
server is installed).  This means that we'll be able to easily list our ldap users and
groups on server2 using the standard posix tools such as `getent`, but not on server1.

On the server that the LDAP client is installed, you can run `getent passwd`
and you'll be able to see users that are in the LDAP database.  The same thing with
`getent group`, you'll be able to see the LDAP groups that have been creatd.
If you can see these users in the `/etc/passwd` file on a server, it means that those
users can login to those servers.

This capability has been setup through the name service switch file.  If you grep
the /etc/nsswitch.conf file for "ldap" you can see what system entities have been
enabled for ldap
```
[root@server2 ~]# grep ldap /etc/nsswitch.conf
passwd:     files sss ldap
shadow:     files sss ldap
group:      files sss ldap
netgroup:   files sss ldap
automount:  files ldap
```

If we don't want ldap enabled for some of these we can turn it off by editing the
nsswitch.conf file and removing ldap for the relevant entities.  You might want to
do this because the more entities that have information stored in ldap, the more lookups
will be made against ldap.

IF you were to login to server1 and run `getent passwd` or `getent group`, you will
not see the ldap users or groups on that machine, only your local users and groups.
This is because it has not been set up as an LDAP client.  As such, LDAP users
will not be able to login to server1.  This is best practice behaviour as standard users
should not be able to login to domain controllers.

### Search the Directory using the OpenLDAP client

If you want to query the LDAP directory from a machine with the LDAP client, you
don't actually need to login to the LDAP server, you can just run a query.

To initiate an ldapsearch.  We can specify the base that we want to search (`-b`)
which in our case is "dc=example,dc=com". The response from this command includes a lot of
metadata
```
[root@server2 ~]# ldapsearch -x -H ldap://server1.example.com -b dc=example,dc=com
# extended LDIF
#
# LDAPv3
# base <dc=example,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# example.com
dn: dc=example,dc=com
dc: example
objectClass: top
objectClass: domain

# people, example.com
dn: ou=people,dc=example,dc=com
ou: people
objectClass: top
objectClass: organizationalUnit

# group, example.com
dn: ou=group,dc=example,dc=com
ou: group
objectClass: top
objectClass: organizationalUnit
...
...
# search result
search: 2
result: 0 Success

# numResponses: 6
# numEntries: 5
```

By adding the `-LLL` flag to our command we will just get back the data from our
search, minus the metadata.
```
[root@server2 ~]# ldapsearch -x -LLL -H ldap://server1.example.com -b dc=example,dc=com
dn: dc=example,dc=com
dc: example
objectClass: top
objectClass: domain

dn: ou=people,dc=example,dc=com
ou: people
objectClass: top
objectClass: organizationalUnit

dn: ou=group,dc=example,dc=com
ou: group
objectClass: top
objectClass: organizationalUnit

dn: cn=ldapusers,ou=group,dc=example,dc=com
objectClass: posixGroup
cn: ldapusers
gidNumber: 4000

dn: uid=fred,ou=people,dc=example,dc=com
uid: fred
cn: fred
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JDFNTi5lNEkvSC81dzZDUWckUDllN29RdWI5dm0vNnBTQWFvZU9
 rbDUxTUMyWnNVbU5pbkNrTmoyMXI2OU1pRGQuUWJpeDhhM0FEdjQuUGxZcGRrVXp1TzBPQVlGT1Jv
 LkFteHIzVjE=
shadowLastChange: 18707
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 4000
gidNumber: 4000
homeDirectory: /home/fred
gecos: fred bloggs
```

If you just want to return users for your search (identified by objectclass=account)
```
[root@server2 ~]# ldapsearch -x -LLL -H ldap://server1.example.com -b dc=example,dc=com "(objectclass=account)"
dn: uid=fred,ou=people,dc=example,dc=com
uid: fred
cn: fred
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JDFNTi5lNEkvSC81dzZDUWckUDllN29RdWI5dm0vNnBTQWFvZU9
 rbDUxTUMyWnNVbU5pbkNrTmoyMXI2OU1pRGQuUWJpeDhhM0FEdjQuUGxZcGRrVXp1TzBPQVlGT1Jv
 LkFteHIzVjE=
shadowLastChange: 18707
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 4000
gidNumber: 4000
homeDirectory: /home/fred
dn: uid=sally,ou=people,dc=example,dc=com
gecos: fred bloggs
```

You can combine filter conditions using the `&`.  You can also use `|` to
create an OR condition
```
root@server2 ~]# ldapsearch -x -LLL -H ldap://server1.example.com -b dc=example,dc=com "(&(objectclass=account)(uid=fred))"
dn: uid=fred,ou=people,dc=example,dc=com
uid: fred
cn: fred
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JDFNTi5lNEkvSC81dzZDUWckUDllN29RdWI5dm0vNnBTQWFvZU9
 rbDUxTUMyWnNVbU5pbkNrTmoyMXI2OU1pRGQuUWJpeDhhM0FEdjQuUGxZcGRrVXp1TzBPQVlGT1Jv
 LkFteHIzVjE=
shadowLastChange: 18707
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 4000
gidNumber: 4000
homeDirectory: /home/fred
gecos: fred bloggs

[root@server2 ~]# ldapsearch -x -LLL -H ldap://server1.example.com -b dc=example,dc=com "(&(objectclass=account)(uid=fred))" uidNumber uid
dn: uid=fred,ou=people,dc=example,dc=com
uid: fred
uidNumber: 4000
```

If you just want certain attributes to be returned for your search, you can list the attributes you want
```
[root@server2 ~]# ldapsearch -x -LLL -H ldap://server1.example.com -b dc=example,dc=com "(&(objectclass=account)(uid=fred))" uidNumber uid
dn: uid=fred,ou=people,dc=example,dc=com
uid: fred
uidNumber: 4000
```

Finally, if you want to use the output of a search as a template to create new users, redirect the
search results to an `ldif` file, then edit that ldif file and put in your new user's details,
and then add the user to LDAP using `ldapadd`.
```
[root@server2 ~]# ldapsearch -x -LLL -H ldap://server1.example.com -b dc=example,dc=com "(&(objectclass=account)(uid=fred))" > newuser.ldif
[root@server2 ~]# vi !$
vi newuser.ldif
[root@server2 ~]# ldapadd -x -W -D cn=Manager,dc=example,dc=com -f newuser.ldif
Enter LDAP Password:
adding new entry "uid=sally,ou=people,dc=example,dc=com"
```

If you run `getent passwd` you can see your new user.  You can now login as the
new user
```
[root@server2 ~]# getent passwd
root:x:0:0:root:/root:/bin/bash
...
...
fred:x:4000:4000:fred bloggs:/home/fred:/bin/bash
sally:x:4001:4000:sally bloggs:/home/sally:/bin/bash
```

## Implementing Kerberos Authentication

### Configure NTP

Ensuring that Kerberos servers and clients are in time sync is important and to that
end we will install NTP on both server1 and server2.  We will configure server1
to get time from an external time source and then we configure server2 to collect time
from the server1 machine.

```
[root@server1 ~]# yum install -y ntp
```

Once installed we can configure it's config file.  We want to ensure that machines on the
same network can access our this server for NTP pursoses.  The NTP config is located at
`/etc/ntp.comf`.  We want uncomment the restrict directive for the local network and change
the subnet mask.  This will allow other machines to query the service and we can set up
peer synchronization.
```
[root@server1 ~]# vi /etc/ntp.conf
...
# Hosts on local network are less restricted.
restrict 192.168.99.0 mask 255.255.255.0 nomodify notrap
```

Then we can enable and start the `ntpd` service
```
[root@server1 ~]# systemctl enable ntpd
Created symlink from /etc/systemd/system/multi-user.target.wants/ntpd.service to /usr/lib/systemd/system/ntpd.service.
[root@server1 ~]# systemctl start ntpd
[root@server1 ~]# systemctl status ntpd
● ntpd.service - Network Time Service
   Loaded: loaded (/usr/lib/systemd/system/ntpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2021-03-28 15:39:01 IST; 4s ago
  Process: 5292 ExecStart=/usr/sbin/ntpd -u ntp:ntp $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 5293 (ntpd)
   CGroup: /system.slice/ntpd.service
           └─5293 /usr/sbin/ntpd -u ntp:ntp -g

Mar 28 15:39:01 server1.example.com ntpd[5293]: Listen normally on 2 lo 127.0.0.1 UDP 123
...
```

We can run an ntpq query to ensure we're up and running.  The server with the * is
the server we're currently synchronizing with.
```
[root@server1 ~]# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
+stratum2-3.ntp. 129.70.137.82    2 u    2   64    1   44.346    0.545   6.999
*ec2-52-17-30-11 89.101.218.6     2 u    1   64    1   15.390   -3.863   1.088
+chris.magnet.ie 88.81.100.130    2 u    -   64    1   14.422   -0.832   7.032
```

Then we need to add a rule to our firewall to allow NTP traffic in
```
[root@server1 ~]# firewall-cmd --add-service=ntp --permanent
success
[root@server1 ~]# firewall-cmd --reload
success
```

Then we need to add an entry to our `/etc/hosts` file for server2, so our hosts
file should have the following
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.99.107 server1 server1.example.com
192.168.99.105 server2 server2.example.com
```

On server2, we need to do some very similar setup, other than the fact that the NTP
server we are syncing up with is our server1 machine.

First we install ntp
```
[root@server2 ~]# yum install -y ntp
```

Then we chance the ntp configuration, and change the server section to that it
points to our server1 machine as the prefered machine (denoted by `prefer`), and
that we have left just one external ntp server as a backup if server1 is unavailable
for any reason. `iburst` just means to synchronize quickly.
```
[root@server2 ~]# vi /etc/ntp.conf
...
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server server1.example.com iburst prefer
server 3.centos.pool.ntp.org iburst
...
```

Then we enable and start the ntp service on server2
```
[root@server2 ~]# systemctl enable ntpd
Created symlink from /etc/systemd/system/multi-user.target.wants/ntpd.service to /usr/lib/systemd/system/ntpd.service.
[root@server2 ~]# systemctl start ntpd
[root@server2 ~]# systemctl status ntpd
● ntpd.service - Network Time Service
   Loaded: loaded (/usr/lib/systemd/system/ntpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2021-03-28 15:44:02 IST; 5s ago
  Process: 4284 ExecStart=/usr/sbin/ntpd -u ntp:ntp $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 4285 (ntpd)
   CGroup: /system.slice/ntpd.service
           └─4285 /usr/sbin/ntpd -u ntp:ntp -g
...
```

We don't need to make any changes to the firewall settings, there will be no
incoming ntp traffic to this machine.

Finally if we do an ntp query, we can see that the server we are synced to for NTP
purposes is our server1 machine
```
[root@server2 ~]# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*server1.example 52.17.30.119     3 u   10   64    1    0.487   11.327   0.067
 ec2-52-17-231-7 193.120.142.71   2 u    9   64    1   15.347   -1.422  16.045
 ```

 ### Install and Configure Kerberos KDC (Key Distrubution Center)

This involves installing the kadmin server and the KDC on server1

The first thing we need to do is install a Random Number Generator for KDC
```
[root@server1 ~]# yum install -y rng-tools.x86_64
```

Then we need to start the rngd service.  Before we do that, we need to edit the
rngd.service file and make sure it uses the /dev/urandom device.
```
[root@server1 ~]# vi /usr/lib/systemd/system/rngd.service

[Service]
ExecStart=/sbin/rngd -f -r /dev/urandom
```

Then we can start and enable the rngd service
```
[root@server1 /usr/lib/systemd/system]# systemctl start rngd
[root@server1 /usr/lib/systemd/system]# systemctl status rngd
● rngd.service - Hardware RNG Entropy Gatherer Daemon
   Loaded: loaded (/usr/lib/systemd/system/rngd.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2021-03-31 21:05:11 IST; 11s ago
 Main PID: 1916 (rngd)
   CGroup: /system.slice/rngd.service
           └─1916 /sbin/rngd -f -r /dev/urandom
...
[root@server1 /usr/lib/systemd/system]# systemctl enable rngd
```

Now we can do our kerberos installation, installing the kerberos server, client and PAM modules
```
[root@server1 /usr/lib/systemd/system]# yum install -y krb5-server krb5-workstation.x86_64 pam_krb5
```


The Kerberos server config is located in /var/kerberos/krb5kdc directory, and if we cat the two files
in that dir, we can see that they are set up to use EXAMPLE.COM as the realm.  This is fortunate
for us because example.com is the domain we are using, so we don't need to make any changes here.
```
[root@server1 /var/kerberos/krb5kdc]# cat kadm5.acl
*/admin@EXAMPLE.COM	*

[root@server1 /var/kerberos/krb5kdc]# cat kdc.conf
[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 EXAMPLE.COM = {
  #master_key_type = aes256-cts
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
 ```

The Kerberos client config is located in /etc/krb5.conf
```
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
# default_realm = EXAMPLE.COM
 default_ccache_name = KEYRING:persistent:%{uid}

[realms]
# EXAMPLE.COM = {
#  kdc = kerberos.example.com
#  admin_server = kerberos.example.com
# }

[domain_realm]
# .example.com = EXAMPLE.COM
# example.com = EXAMPLE.COM
```

We need to edit this file and uncomment the default_realm, the realms block and
the domain_realm block.  And we need to change the kdc and admin_server to point at
server1.example.com.  The file should look like the following.
```
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
 default_realm = EXAMPLE.COM
 default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 EXAMPLE.COM = {
  kdc = server1.example.com
  admin_server = server1.example.com
 }

[domain_realm]
 .example.com = EXAMPLE.COM
 example.com = EXAMPLE.COM
```
