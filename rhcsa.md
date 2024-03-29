## order of courses on pluralsight

* RHEL 8: Using Essential Tools
* RHEL 8: Creating Shell Scripts
* RHEL 8: Configuring Local Storage
* RHEL 8: Creating and Configuring File Systems
* RHEL 8: Operating Running Systems
* RHEL 8: Deploying, Configuring and Maintaining Systems
* RHEL 8: Managing Users and Groups
* RHEL 8: Managing Networking
* RHEL 8: Managing Security
* What's New In RHEL 9
* Getting Started with Podman


# RHEL 8: Using Essential Tools

# introducing red hat and enterprise linux

## red hat linux family
* RHEL 8: free but requires support subscription for updates (via developers.redhat.com)
* centos 8: red hat rebuild with community support and updates, project ended December 2021.
* centos stream: tracking ahead of RHEL centos stream becomes a dev platform for RHEL

## installing RHEL 8

* this module is a summary of installing rhel8 via virtualbox.
* since I already went through this on the RHCE training course before starting this course, I'll be using vagrant, which is covered in a later module.  You can skip ahead then come back.
* I'll document the modules in order, so I'll now proceed to "Understanding Linux Commands."


## Understanding Linux Commands
* Please subscribe to RobertElderSoftware on youtube and watch his shorts. https://www.youtube.com/shorts/SHI0TL415R8
* also check out explainshell.com

1. access the VM via ssh
```
vagrant ssh rhel8
```

2. you can run commands, they are case sensitive.
```
ls
```

3. you can run commands with arguments
```
ls -a
```

4. you can run commands with specific arguments
```
ls -l /etc/
```

5. invoking with `--help` is generally accepted as the argument to return help info on a command.  `man` will call documentation as well
```
ip addr help
man ip
man man #this will return help on the `man` command
ls /usr/share/doc #additional info
```

## execute commands in linux (lab)

* keep in mind that I installed with vagrant in the next exercise before completing this exercise.

1. login
```
vagrant ssh rhel8
```

2. gather info on the host
```
hostnamectl
```

3. clear the screen with a hotkey
```
hit ctrl+L
```

4. obtain the ip address
```
ip -4 addr show
```

5. show socket status
```
ss -ntl #show listening tcp ports and validate that port 22 is listening
```

## using vagrant (to manage) virtual machines

* using virtualbox with vagrant allows a far simpler approach to VM management and can allow someone else to install RHEL.

### summary
* on host OS
* create the required directories in the host file system
* create/download the Vagrantfile

1. sign up for redhat developer subscription
* https://developers.redhat.com/

2. install vagrant
* https://developer.hashicorp.com/vagrant/downloads and install

3. install virtualbox
* https://www.virtualbox.org/wiki/Downloads

4. create directories (in pwsh)
```
mkdir -f ~/vagrant/rhce
cd ~/vagrant/rhce
```

5. download rhel8 and start a VM
```
vagrant init generic/rhel8
vagrant up
```

6. connect via vagrant ssh
```
vagrant status
vagrant ssh
```

7. register with redhat
```
# there is no longer a need to use auto-attach as simple content access is enabled by default on new dev subscriptions
sudo subscription-manager register --username mbrowndc #this takes a lot longer than you think it should (~90 seconds)
# validate registration
sudo subscription-manager status
+-------------------------------------------+
   System Status Details
+-------------------------------------------+
Overall Status: Disabled
Content Access Mode is set to Simple Content Access. This host has access to content, regardless of subscription status.

System Purpose Status: Disabled
```

# working with text files

## understanding shell redirection and redirection from the CLI (lab)

1. create a file
```
touch file1
```

2. get info on file
```
file file1
```

3. add text with redirection
```
echo hello > file1
```

4. get info on file
```
file file1
```

### understand named pipes

* there are more named pipes than stdout and stderr

1. stdout (redirect standard output with `>`)
```
ls -l /etc/hosts #by default stdout is directed to the console
```

2. stderr (redirect standard error with `2>`)
```
ls -l /etc/foo #by default stderr is also directed to the console
```

### redirect output

1. stdout
```
ls -l /etc/hosts > stdoutput #outputs stdout to the file `stdoutput`
ls -l /etc/hosts >> stdoutput #outputs stdout to the file `stdoutput`, appending the output of this command to the file
```

2. stderr
```
ls -l /etc/Hosts 2> stderror #outputs stderr to the file `stderror`
ls -l /etc/Hosts 2>> stderror #also works
```

3. stdout + stderr
```
ls -l /etc/hosts /etc/Hosts 2>&1 &> stdoutputerror #this directs... stderr to stdout, then directs all output to the file `stdoutputerror`.  yea, you could just use `&>``
```

### redirect input
* in a herestring
```
cat > story.txt << END
line 1
line 2
END
```

## using `tee` for redirection

* `tee` is used to duplicate redirection of output

1. fails since shell redirection will redirect output of `sudo` command, not the output of `echo`
```
sudo echo "1.0.0.1 cloudflare" >> /etc/hosts
```

2. succeeds since the `|` passes stdin to the `sudo` command, which will pass to the invoked command `tee`
```
echo "1.0.0.1 cloudflare" | sudo tee -a /etc/hosts
```

## understanding text editors / editing files using nano / using the vim text editor

* disclaimer: `nano` upsets me (not as much as emacs), so I'm not installing it.

* common linux text editors
  * nano: simple to use and little experience is needed
  * vim: powerful text editor but more time is needed in learning the editor (see https://mbrownnyc.wordpress.com/misc/qvm/ for a quick cheat sheet, and https://tryhackme.com/room/toolboxvim )

* basic software package management introduction
1. list software repositories
```
sudo yum repolist
```

2. no nano! >:|
```
sudo yum install -y vim bash-completion
```

3. note that if you need to access a file with restricted permissions, you must invoke the editor command via `sudo`
```
sudo vim /etc/hosts
```

## working with directories

* `pwd`: print working directory
* `cd`: change working directory, if invoked without args, it will return to your homedir (as if you invoked `cd ~`)
  * `cd -`: change to previous directory

1. create a directory
```
cd
mkdir my_directory
mkdir ~/my_directory
```

2. create all directories in a structure
```
cd
mkdir -p dir1/dir2
```

3. remove directory with files/dirs
* remove an empty directory: `rmdir`.  this will fail if there are files within that directory.
```
rm -rf dir1
```

## file operations (copy, move, delete)

### globbing

* wildcards, like * (any amount of any char) and ? (single char)

### "ranges"
* commands accept `{N..Nx}`

1. create 12 files 
```
touch foo{1..12}
```
  
1. copy a file
```
cd
cp /etc/hosts .
cat ~/hosts
```

2. moving a file
```
mv /etc/hosts . #fails because you can't delete the file at source
```

3. delete a file
```
cd
mkdir test
cd test
rm files??
fm -i files?
```

## reading text files

* `cat`: read entire contents
* `head`: reads number of specified lines from beginning of file
* `tail`: reads number of specified lines from end of file
* `less`: page through files
* `grep`: search file contents

1. read
```
cat /etc/hosts
head -n2 /etc/passwd #first two lines
tail -n2 /etc/passwd #last two lines
wc -l /etc/services #count lines
```

2. grep
* regex101.com is a great site for regex testing
```
sudo grep Password /etc/ssh/sshd_config #lots of lines
sudo grep ^Password /etc/ssh/sshd_config #only matches regex ^ start of line, followed by string
```

# securing files in the filesystem

* File permissions
  * ACLs are supported by default on XFS, which comes default in RHEL8.
  * ACLs are only supported in ext3 if the fs is mounted with the `acl` option.


## understanding file metadata

1. list file permissions
```
ls -l /etc/hosts
ls -l /etc/shadow
```

2. review output of `ls -l`
![](2024-01-11-06-38-23.png)

* file types
  * regular file (`-`)
  * directory (`d`)
  * link (`l`)
  * pipe (`p`)
  * block/character (`b` or `c`)
  * socket (fifo, etc) (`s`)

3. additional metadata
```
[vagrant@rhel8 ~]$ stat /etc/hosts
  File: /etc/hosts
  Size: 207             Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 67118389    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:net_conf_t:s0
Access: 2024-01-11 10:39:35.946727108 +0000
Modify: 2024-01-11 10:39:28.404596100 +0000
Change: 2024-01-11 10:39:28.404596100 +0000
 Birth: 2023-03-30 00:06:00.025483784 +0000
[vagrant@rhel8 ~]$ stat -c %a /etc/hosts
644
[vagrant@rhel8 ~]$ stat -c %A /etc/hosts
-rw-r--r--
[vagrant@rhel8 ~]$ stat -c %Afoo%a /etc/hosts
-rw-r--r--foo644
```

4. interpolation of command output
* the output of an command invocation can be interpolated into another command invocation via two methods in bash:
  * `$(command)`
  * `` `command` `` (backticks)
```
[vagrant@rhel8 ~]$ tty
/dev/pts/0

[vagrant@rhel8 ~]$ stat
stat: missing operand
Try 'stat --help' for more information.

[vagrant@rhel8 ~]$ stat $(tty)
  File: /dev/pts/0
  Size: 0               Blocks: 0          IO Block: 1024   character special file
Device: 16h/22d Inode: 3           Links: 1     Device type: 88,0
Access: (0620/crw--w----)  Uid: ( 1000/ vagrant)   Gid: (    5/     tty)
Context: unconfined_u:object_r:user_devpts_t:s0
Access: 2024-01-11 11:49:44.091155074 +0000
Modify: 2024-01-11 11:49:44.091155074 +0000
Change: 2024-01-11 10:01:05.092155575 +0000
 Birth: -

[vagrant@rhel8 ~]$ stat `tty`
  File: /dev/pts/0
  Size: 0               Blocks: 0          IO Block: 1024   character special file
Device: 16h/22d Inode: 3           Links: 1     Device type: 88,0
Access: (0620/crw--w----)  Uid: ( 1000/ vagrant)   Gid: (    5/     tty)
Context: unconfined_u:object_r:user_devpts_t:s0
Access: 2024-01-11 11:50:48.091155074 +0000
Modify: 2024-01-11 11:50:48.091155074 +0000
Change: 2024-01-11 10:01:05.092155575 +0000
 Birth: -
```

## default permissions and the umask
* POSIX permissions

* file permissions are bitwise
  * read = 4 in decimal or 100 in binary
  * write = 2 in decimal or 010 in binary
  * execute =1 in decimal or 001 in binary (needed to `cd` to dir)
* Default permissions are:
  * file = 666
  * directory = 777

1. umask value is the default permissions
```
[vagrant@rhel8 ~]$ umask
0002
```

2. change umask

* why use this?
  * the `umask` command only affects the environment (the shell) for which you are working.

```
[vagrant@rhel8 ~]$ umask
0002
[vagrant@rhel8 ~]$ touch test1
[vagrant@rhel8 ~]$ mkdir dirtest1

[vagrant@rhel8 ~]$ umask 0
[vagrant@rhel8 ~]$ umask
0000
[vagrant@rhel8 ~]$ touch test2
[vagrant@rhel8 ~]$ mkdir dirtest2

[vagrant@rhel8 ~]$ ls -ald test? dirtest?
drwxrwxr-x. 2 vagrant vagrant 6 Jan 11 11:56 dirtest1
drwxrwxrwx. 2 vagrant vagrant 6 Jan 11 11:57 dirtest2
-rw-rw-r--. 1 vagrant vagrant 0 Jan 11 11:55 test1
-rw-rw-rw-. 1 vagrant vagrant 0 Jan 11 11:54 test2
```

## changing file permissions

* user: permissions granted to the user owner of the file.  If permission is granted, no other permissions apply.
* group: if the current user does not match the user owner, group membership is checked.
* other: if the current user and groups don't match, then other permission is applied.

1. create a new file
```
touch file_perms
ls -l file_perms
chmod -v 666 file_perms # actively sets all bits in permissions control entry
chmod -v +w file_perms # actively sets the owner bit to write only if it is allowed by the current `umask` in the permissions control entry
chmod -v o+w file_perms # actively sets the owner bit to write in the permissions control entry
```

## changing file ownership
* chmod, chown, chgrp

* review info about your user
```
[vagrant@rhel8 ~]$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```
* change owner
```
sudo chown root f1
```
* change owner and group
```
sudo chown root:root f1
```
* change group
```
sudo chgrp root f1
```

## managing links in linux

### hardlinks
* can be created for files, not directories
```
cd
mkdir new_dir
ls -ld new_dir #see hardlink count of two
ls -ldi new_dir #also lists inodes, which are the same for the entries: `.` and `new_dir`

mkdir new_dir/dir1
ls -ldi new_dir #now there are three hardlinks,  `.`, `new_dir`, and `..`
```

### softlinks
```
ln -s /etc/services #creates `services` softlink file within current directories
ls -l services
```

## switching user and groups IDs
* wheel groups: https://www.youtube.com/watch?v=5vmp6Pg8rWc
  * https://catb.org/jargon/html/W/wheel

* you can apply a group membership, but it isn't effective on the user until they start a new sessions
```
[vagrant@rhel8 ~]$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[vagrant@rhel8 ~]$ sudo usermod -aG wheel vagrant
[vagrant@rhel8 ~]$ id vagrant #current candidate config
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant),10(wheel)
[vagrant@rhel8 ~]$ id #current running config
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

* switching groups
  * this controls the group ownership of new files.
  * you can spawn a new shell with a different groupid
```
[vagrant@rhel8 ~]$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[vagrant@rhel8 ~]$ id vagrant
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant),10(wheel)
[vagrant@rhel8 ~]$ touch gid_file; ls -l gid_file
-rw-rw-r--. 1 vagrant vagrant 0 Jan 12 10:52 gid_file
[vagrant@rhel8 ~]$ newgrp wheel
[vagrant@rhel8 ~]$ id
uid=1000(vagrant) gid=10(wheel) groups=10(wheel),1000(vagrant) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[vagrant@rhel8 ~]$ touch wheel_gid_file; ls -l wheel_gid_file
-rw-r--r--. 1 vagrant wheel 0 Jan 12 10:53 wheel_gid_file
[vagrant@rhel8 ~]$ exit
exit
[vagrant@rhel8 ~]$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[vagrant@rhel8 ~]$ ls -l wheel_gid_file
-rw-r--r--. 1 vagrant wheel 0 Jan 12 10:53 wheel_gid_file
[vagrant@rhel8 ~]$ touch post_wheel_gid_file; ls -l post_wheel_gid_file
-rw-rw-r--. 1 vagrant vagrant 0 Jan 12 10:53 post_wheel_gid_file
```

* switch user id
```
sudo passwd root
su - #switches to root account, and sets environment
```

# archiving files in linux

## understanding special case permissions

### write only files
* although not strictly archiving files, allow users to submit data to a log file without being able to read the file content.
  
### securing directories
* read allows file name listing (this is what you want to do if you want to restrict)
* execute allows access to directory
  * by assigning execute only, it allows users only access to the files that they know exist.
* write allows users to create files and access them by name.

### archiving files using `tar` and `star`

### file compression using `gzip` and `bzip2`

## working with special case permissions

### working with files
```
touch log
ls -l log
cat log # can read
chmod -v u=w log #set explicit permissions
cat log #nope
echo hello >> log #yep
cat log #nope
sudo cat log #yep
hello
```

### working with dirs
```
mkdir -m 155 foo #POSIX, set the `mode` of the directory
[vagrant@rhel8 ~]$ ls -al | grep foo
d--xr-xr-x. 2 vagrant vagrant    6 Jan 12 11:10 foo
cd foo # can "execute" (enter) the directory
[vagrant@rhel8 foo]$ ls -al . #nope
ls: cannot open directory '.': Permission denied

[vagrant@rhel8 ~]$ chmod -v u=wx foo #set write and exec permissions
mode of 'foo' changed from 0155 (--xr-xr-x) to 0355 (-wxr-xr-x)
[vagrant@rhel8 ~]$ echo test >> foo/file #yep
[vagrant@rhel8 ~]$ cat foo/file
test
```

## using the `tar` archiver

* create a tar
```
[vagrant@rhel8 ~]$ sudo du -sh /etc
26M     /etc
[vagrant@rhel8 ~]$ sudo tar -cf etc.tar /etc
tar: Removing leading `/' from member names
[vagrant@rhel8 ~]$ ls -lh etc.tar
-rw-r--r--. 1 root root 25M Jan 12 11:16 etc.tar
```

* review contents of tar
```
tar -tf etc.tar
```

* extract tar file
```
#extract to current dir
tar -xf etc.tar
[vagrant@rhel8 ~]$ ll ./etc | wc -l
186

#extract to target dir
sudo tar -xf /home/vagrant/etc.tar etc/hosts
```

## using the `star` archiver
* more options than `tar`

1. install star package
```
sudo yum install -y star
man star
star --help
```

2. backup files from locally expanded etc directory (note that you don't have permissions to everything)
```
star -c -f myetc.tar ./etc
```

3. list
```
star -t -f myetc.tar
```

4. extract
```
star -x -f myetc.tar # has overwrite protection, which `tar` does not
```

## compressing files
* tar can call gzip or bzip2
```
tar -czf #gzip, faster but less compressed
tar -cjf #bzip2, slower but more compressed
```

1. clear homedir
```
cd ~
sudo rm -rf *
```

2. tar etc uncompressed
```
sudo tar -cf etc.tar /etc
[vagrant@rhel8 ~]$ ls -lh etc.tar
-rw-r--r--. 1 root root 25M Jan 12 11:26 etc.tar
```

3. compress the archive with `tar`, timing the zip action, verify size, then decompress
```
[vagrant@rhel8 ~]$ time gzip etc.tar

real    0m0.891s
user    0m0.874s
sys     0m0.009s
[vagrant@rhel8 ~]$ ls -lh
total 7.0M
-rw-r--r--. 1 vagrant vagrant 7.0M Jan 12 11:26 etc.tar.gz
[vagrant@rhel8 ~]$ gunzip etc.tar.gz
[vagrant@rhel8 ~]$ ls -lh
total 25M
-rw-r--r--. 1 vagrant vagrant 25M Jan 12 11:26 etc.tar
```

4. compress the archive with `bzip2`
```
[vagrant@rhel8 ~]$ time bzip2 etc.tar

real    0m1.855s
user    0m1.812s
sys     0m0.021s
[vagrant@rhel8 ~]$ ls -lh
total 5.5M
-rw-r--r--. 1 vagrant vagrant 5.5M Jan 12 11:26 etc.tar.bz2
[vagrant@rhel8 ~]$ bunzip2 etc.tar.bz2
rm -rf etc.tar
```

5. use tar with gzip option
```
sudo tar -vzf etc.tar.gz /etc 
```

# RHEL 8: Creating Shell Scripts

# automating tasks using bash scripts

## what are shell scripts? / using script syntax at the CLI
* shell scripts are the same as batch scripts... straight execution
* you heavily leverage exit codes of previous step (`0` == success, `N` == some other error)
* exit code is accessible via the bash var `$`

### `for` loop
```
[vagrant@rhel8 ~]$ for item in hi salut ciao ; do echo $item ; done
hi
salut
ciao
```

### `for` loop to create a some users and setting their password
```
for item in bob sally sue
do
sudo useradd -m $item
echo Password1 | sudo passwd --stdin $item
tail -n1 /etc/passwd
done
```

### `while` loop
```
[vagrant@rhel8 ~]$ i=3; while [ $i -gt 0 ]; do echo $i; let i-=1 ; done
3
2
1
```

## understanding script logic

* `$?` returns the exit code, other than 0 indicates a failure.
* you can chain using:
  * `&&` if first command succeeds, then execute second
  * `||` if first command fails, then execute second

## using simple logic at the CLI (lab)
* check output
```
[vagrant@rhel8 ~]$ echo $?
0
[vagrant@rhel8 ~]$ tail -n3 /etc/fail
tail: cannot open '/etc/fail' for reading: No such file or directory
[vagrant@rhel8 ~]$ echo $?
1
```
* dynamically use the logic results
```
[vagrant@rhel8 ~]$ mkdir etc || cd etc #if failed
mkdir: cannot create directory ‘etc’: File exists
[vagrant@rhel8 etc]$ pwd
/home/vagrant/etc
[vagrant@rhel8 etc]$ mkdir sales && cd sales #if succeeded
[vagrant@rhel8 sales]$ pwd
/home/vagrant/etc/sales
```
* grouping logic with `{}`
```
cd etc || { mkdir etc && cd etc; }
#if cd fails, then... mkdir, if mkdir succeeds, then cd
```

# creating your own shell scripts

## implementing the shebang

* note that you do NOT need to know vim at the level in which Andrew teaches it.

* what is shebang, designates a bash script
```
[vagrant@rhel8 ~]$ echo "foo" > foo.sh
[vagrant@rhel8 ~]$ file foo.sh
foo.sh: ASCII text
[vagrant@rhel8 ~]$ echo "#! /bin/bash" > test.sh
[vagrant@rhel8 ~]$ cat test.sh
#! /bin/bash
[vagrant@rhel8 ~]$ file test.sh
test.sh: Bourne-Again shell script, ASCII text executable
```
* make a file executable
```
chmod +x my.sh
```

## working with vim abbreviations
* you should train on vim separately.

## working with PATH and execute permissions

* can run scripts with `bash script.sh`
* you can `chmod +x script.sh` then invoke with `script.sh`
* the PATH variable exists, and by default `~/bin` is in PATH var
* review the `which` command

## special variable / using special variables in scripts
* `$$`: current PID
* `$?`: exit status
* `!$`: last argument in bash history
* `$0`: is the root name of the program name (also note the use of `basename` command)
* `$[N]`: arguments
* `$*`: all arguments as a string
* `$@`: all arguments as an array
* `$#`: arg count

### setting a default value for an arg
```
PID=${*:-"1"} #since there are {}, this will only be called if there's no value for var
```

# automating the user creation process

## testing user input
* check arg count
```
#!/bin/bash
if [ "$#" -lt 1 ] ; then
  echo "no args bro"
  echo "usage: $0 <username>"
  exit 1 #error code 1
fi
```

## ensure user is not already present
* use `getent` to check `passwd`
```
#!/bin/bash
if [ "$#" -lt 1 ] ; then
  echo "no args bro"
  echo "usage: $0 <username>"
  exit 1 #error code 1
elif getent passwd "$1"; then
  echo "the user $1 already is on the system"
  exit 2
fi
```

## using read during script execution
* use `read` to take in user input and set a var (`USER_PASSWORD`)
```
#!/bin/bash
if [ "$#" -lt 1 ] ; then
  echo "no args bro"
  echo "usage: $0 <username>"
  exit 1 #error code 1
elif getent passwd "$1"; then
  echo "the user $1 already is on the system"
  exit 2
fi

#get the password
read -s -p "enter password for user $1: " USER_PASSWORD # -s is silent, -p is a prompt

if [ (echo "$USER_PASSWORD" | wc -c ) -lt 1 ] ; then
  echo "no args bro"
  exit 3
fi

#create the user
sudo useradd -m "$1"

#set the password
echo "$1:$USER_PASSWORD" | sudo chpasswd

#validate
getent passwd "$1"
```

## using functions and loops in scripts
