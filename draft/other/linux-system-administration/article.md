# Section 0 - Prerequisites
Prospective Linux system administrators (aka *sysadmins*) are expected to be well-acquainted with the command line. However, in this section we will review some of those commands with their most common options to help you brush up your skills (for more information, you can always refer to each *man page*). Additionally, it is important to be familiar with a text mode editor such as `vim`, `nano`, or `emacs` since the configuration directives for system services and user environments -to name a few examples- are stored in plain-text files.
> Unless where explicitly noted, this tutorial is distribution-agnostic, meaning the concepts and commands covered herein apply regardless of the Linux flavor of your choice. On a side note, commands that require super-user permissions are noted accordingly.

## System information commands
The commands in this section will help *sysadmins* get to know the systems they are responsible for like the palm of their hands:
- When run without options, `hostname` wil shot the system's **host name** as defined in `/etc/hosts` or `/etc/hostname`. By adding `-I` (or its equivalent `--all-ip-addresses`), all the configured addresses on all network interfaces will be returned.
- **Disk usage and available storage space** can be easily found out using `du` and `df`, respectively. Strictly speaking, the former will estimate the file space usage by file or directoty, whereas the latter will report the same information on a per-filesystem basis. Both support the `-h` flag, which formats the output if human-readable form (10K, 15M, 3G, for example) instead of using bytes.
When followed by a directory and the star wildcard `*`, the combined switch `-sch` allows `du` to summarize totals for the directory and each subdirectory within. Also, a grand total is provided. For example, `du -sch /var/log/*` will print the file space usage of all files and subdirectories inside, along with the grand total for `/var/log`. To view only the grand total, remove the `c` from `-sch`.
- **Memory and CPU** represent another essential set of information when it comes to planning the kind of software that can run on the system. For example, you would not run a mail service on a machine with only 1 GB of RAM (believe me, I've tried!). To find out the architecture, number of cores, vendor id, model name, and speed (and more) of the CPU, you can use `lscpu`. On the other hand, the `free` command (which, by the way, also supports the `-h` for returning output in human-readable form) shows the amount of free and used memory in the system. For a thorough explanation on how Linux actually handles system memory, you may want to refer to [Linux Ate My RAM](http://www.linuxatemyram.com/).
- **Uptime** is a critical metric that indicates the time during which the system has been operational. If you're only interested in knowing how long the machine has been running, use the `-p` option. If you remove that flag, `uptime` will also return the current system time, the number of logged users, and the load average for the past 1, 5, and 15 minutes. We will revisit this topic in *Section 3 - Processes* later in this guide.

## Text processing commands
It is often said that in Unix and its derivatives everything is a file, and thus the need to master command-line text processing tools. The following commands are a must for every system administrator.
> When in the Linux community we speak of everything being a file, we actually mean that every system object can be opened, read from, and written to as a file in the regular sense of the word (given you have the necessary permissions to do so).

- To display **the first or last ten lines of a file**, use `head` or `tail`, respectively. Also, you can use `head -n` or `tail -n` followed by an integer to view a different number of lines. For example, `head -n 2 /etc/passwd` will return the first two lines of `/etc/passwd`.
Often, we will be interested in viewing the last lines of a file and watch it as more lines are added. This is the case when we want to monitor a system log as events are recorded. To do so, we use `tail -f` followed by the path to the log file, such as `tail -f /var/log/httpd/access_log` (the access log of the Apache web server in a CentOS 7 server, to name an example).
- To **search for patterns and regular expressions** in files, `grep` is the tool for the job. This command must be followed by a pattern, [character class](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_04_03.html#sect_04_03_02), or [regular expression](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_04_02.html#sect_04_02_02) and the file where the search must be performed. For example, `grep student /etc/passwd` will return the line(s) in `/etc/passwd` where the word *student* is found. To ignore case so that both *Student* and *StUdEnT* are also matches, do `grep -i student /etc/passwd`. 
If instead a simple pattern you need to use a regular expression, add the `-E` switch as in `grep -Ei $nologin$ /etc/passwd`. Here, the `$` followed by the word `nologin` is a regular expression that will match all lines in `/etc/passwd/` that end in `nologin`.
- To **search for system objects** (most likely, files or directories we want to locate or work with), we'll use `find`. First off, we need to indicate the directory where we must start the search, the object type, and the name. Although there are other filter criteria that we can add to narrow down our search, this is a typical example of the use of `find`. For example, `find /etc -type f -name sshd_config` will search for a regular file (`type f`) named `sshd_config` starting at `/etc`.
- To **extract sections from each line in a file**, `cut` will by our best ally. This tool is often used with `-d` to specify a delimiter surrounded in single quotes, and `-f` followed by a number, range, or a comma-separated list of files that indicate which section(s) we're interested in. If you're interested in listing the users (1st field) found in `/etc/passwd` with their default shell (7th field), you can do `cut -d':' -f1,7 /etc/passwd`. To add the home directories (6th field), you would do `cut -d':' -f1,6-7 /etc/passwd` instead.

## Empty Files, Redirection And Pipes
Linux allows you to create empty files using the `touch` command followed by the name of the new file. This is useful to initialize files for a sandbox environment. If `touch` is followed by the name of an existing file, its modification time will be updated to the current system time. 
- To initialize a file and add content to a file (or overwrite its contents) in one single step, we will use the `>` redirection operator followed by the name of the file. Keep in mind that the redirection operator must be preceded by the command whose output will be inserted in the file. For example, `cut -d':' -f1,7 /etc/passwd > usersandshells.txt` will create a file named `usersandshells.txt` in the current working directory (if it does not exist previously) or overwrite its contents (if such file exists) with the output of `cut -d':' -f1,7 /etc/passwd`.
- To avoid overwriting an existing file when adding content, use `>>` instead. This approach will also initialize the file if it does not exist.
> Using `>` or `>>` is not a one-size-fits-all solution, but depends on the specific scenario where it is to be utilized.
- Most often, you will need to **chain two or more commands** together so that the output of one is sent as input to the next. To do this, we will use the vertical bar `|`, also known as *pipeline*. Thus, `cut -d':' -f1,7 /etc/passwd | grep gacanepa` will filter out the line containing the word *gacanepa* from the output of `cut -d':' -f1,7 /etc/passwd`.
- To **sort unique lines** in a file or from the output of a command, we'll use `sort` and `uniq` together. For example, `cut -d':' -f7 /etc/passwd | sort | uniq` will return the default shell(s) for the accounts listed in `/etc/passwd`. There is a good reason why `sort` follows the first pipeline and then sends its output to `uniq` and not the other way around. The latter requires its input to be sorted beforehand in order to return unique lines.

You are highly encouraged to review this section before proceeding with the rest of the tutorial. After reviewing the man pages, you may want to consider referring to [The Linux Documentation Project](http://tldp.org/) web site for further details on the use of command-line tools that are a must for prospective Linux system administrators.

# Section 1 - User And Group Management
Since Linux is a multi-user operating system, several people may be logged in and actively working on a given machine at the same time. Security-wise, it is never a good idea to allow users to share the credentials of the same account. In fact, best practices dictate the use of as many user accounts as people needing access to the machine. At the same time, it is to be expected that two or more users may need to share access to certain system resources, such as directories and files. User and group management in Linux allows us to accomplish both objectives. In this section and in the next (*Chapter 2 - Permissions*) we will explain how.

## A Note On Super-User Permissions
Since adding a new user involves dealing with an account other than your own, it requires super-user (aka *root*) privileges. The same applies to other user or group management tasks, such as deleting an account, updating it, and creating and removing groups. These operations are performed using the following commands:
- `adduser`: add a user to the system.
- `userdel`: delete a user account and related files.
- `addgroup`: add a group to the system.
- `delgroup`: remove a group from the system.
- `usermod`: modify a user account.
- `chage`: change user password expiry information.
- Relevant files: `/etc/passwd` (user information), `/etc/shadow` (encrypted passwords) and `/etc/group` (group information).

>Super-user permissions can be gained either by changing to the *root* user with the `su` command or using `sudo`. The latter approach is used by default in Ubuntu and derivatives, and is preferred over the former in other distributions as well. It is also important to note that, as opposed to other Linux flavors, the user that is created when Ubuntu is first installed has super-user privileges out-of-the-box.
>You can verify whether `sudo` is installed on your machine by running `which sudo` on a terminal. If this command returns the absolute path of the associated file (typically `/usr/bin/sudo`), it means that the package is installed. Otherwise, you can install it with `apt-get install sudo` on Debian, or `yum install sudo` in CentOS or similar.

## Adding A New Regular Account
To begin, let's create a new user named **pluralsight** using Ubuntu and CentOS as representative distributions. 

In Ubuntu or derivatives, this is as easy as doing (you will be required to enter your password to run `sudo`):
```
sudo adduser pluralsight
```

In other distributions, login as **root** and do:
```
adduser pluralsight
```

> You may be prompted to set the new user's initial password, and other optional information (such as full name, work phone, etc) to be stored in `/etc/passwd` using colons as field separators. If not, you can assign a password for **pluralsight** with `passwd pluralsight` and entering it twice. Needless to say, you must do `sudo passwd pluralsight` if you're in Ubuntu.

Now that we have a regular user account created, we will explain how to utilize it to perform user management tasks.

> When a new user is added, a group (more on this in *Chapter 2 - Permissions*) with the same name is created automatically. This is called a *primary group*.

## The /etc/sudoers File
To grant **pluralsight** super-user permissions, we will need to add an entry for it in  `/etc/sudoers`. This file is used to indicate which users can run what commands with elevated permissions (most likely as *root*).

### Step 1 - Open /etc/sudoers With Visudo
Although `/etc/sudoers` is nothing more and nothing less than a plain text file, it must **NOT** be edited using a regular text editor. Instead, we will use the `visudo` command. As opposed to other text editors, by utilizing `visudo` we will ensure that 1) no one else can modify the file at the same time, and 2) the file syntax is checked upon saving changes.

>To launch `visudo`, just type the command and press Enter. Don't forget to do `sudo visudo` instead if you're in Ubuntu. In any event, the file will be opened using your default text editor.

### Step 2 - Add An Entry in /etc/sudoers For The New User Account
The easiest method to grant super-user permissions for **pluralsight** is adding the following line at the bottom of `/etc/sudoers`:
```
pluralsight    ALL=(ALL) ALL
```
Let's explain the syntax of this line:
- First off, we indicate which user this rule refers to (**pluralsight**).
- The first `ALL` means the rule applies to all hosts using the same `/etc/sudoers` file. Nowadays, this means the *current host* since the same file is not shared across across other machines. 
- Next, `(ALL) ALL` tells us that **pluralsight** will be allowed to run *all* commands as any user. Functionally speaking, this is equivalent to `(root) ALL`.

>For more information on the available options in `/etc/sudoers`, refer to `man sudoers`.

While saving, `visudo` will alert you if a syntax error is found in the file, and indicate the line where the error is found so that you can identify it more easily. 

## Switching Users
If no errors are found while saving the recent changes in `/etc/sudoers`, we'll be ready to start using **pluralsight** to perform administrative tasks. To do so, use the `su` command to change to that account. 

>Note that from this point, there is no need to use the **root** account.

Additionally, the `-l` option will allow to provide an environment similar to what the user would expect if he or she had logged in directly:
```
su -l pluralsight
```
and press Enter.

## Getting Started With User Management
While we're logged as **pluralsight**, let's add yet another user account called **student** with a password of our choice. You can skip the second command if the first one prompts you to enter the password for **student**:
```
sudo adduser student
sudo passwd student
```
(enter password twice).
If everything went as expected, a new user and a primary group called **student** were created with an unique user and group id, respectively. Additionally, the new user is assigned a personal directory (`/home/student` in this case), and a login shell (`/bin/bash` by default). 
Using `usermod` we can change the *home* directory to another existing one, edit the login shell, and an add an optional comment on the user (such as full name or employee information) as follows:
- To change the *home* directory to `/Users/student` (this directory must exist), use the `--home` (or its short equivalent `-d`) option: `sudo usermod --home /Users/student student`
- If the user prefers to use `/bin/sh` as login shell (or company policies require employees to use it), the `--shell` (or `-s`)  flag will do the trick: `sudo usermod --shell /bin/sh student`
- To add a descriptive comment to the user account, use `--comment` (or `-c`), followed by the comment enclosed between double quotes. For example, you can do `sudo usermod --comment "Account used for Pluralsight guide" student`

The above commands can be grouped into one as follows:
```
sudo usermod --home /Users/student --shell /bin/sh --comment "Account used for Pluralsight guide" student
```
In Fig. 1 we see the contents of `/etc/passwd` *before* and *after* modifying the user information:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c8a0e38f-3432-42f5-8fb1-f4f6b5ce5743.png)


>As you can see in these examples, the syntax of `usermod` consists in invoking the command followed by one or more options (with their corresponding values) and the user account they should be applied to.

In addition to changing the user's home directory, login shell, and descriptive comment, `usermod` also allows to lock (and unlock) an account and set its expiration date. To do so, use `--lock` (or `-L`), `--unlock` (or `-U`), and `--expiredate` (or `-e`), respectively. The expiration date must be specified using the **YYYY-MM-DD** format.

For example, to lock **student**, do:
```
sudo usermod --lock pluralsight
```
If we now try to login as **student**, we will get an `Authentication failure` error, as shown in Fig. 2. After unlocking the account with `sudo usermod --unlock student`, we will be able to use the account again, as also observed in Fig. 2.
>When an user is locked, an exclamation sign `!` is placed before the encrypted password in `/etc/shadow`, thus disabling the account.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/62679a1e-8f87-4b1c-b3b5-5811bf6e2042.png)


To set the expiration date of **student**, do
```
sudo usermod --expire-date 2017-10-31 student
```

The changes can then be viewed with 
```
sudo chage -l student
```
By the way, you can use `chage` to enforce a password change policy. As a safety measure, it is important to have users change their passwords after a given period of time. For example, to force **student** to change his password every 60 days, do:
```
sudo chage --maxdays 60 student
```
 Fig. 3 shows **student**'s password information after performing the above changes:
 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/763f6f34-715d-4287-9ca8-428299f517be.png)


>In `man chage` you can find more information about other useful password expiration tasks.

### Groups
In Linux, groups can be defined as a way to organize users that need the same type of access to a directory or file. In this section we will explain how to create and remove groups, whereas in *Chapter 2 - Permissions* a more detailed discussion on user and group permissions will be given, along with file and directory ownership.

To create a new group named **finances**, do `addgroup finances`. To remove it from the system, use `delgroup finances`. The information of the new group is stored in `/etc/group`, where each line shows the name of the group and the user accounts that are associated with it.

>It is important that you practice the commands and examples outlined in this section until you feel confident using them. Then proceed with the next section where we will be adding and removing users to and from groups, and granting or preventing access to files and directories.

# Section 2 - Permissions

Every file, directory, and other system objects in Linux are assigned an owner and a group. This is the most basic, yet essential, part of system security that protects users from each other. Owners, users belonging to a group, and all others may be granted different types of access to read from, write to, or execute files. This is generally referred to as *file permissions* in Linux. 
To set permissions and manage ownership, we will use the following commands:
- `chmod`: change file permissions.
- `chown`: change file owner.
- `chgrp`: change group ownership.
- `id`: print user and group IDs.

### Users, Groups, and Everyone Else
Typically, the owner of a file is the user who created it, and (at least initially) the group is the one associated with the owner (aka *primary group*). To illustrate, let's create a new file called `test1` in the current working directory:
```
echo "This is a dummy file called test1" > test1
```
Then let's do `ls -l` on it:
```
ls -l test1
```
As seen in Fig. 1, the first character in the output indicates that `test1` is a regular file (i.e. not a directory or other type of system object). The next nine  characters (divided in 3 groups of 3) indicate the *read* (`r`), *write* (`w`), and *execute* (`x`) permissions of the owner, the group owner, and the other users in the system.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6d74f02b-ea4a-48ee-83d7-0574e9064fce.png)

The first group of 3 characters (`rw-`) indicate that the owner of the file (**pluralsight**) has *read* and *write* permissions. In this example, the next 2 groups of 3 characters are identical, which means that both the members of the group owner (**pluralsight**) and the rest of the users have the same permissions on the file, respectively.

> The *read* permission on a file is necessary for it to be opened and read. The same permission on a directory (along with *execute*) is required to list its contents and to move into it.

To change the permissions on a file, we will use `chmod`. This command must be followed by a symbolic representation that indicates 1) who the new permissions should be applied to:
- `u` means *user* (or more precisely, the owner of the file).
- `g` means *group*.
- `o` means **all other** users.
- `a` means **all** users.

and 2) the type of permission:
- `+r` *adds* read permission
- `-r` *removes* read permission
- `+w` *adds* write permission
- `-w` *removes* write permission
- `+x` *adds* execute permission
- `-x` *removes* execute permission
- `+rw` *adds* read **and** write permissions
- `+rwx` *adds* read **and** write **and** execute permissions

and so on. Finally, we also need to indicate the name of the file or directory. Thus, we can change the permissions on `test1` as follows:
- Add *execute* permissions for the owner only: `chmod u+x test1`
- Remove *read* and *write* permissions from users other than the owner and not in the group owner: `chmod o-rw test1`.

Fig. 2 shows the current permissions of `test1` after making the above changes:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/3f244cb1-7a0c-4fd3-9e45-b5988b6b58b9.png)

Another way to set permissions is using an octal instead of a symbolic expression. To *translate* into octal form, we must split the desired permissions in groups of 3 characters, and use the following table to replace each group with its octal equivalent:

| Permissions 	| Binary 	| Octal 	|
|:-----------:	|:------:	|:-----:	|
|     ---     	|   000  	|   0   	|
|     --x     	|   001  	|   1   	|
|     -w-     	|   010  	|   2   	|
|     -wx     	|   011  	|   3   	|
|     r--     	|   100  	|   4   	|
|     r-x     	|   101  	|   5   	|
|     rw-     	|   110  	|   6   	|
|     rwx     	|   111  	|   7   	|

>Each permission is often referred to as *bit* based on the above table. Thus, the presence of a given permission means the corresponding *bit* is set. For example, `r-x` means that both the *read* **and** *execute* bits are set, whereas the *write* bit is not.

Thus, if you need to assign `rwx` permissions on `test1` for the owner, `rw` for the group, and only `r` for everyone else, you should do `chmod 764 test1`. Here, the 7, 6, and 4 represent `rwx`, `rw`, and `r`, respectively.

Other examples:
- `rw` permissions for the owner and the group and no access whatsoever for other users: `chmod 660 test1`
- All (`rwx`) permissions for the owner, but only *read* **and** *execute* permissions for everyone else (this includes both the group and other users): `chmod 755 test1`

>When to use the symbolic expression or the octal form to set permissions depends on the specific scenario. If only a bit needs to be set, it's likely that the symbolic expression will be faster. But if we want to set different permissions for the owner, the group, and all others, the octal form is easier and more straightforward.

If we executed the above commands, the current permissions of `test1` are `rwxr-xr-x` (`755`) and both the owner and the group are **pluralsight** and **pluralsight**, respectively. Let's now introduce the use of `chgrp` to change the group to **finances**, which we created in *Chapter 1 - User and Group Management*. The command must be followed by the group name and the name of the file whose ownership needs to be set (`test1` in this case):

```
sudo chgrp finances test1
```
Next, let's see if user **student** can write to this file. As we can see in Fig. 3, we get a *Permission denied* error. To give **student** write access to the file, we can set the corresponding bit for the group:

```
sudo chmod g+w test1
```

and add the account to **finances**, again using `usermod` but this time with the `-aG` combined option as follows:

```
sudo usermod -aG finances student
```

>The `-aG` combined option is short for *append to group*. Now **finances** is said to be a *secondary* or *supplementary* group for user **student**. The new access permissions will take effect the next time **student** logs in.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8fa07297-9b0f-4dbe-9360-7dc505fa9c08.png)

If instead of adding **student** to the **finances** group, we wanted to make him the owner of `test1`, we can use `chown` followed by the user and file names, in that order:

```
sudo chown student test1
```
> Note that the above command will cause that **pluralsight** does not have access to `test1` anymore, as such account is now neither the owner of the file or has not been added to the **finances** group.

### Special Permissions

Besides the standard `rwx` file permissions, there are three others that are worth mentioning: **setuid**, **setgid**, and the **sticky bit**. Let's examine each of them.

- When the **setuid** bit is set on an executable file, any user can execute it using the permissions of the *owner* of such file.
- When the **setgid** bit is set on an executable file, any user can execute it using the permissions of the *group* of such file.

These special permissions pose a security risk when used carelessly. For example, if any user is allowed to run a command with superuser privileges, he or she can gain access to files owned by other users, and even by root. It isn't hard to see how this can easily cause havoc in a system: essential files could be removed, personal directories could be wiped out entirely, and even hardware can experience erratic behavior. It only takes a malicious or irresponsible individual to cause all this. Thus, the use of the **setuid** and the **setgid** bits must be highly restricted. A valid case where the **setuid** bit must be used is in `/usr/bin/passwd`. This file is owned by root but needs to be used by any user to change his or her own password (but not that of other users).

- When the **sticky bit** is set on a directory, the files within that directory can be deleted only by the owner, the owner of the directory, or by root. This is often used in shared directories to prevent a user from deleting other people's files from it.

To set the **setuid** bit from `test1`:
```
sudo chmod u+s test1
```
To set the **setgid** from `test1`:
```
sudo chmod g+s test1
```
To create a directory named `files` and set the **sticky bit** on it:
```
sudo mkdir files
sudo chmod +t files
```

To illustrate the use of these special permissions, let's refer to Fig. 4. Note how now there is a `t` where the *execute* permission should be for all users in `files`, and two `s`'s in the same position for the owner and the group.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/bef1d9a8-91bb-43ce-8298-c3eab49b31d6.png)

Under this scenario and if `test1` was a program, any user could execute it using the permissions of **pluralsight** (owner) or the access privileges of the **finances** group.


### Revisiting /etc/sudoers


# Section 3 - Processes



# Section 4 - Shell Scripting With Bash



# Section 5 - Scheduling And Running Tasks With Cron



# Section 6 - Managing Software Using Yum Or Aptitude



# Section 7 - Setting Up And Securing An OpenSSH Server




# Section 8 - RAID




# Section 9 - LVM



# Section 10 - Disk Quotas