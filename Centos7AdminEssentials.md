# Learning the Essentials of CentOS Enterprise Linux 7 Administration - Course Notes

Path to pluralsight course : https://app.pluralsight.com/library/courses/lfcs-red-hat-7-essentials/table-of-contents

## General Linux Commands

To get info regarding your linux Version
```
lsb_release -d
```

Info regarding file systems

* `df` - disk free info
* `df -h` - disk free info with sizes in human readable form
* `df -T` - also displaying the filesystem TYPE
* `mount` - displays the mounted file systems

Check your IP settings
```
ip a s
```

To check the status of your network connections
```
nmcli conn show
```

Activate a network connection (where the connection in question is enp0s3)
```
nmcli conn up enp0s3
```

Search for a string and replace it, in a given file
```
sed -i s/ONBOOT=no/ONBOOT=yes/ /etc/sysconfig/network-scripts/ifcfg-enp03
sed -i s/ONBOOT=no/ONBOOT=yes/ /etc/sysconfig/network-scripts/ifcfg-enp08
```

Search for a string using the last argument used in a previous command
```
grep ONBOOT !s
```

## Consoles

* Physical TTY
* Local Pseudo TTY
* Remote Pseudo TTY (ssh from another machine)

When logging on, usernames and passwords are case sensitive
When using Centos with a graphical environment, the login screen you see in that environment is called a "display manager"

In the Centos7 gui, you can launch a terminal window by right-clicking on the desktop and select Open Terminal,
or by clicking the Terminal Icon in the top bar

`CTRL/Shift/+` to increase the font size of the terminal window
`CTRL/-` to decrease the font size

To exit the terminal window, use `CTRL/D` or type exit.

To switch to the physical console hit the Right CTRL key and F2.  To switch back to the graphical terminal hit the right CTRL key and F1.

To return to your home directory
```
cd
```
To see what terminal you are currently using
```
tty
```
To see who is currently logged in and what terminal they are logged into
```
who
```

## WORKING WITH FILES AND DIRECTORIES

### Listing files

List files in the current directory (not including hidden files)
```
ls
```

List files in the current directory (not including hidden files), displayed in a list.  It shows permissions, number of hard links, group ownership, size, and date last modified
```
ls -l
```

As above but with size displayed in human readable format
```
ls -lh
```

List files in the current directory (not including hidden files), including their inode number (their entry in the file system)
```
ls -i
```

To show everything in a directory, including hidden files.  Hidden files/dirs always begin with a dot (.)
```
ls -a
```

Shows a list of files, sorting on the timestamp (t) in reverse order (r)
```
ls -ltr
```

Shows the file type. Directories will be shown with a forward slash after their name
```
ls -F
```
### File Types

Everything in Linux is represented as a File

* `-` : A regular file
* `d` : A directory
* `l` : Symbolic link
* `b` : A block device
* `c` : A character device file
* `p` : Named pipes
* `s` : Sockets

### File Management Commands

* `cp`
* `mv`
* `rm`


Options that can be used with above;

* `-i` : interactive mode
* `-r|R` : used for Recursion
* `*` : wildcard to group files together, eg rm *.txt.  * * means zero or more characters
* `?` : represents a single character
* `[]` : We can put groups of characters in the square brackets

### Directory Management Commands

Make a directory
```
mkdir mydir1
```

Make a nest of directorys, creating the parent one on the way
```
mkdir -p mydir2/subdir2
```

Make a directory and set the permissions at creation time
```
mkdir -m 777 mydir3
```

To make a dir and set the sticky bit. The `sticky bit` is a permission bit that is set on a file or a dir that lets
only the ownder of the file/dir or the root user to delete or rename a file. No other user is given privileges to delete the file created by another user
```
mkdir -m 1777 mydir4
```

Remove a directory
```
rmdir mydir1
```

Remove a directory and it's contents recursively
```
rm -rf mydir1
```

Copy a directory and it's contents recursively to another location
```
cp -R sales /tmp
```

### HARD LINK COUNT

Can be used to count the number of subdirectorys in a directory (total hard link count -2). Hard link count includes the dot file (.) and the dot dot file (..). The `dot` file represents the metadata entry for the current directory. The `dot dot` file represents the metadata entry for the parent directory

To get the details (not the contents) of a directory
```
ls -ld mydir1
```

Create a link (can only link files in the same file system)
```
ln file1 file2
```

Create a soft link or symbolic link (can link files in different file systems)
```
ln -s sourcefile linkfile
```

## Reading from files

Concatenate command, you can give it two file arguments and it will concatentate the contents into one output.
```
cat file1 file2
```

It you give it one file it will just output that file
```
cat file1
```

The `less` command can be used to page through larger files
```
less file1
```

When in less mode, you can use `/` to search forwards and `?` to search backwards using the text we want to search for

To read the first few lines of a file, and the first 10 lines of a file
```
head
head -n 10
```
// To read the last few lines of a file, and the last 10 lines in a file
```
tail
tail -n 10
```

Search through files
```
grep
```

Using enhanced grep, where you can set regular expressions to search for
```
grep -E
```

* `^` : Begin with
* `$` : Ends with
* `^$` : Empty lines

Install the words package, which is a dictionary of all words
```
sudo yum install -y words
```

Will search for words in the dictionary that have 5 consecutive vowels in them.  The square brackets is Or'ing the options in the brackets
```
grep -E '[aeiou]{5}' /usr/share/dict/words
```

How to check an aliase on a command, `type <cmd>`;
```
type cp
type ls
```


Sed command, used for searching, find and replace, insertion and deletions
```
sed
```

Create a function that lives in memory, using sed.  After you have created the function in memory, the function cam be run to remove comments and empty lines from the target file (`hosts`, in this example)
```
$ function clean_file {
   sed -i '/^#/d;/^$/d' $1
}
clean_file ./hosts
```

In place edit, used for find and replace, insertion and deletions
```
sed -i
```

Compare text files
```
diff file1 file2
diff ntp.conf /etc/ntp.conf
```

Compare binary files, will output a checksum of the binary file
```
md5sum
md5sum /usr/bin/passwd
```

Verify a package, it compares the checksum of files in your package against a central DB to see if the checksums match
```
rpm -V ntp
```
Searching for actual files.  Will look for files of type Link in the current dir, or level of directory below
```
find /etc -maxdepth 1 -type l
```

Will look for files of type File in the current dir, greater than 10mb in size
```
find /boot -type f -size +10000k
```

Will search for all pdf files in the /usr/share dir and copy them to the current directory
```
find /usr/share -name '*.pdf' -exec cp {} . \;
```

Get detailed information relating to a file called file1
```
stat file1
```

## VIM Text Editor

Will create an empty file called file1, this will change the access time and the modification time
```
touch file1
```
Will also do the same thing as above, using STDOUT Redirection
```
> file
```
Will avoid creating a new file if the file doesn't exist already
```
touch -c file1
```

This will change the access time only
```
touch -a file1
```

This will change the modification time only
```
touch -m file1
```

To copy the modification and access times from one file to the other
```
touch file1 -r file2
```

To create a file with a specific timestamp
```
touch -t YYMMDDHHMM.SS file1
```

To change the timestamp of a file
```
touch -c -t YYMMDDHHMM.SS file1
```

Will change the access and modification time using a human readable date
```
touch -d '16 November 1973' file
```

To install the nano Editor
```
sudo yum install -y nano
```

A tutor program that will exercise various vi Commands
```
vimtutor
```

## PIPING AND REDIRECTION

Redirection of standard output, creating an empty file using standard output
```
> file
```

Writing the standard output (STDOUT) of the ls command to file1
```
ls > file1
```

Using the numermical designator for redirection to definitely redirect standard output, `1>` is redirection for standard output
```
ls 1> file1
```
To append to the file instead of overwriting it
```
ls 1>> file1
```

To prevent accidental overwrite, you can set the noclobber option in bash to prevent overwrites
```
set -o noclobber
```

If you then want to forceibly overwrite a file you can then do so using the following
```
ls >| file1
```

Redirecting STDERR output. The following command will return standard output and standard error output. Some of the files returned by the find command will have a permission denied error, which is STDERR. The files that we have permission to read will be STDOUT
```
find /etc -type l
```

To redirect just the errors we can redirect the STDERR to /dev/null, which is like a black hole for data
```
find /etc -type l 2> /dev/null
```

To redirect STDOUT and STDERR to the same file
```
find /etc -type l &> file1
```

Reading from files using STDIN, first run a command to get the free diskspace in the filesystem and write it to a file
```
df -hlT > diskfree
```

Then we can use the mail command to send a mail to ourselves but using the contents of the diskfree file as the mail body
```
mail -s "Disk free" centos < diskfree
```
### HERE Documents
```
cat > newfile <<END
> This is a new file
> that can be created on the
> command line or scripts
> END
```
In the above case the we are using cat to redirect to newfile, reading in from a here document called END.
Each line will be read in until we read in the name of the here document, which is END


### Command pipelines
Piping the output of a command to another command
```
cut -f 7 -d : /etc/passwd | sort | uniq
```

### Named Pipes.
Used for interprocess communication.  Created with the mkfifo command
```
mkfifo mypipe
```
You can then redirect information to the pipe, this command sends the output of the ls command to the named pipe
```
ls > mypipe
```
In a separate process (can be simulated by logging in with a new bash session) you can process the data from the named pipe using the following
```
wc -l < mypipe
```

### tee
The `tee` command.  This is used to see the output while it is being redirected to a file.  If you are doing straight-up redirection, you won't see the output
```
ls | tee file1
```

## Archiving Files

### Using tar
* `tar -c` - Create a tar file
* `tar -t` - Looking inside a tar
* `tar -x` - Expanding a tar
* `tar -z` - To compress the tar file as well
use the `-f` option in addtion to the above to specify the file that we backing up to or expanding from

### Using Gzip and gunzip
Used to compresse and decompress zip files
`.gz` or `.tgz` are common file extentions used to denote compressed files

#### cpio
* `-i` - input mode
* `-o` - output mode
* `-d` - create the necessary directories when expanding

To create a CPIO archive file from the output of the find command
```
find /usr/share/doc -name '*.pdf' | cpio -o > /tmp/pdf.cpio
```

To expand a cpio file into your current dir //
```
cpio -id < /tmp/pdf.cpio // Input mode
```
### dd command
Usage to make images of disks

* `if` - input file
* `of` - output file

To make an image of the disk loaded into a CD ROM drive
```
dd if=/dev/sr0 of=/tmp/disk.iso
```

To backup the master boot record //
```
dd if=/dev/sda of=/tmp/sda.mbr count=1 bs=512
```

## ACCESSING COMMAND LINE HELP
`-h` `--help` `/?`

To get help for the `ls` command
```
ls -h
ls --help
```
### man pages
`man <command>`
```
man ls
man pwd
```

To find out what the man sections are then run the man command on man itself
```
man man
```

To access a particular section of a man page for a command, you can specify the section number
```
man 5 ls
```

### Info pages
eg. `info ls`

Info pages contain other sections that are linked in the main page.  Go down to the menu item and hit return on the menu item you want to access.  To return to the previous level use the "u" key

### RCS (Revision Control System)
You can maintain multiple versions of files using RCS
ci // command is used to check in a file
ci -f // To force checkin a file
co -l // is used to checkout and lock the file (so nobody else can access it)
co -l -r1.1 hello.sh // to check out a specific version of a file
rlog -b // to get a list of versions of a particular file

## UNDERSTANDING FILE PERMISSIONS
In general linux file systems will support permissions, however FAT based file
systems do not.

Default file permissions : 666
Default directory permissions : 777

// Linux ACLs //
Additional permissions can be added via ACLs. In the default XFS file system of
centos 7 this is built-in.  In EXT based file systems the mount option "acl" needs
to be added

// Listing permissions //
ls -l <file name>

-r-xr-xr-x. 1 centos centos 522 Feb 22 17:16 hello.sh

Permissions are displayed like the following;
-r-xr-xr-w

This can be displayed in octal format - 0555
r = 4
w = 2
x = 1

The leading dash tells us it's a regular file.  For other file types you could see
different characters there, eg. "c" for character type file, "b" for block file

The next 9 characters are divided up into 3 blocks of 3.  The first 3 chars relate
to "user".  The next 3 relate to "group" and the last 3 relate to "other".

In a block of 3 characters, such as r-x, the first character (r) is for read permissions,
the second character (w) for write permissions, and the last character (x) is for execute permissions

If one of these characters is hyphened (-), that means that the the permission is not present.

After the file permissions block is shows the name of the user that owns the file (centos),
and the name of the group (centos).

stat hello.sh,v // Gives more indepth data on a file and it's permissions

  File: ‘hello.sh,v’
  Size: 522       	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 508509      Links: 1
Access: (0555/-r-xr-xr-x)  Uid: ( 1000/  centos)   Gid: ( 1000/  centos)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2021-02-22 17:16:47.458505685 +0000
Modify: 2021-02-22 17:16:31.318061825 +0000
Change: 2021-02-22 17:16:31.318061825 +0000
 Birth: -

stat -c %A hello.sh,v // Will just show the symbolic permissions
-r-xr-xr-x

stat -c %1 hello.sh,v // Will just show the numeric permissions
555

// Managine default permissions with umask //

umask can be used to restrict permissions to the various groups (user, group, other)

//To find out what the current restrictions are
umask

This will return a numerical value. Example return vals are as follows;
2  // This removes write permissions from the others group
22 // Removes write permissions from the group group.
222 // Removes write permissions from user, group and others group

217 // Removes write permissions from user, execute from group, and all permissions from others.

umask u=rwx,g=rx,o=rx // This is a different way of specifying what permissions are allowed for each group.

// Using chmod to set permissions //
chmod 467 file1 // setting user read, group read/write, and others read/write/execute

chmod u=r,g=rw,o=rwx file1 // Doing the same as above but doing it symbolically

chmod u+x,g+r,o+w file4 // To add various permissions to different groups, symbolically

chmod a+x file4 // To add a permission to all groups at the same time

chmod a-wx file4 // To remove permissions from all groups

chmod o= file4 // Removing all permissions for the others group

chmod go-w file4 // Removing write permission from group and others

// setuid bit, setgid bit and the directory sticky bit //

Normally, on a unix-like operating system, the ownership of files and directories is
based on the default uid (user-id) and gid (group-id) of the user who created them.
The same thing happens when a process is launched: it runs with the effective user-id
and group-id of the user who started it, and with the corresponding privileges. This
behavior can be modified by using special permissions.

// setuid bit //

When the setuid bit is used, the behavior described above it's modified so that
when an executable is launched, it does not run with the privileges of the user
who launched it, but with that of the file owner instead. So, for example, if an
executable has the setuid bit set on it, and it's owned by root, when launched by
a normal user, it will run with root privileges. It should be clear why this
represents a potential security risk, if not used correctly.

An example of an executable with the setuid permission set is passwd, the utility
we can use to change our login password. We can verify that by using the ls command:

ls -l /bin/passwd
-rwsr-xr-x. 1 root root 27768 Feb 11  2017 /bin/passwd

How to identify the setuid bit? As you surely have noticed looking at the output
of the command above, the setuid bit is represented by an s in place of the x of
the executable bit. The s implies that the executable bit is set, otherwise you
would see a capital S. This happens when the setuid or setgid bits are set, but
the executable bit is not, showing the user an inconsistency: the setuid and setgit
bits have no effect if the executable bit is not set.

ONLY APPLIES TO FILES.  The setuid bit has no effect on directories.

// setgid bit //
Unlike the setuid bit, the setgid bit has effect on both files and directories.
In the first case, the file which has the setgid bit set, when executed, instead
of running with the privileges of the group of the user who started it, runs with
those of the group which owns the file: in other words, the group ID of the process
will be the same of that of the file.

When used on a directory, instead, the setgid bit alters the standard behavior so
that the group of the files created inside said directory, will not be that of the
user who created them, but that of the parent directory itself. This is often used
to ease the sharing of files (files will be modifiable by all the users that are
part of said group). Just like the setuid, the setgid bit can easily be spotted
(in this case on a test directory):

ls -ld test
drwxrwsr-x. 2 egdoc egdoc 4096 Nov  1 17:25 test

This time the s is present in place of the executable bit on the group sector.

// Directory Sticky Bit //
// To create a directory with the sticky bit set //
// The sticky bit is a permission bit that is set on a file or a dir that lets
// only the owner of the file/dir or the root user to delete or rename a file
// No other user is given privileges to delete the file created by another user
mkdir -m 1777 test
chmod o+t test // setting it symbolically

Output : drwxrwxrwt. 2 centos centos 6 Feb 22 21:27 test

the "t" in the others group indicates that the sticky bit is set on the dir

// To remove the sticky bit from a dir //
chmod o-t test // The sticky bit will revert to x

// File Ownership //
Files have owners, the user owner and the group owner
-rwx------. 1 centos centos 0 Feb 22 20:54 file4

In the above, centos is the name of user that owns the file, and the other "centos" is
the name of the primary group that the centos user belongs to.

id // When run by itself will give the full user and groud info for the current user
[root@server1 Documents]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

id -u // gives the id of the current user logged in
Output : 1000

id -un // gives the username of the current user logged in
Output : centos

id -g // gives the primary group id that the current user belongs to
Output : centos

id -gn // gives the primary group name that the current user belongs to
Output : centos

id -G // gives the secondary group id (as well as the primary) that the current user belongs to
Output : 1000 10

id -Gn // gives the secondary group name (as well as the primary) that the current user belongs to
Output : centos wheel

// Primary Group and Secondary Group //
The primary group is used to create new file resources.
The secondary/complimentary groups relate to accessing resources


// To switch your primary group (to use a secondary group for example)
// This actually creates a new bash shell, use exit to revert to previous shell
$ newgrp wheel
$ id -gn
wheel
$ touch newgroup
$ ls -l newgroup
-rw-r--r--. 1 centos wheel 0 Feb 23 11:37 newgroup

// To change the group for a file (you can only change the group for a file, if you are
// a member of that group)
$ chgrp wheel file2
$ ls -l file2
-rwxrwxrwx. 1 centos wheel 0 Feb 22 20:53 file2

// Changing file ownership //
chown root file2 // Making root the owner user of file2
chown centos.centos file2 // Changing the user owner and the group owner at the same time
chown centos:wheel file2 // Same idea as above, only we can use the colon as well

// Maintaining file ownership when copying files //
If you copy a file into the root directory (as the root user) the ownership of the
file will be the root user and the root group.

[root@server1 Documents]# cp file2 /root/file2na
[root@server1 Documents]# ls -l /root/file2na
-rwxr-xr-x. 1 root root 0 Feb 23 11:50 /root/file2na

If you use cp -a instead, it will copy the files with the original ownership
and timestamps in place
[root@server1 Documents]# cp -a file2 /root/file2a
[root@server1 Documents]# ls -l !$
ls -l /root/file2a
-rwxrwxrwx. 1 centos wheel 0 Feb 22 20:53 /root/file2a

## ACCESSING THE ROOT ACCOUNT

// su (switch user) command //
su // When used without a username it will assume you're trying to switch to the root user, you will be prompted for a pw

When using su, you need to have the password of the user you are switching to
Using su will create a new bash shell, use exit to revert to the previous shell

The su command without any flags will leave you in the current directory, and will
leave some environment variables as their previous value, such as USER.

[root@server1 Documents]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@server1 Documents]# echo $USER
centos

su - or su -l will take you to a full login shell.

This will take you to the root directory and set up the whole environment to run
as root.

[centos@server1 Documents]$ su -l
Password:
Last login: Tue Feb 23 12:05:40 GMT 2021 on pts/1
[root@server1 ~]# pwd
/root
[root@server1 ~]# echo $USER
root
[root@server1 ~]#

// sudo command //
The sudo command can be used to gain root privileges without the root password.
When you use sudo a command you will be prompted for your password.
Once you authenticate, the session lasts for 5 mins, so even if you sudo again in
that timeframe, you won't be prompted for a password again

// sudoers file //

Location : /etc/sudoers

Edit the sudoers command using the visudo command

The sudoers file is a file Linux and Unix administrators use to allocate system
rights to system users. This allows the administrator to control who does what.
Remember, Linux is built with security in mind. When you want to run a command
that requires root rights, Linux checks your username against the sudoers file.
This happens when you type the command “sudo”. If it determines, that your
username is not on the list, you cannot run the command/program logged in as that user.

Syntax of user permissions is as follows;
users hosts=(user:group) commands

// Examples //
root ALL=(ALL) ALL // Allow
Tim,Tammy,Roger,Dawn 192.168.0.6=(admins)/sbin/halt, /sbin/init, /sbin/poweroff, /sbin/reboot, /sbin/shutdown, /sbin/telinit

// Using Aliases in the sudoers file //

//Aliases can be used. Instead of listing Tim, Tammy, Roger, Dawn, a User_Alias can be created.
User_Alias      REBOOT_USERS = Tim, Tammy, Roger, Dawn

//Instead of listing halt and init and poweroff and reboot and shutdown and telinit, a Cmnd_Alias can be created.
Cmnd_Alias      REBOOT_COMMANDS = /sbin/halt, /sbin/init, /sbin/poweroff, /sbin/reboot, /sbin/shutdown, /sbin/telinit

//Instead of listing 192.168.0.6, a Host_Alias can be created.
Host_Alias      REBOOT_HOSTS = 192.168.0.6

//Instead of listing root:admins, a Runas_Alias can be created..
Runas_Alias     REBOOT_RUNAS = admins

//The aliases can then be used together
REBOOT_USERS       REBOOT_HOSTS=(REBOOT_RUNAS)REBOOT_COMMANDS

// Restricting root access to SSH //
In the /etc/ssh/sshd_conf file root login is enabled by default.

To disable root login by ssh, you need to make sure the PermitRootLogin setting
is uncommented, and the value is "no".  Once you change the file, you will need
to restart the sshd daemon with "systemctl restart sshd"

sudo sshd -t // Will ensure the sshd_conf file is properly formatted

Then if you try ssh in as root, it will still prompt you for a password, but even
if you provide a valid password, you will get an access denied message.

## Accessing Servers with SSH
When you ssh to a different server for the first time, a .ssh directory is Created
in the user's home directory (the user that ssh'd) and a known_hosts file is in that
folder.  When you successfully ssh to a different server an entry is place in the
known_hosts file for that server

ssh 192.168.99.105 // Will try ssh to the host using the user name of the current user and the default ssh port (22)

ssh centos@192.168.99.105 // Will try log in as the centos user
ssh -i <KEY FILE> centos@192.168.99.105
ssh -l centos 192.168.99.105

// To remove a host from known_hosts
ssh-keygen -R <hostname or IP Address>

You can add a configuration file to the .ssh dir to help speed things up.
First make sure you log in as the root user using su -l.  Then run "vi config" to
create a config file and in the config file.  In the config file we can add host
entries;

Host server2
HostName 192.168.99.105
User centos
Port 22

When you save the file, you can now run "ssh server2" and it will pick up the ssh
client config for that host from the .ssh/config file.

// to generate public-private key pair for ssh access //
ssh-keygen -t rsa // Will generate a public and private key for the current user, using rsa encryption

For the above command you will be asked for a passphrase for your private key, you
should provide one

Once you have a private key generated you can then distribute the public key to
the servers that you want to ssh with using key based authentication

// To copy your public key to other servers
ssh-copy-id -i ~/.ssh/id_rsa.pub server2 // This assumes that server2 is set up in the .ssh/config file

When you perform this command your public key gets added to the .ssh/authorized_keys file.

// Authenticate using your key //
ssh server2 // You will then be asked to provide the passphrase of the private key, and not the password of the user

// Pre-authenticate your private key //
ssh-agent bash // Opens up a new bash shell
ssh-add // Will add the private key for the current logged in user if it has been created.  You will need to enter the passphrase

ssh server2 // You will not need to enter your private key passphrase as you have pre-authenticated

// To ensure that you can only ssh as root on the target server using key auth //
You need to login to the target server.  Once on the target server login as root (su -l).
vi /etc/ssh/sshd_config

Ensure that the setting for "PermitRootLogin" is uncommented and has a value of "without-password"

Save the file and restart sshd using "systemctl restart sshd

// Copying files securely from one server to another //

// First pre-authenticate
[centos@server1 .ssh]$ ssh-agent bash
[centos@server1 .ssh]$ ssh-add
Enter passphrase for /home/centos/.ssh/id_rsa:
Identity added: /home/centos/.ssh/id_rsa (/home/centos/.ssh/id_rsa)

// Copy a file from my current server(server1) to server2 //
[centos@server1 .ssh]$ scp /etc/hosts server2:/tmp
hosts                                              100%  172    69.6KB/s   00:00

// Copy a file from server2 to my current server (server1) //
[centos@server1 .ssh]$ scp server2:/tmp/hosts .
hosts                                              100%  172    69.6KB/s   00:00

## USING SCREEN AND SCRIPT
Allows you to run commands on many systems simultaneously

// Using Script to record commands //
script // When entered by itself, it starts a recording session and saves the output to a file called typescript

To stop recording enter "exit".  Then you can view the entire recording by "cat typescript"

// Using script to monitor a user in real time //
mkfifo /tmp/mypipe // Create a named pipe that we will write our script recording to

script -f /tmp/mypipe // Start a script and write the recording to your named pipe

cat /tmp/mypipe // In a separate bash session logged in as a different user (say, root) cat the mypipe and you will see the script as it's being written to

// Screen command //
Screen can help you automatically set up multiple connections with other servers and you can toggle between then
You first need to create a .screenrc file in your home directory, like this;

[root@server1 ~]# cat .screenrc
screen -t server1 0 bash
screen -t s1 1 ssh server2

In the above set up, we have two connections set up.  The first one (numbered 0) is to the current machine
and it will invoke a bash shell

The second one (numbered 1) will set up a connection by invoking "ssh server2"
This assumes you have your ssh client set up with the server2 details, have enabled key based auth
with server2, and have pre-authenticated your bash session with "ssh-add"

Once the screen sessions have been established you can toggle between screens using;
CTRL-A and then press "n" to move to the next screen session.
CTRL-A and then press "p" to move to the previous screen session.
CTRL-A and then pressing the double quote button (") will show a menu of all the screen connections,
and you can select one and hit enter to move to it.

The real power of screen is that once you have multiple sessions set up simultaneously,
you can then run commands on the boxes simultaneously. To do this;

CTRL-A and the press ":".  A colon prompt will appear at the bottom left of the screen.  Then type;

:at "#" stuff "yum install -y zsh^M" .  The # represents all the screens we have.
The stuff command is used to load a command into the window buffer of the target screens.
In this case we want to install the zsh shell. The ^M at the end of the command is the end of line character.

To close an individual screen session, just type exit in each one until screen is fully terminated.
