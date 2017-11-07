# Section 0 - Prerequisites
Prospective Linux system administrators (aka *sysadmins*) are expected to be well-acquainted with the command line. However, in this section we will review some of those commands with their most common options to help you brush up your skills (for more information, you can always refer to each *man page*). Additionally, it is important to be familiar with a text mode editor such as `vim`, `nano`, or `emacs` since the configuration directives for system services and user environments -to name a few examples- are stored in plain-text files.
> Unless where explicitly noted, this tutorial is distribution-agnostic, meaning the concepts and commands covered herein apply regardless of the Linux flavor of your choice. On a side note, commands that require superuser permissions are noted accordingly.

## System Information Commands
The commands in this section will help *sysadmins* get to know the systems they are responsible for like the palm of their hands:
- When run without options, `hostname` wil shot the system's **host name** as defined in `/etc/hosts` or `/etc/hostname`. By adding `-I` (or its equivalent `--all-ip-addresses`), all the configured addresses on all network interfaces will be returned.
- **Disk usage and available storage space** can be easily found out using `du` and `df`, respectively. Strictly speaking, the former will estimate the file space usage by file or directoty, whereas the latter will report the same information on a per-filesystem basis. Both support the `-h` flag, which formats the output if human-readable form (10K, 15M, 3G, for example) instead of using bytes.
When followed by a directory and the star wildcard `*`, the combined switch `-sch` allows `du` to summarize totals for the directory and each subdirectory within. Also, a grand total is provided. For example, `du -sch /var/log/*` will print the file space usage of all files and subdirectories inside, along with the grand total for `/var/log`. To view only the grand total, remove the `c` from `-sch`.
- **Memory and CPU** represent another essential set of information when it comes to planning the kind of software that can run on the system. For example, you would not run a mail service on a machine with only 1 GB of RAM (believe me, I've tried!). To find out the architecture, number of cores, vendor id, model name, and speed (and more) of the CPU, you can use `lscpu`. On the other hand, the `free` command (which, by the way, also supports the `-h` for returning output in human-readable form) shows the amount of free and used memory in the system. For a thorough explanation on how Linux actually handles system memory, you may want to refer to [Linux Ate My RAM](http://www.linuxatemyram.com/).
- **Uptime** is a critical metric that indicates the time during which the system has been operational. If you're only interested in knowing how long the machine has been running, use the `-p` option. If you remove that flag, `uptime` will also return the current system time, the number of logged users, and the load average for the past 1, 5, and 15 minutes. We will revisit this topic in *Section 3 - Processes* later in this guide.

## Text Processing Commands
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
Since Linux is a multi-user operating system, several people may be logged in and actively working on a given machine at the same time. Security-wise, it is never a good idea to allow users to share the credentials of the same account. In fact, best practices dictate the use of as many user accounts as people needing access to the machine. At the same time, it is to be expected that two or more users may need to share access to certain system resources, such as directories and files. User and group management in Linux allows us to accomplish both objectives. In this section and in the next (*Section 2 - Permissions*) we will explain how.

## A Note On Superuser Permissions
Since adding a new user involves dealing with an account other than your own, it requires superuser (aka *root*) privileges. The same applies to other user or group management tasks, such as deleting an account, updating it, and creating and removing groups. These operations are performed using the following commands:
- `adduser`: add a user to the system.
- `userdel`: delete a user account and related files.
- `addgroup`: add a group to the system.
- `delgroup`: remove a group from the system.
- `usermod`: modify a user account.
- `chage`: change user password expiry information.
- Relevant files: `/etc/passwd` (user information), `/etc/shadow` (encrypted passwords) and `/etc/group` (group information).

>Superuser permissions can be gained either by changing to the *root* user with the `su` command or using `sudo`. The latter approach is used by default in Ubuntu and derivatives, and is preferred over the former in other distributions as well. It is also important to note that, as opposed to other Linux flavors, the user that is created when Ubuntu is first installed has superuser privileges out-of-the-box.
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

> When a new user is added, a group (more on this in *Section 2 - Permissions*) with the same name is created automatically. This is called a *primary group*.

## The /etc/sudoers File
To grant **pluralsight** superuser permissions, we will need to add an entry for it in  `/etc/sudoers`. This file is used to indicate which users can run what commands with elevated permissions (most likely as *root*).

### Step 1 - Open /etc/sudoers With Visudo
Although `/etc/sudoers` is nothing more and nothing less than a plain text file, it must **NOT** be edited using a regular text editor. Instead, we will use the `visudo` command. As opposed to other text editors, by utilizing `visudo` we will ensure that 1) no one else can modify the file at the same time, and 2) the file syntax is checked upon saving changes.

>To launch `visudo`, just type the command and press Enter. Don't forget to do `sudo visudo` instead if you're in Ubuntu. In any event, the file will be opened using your default text editor.

### Step 2 - Add An Entry in /etc/sudoers for the new User Account
The easiest method to grant superuser permissions for **pluralsight** is adding the following line at the bottom of `/etc/sudoers`:
```
pluralsight    ALL=(ALL) ALL
```
Let's explain the syntax of this line:
- First off, we indicate which user this rule refers to (**pluralsight**).
- The first `ALL` means the rule applies to all hosts using the same `/etc/sudoers` file. Nowadays, this means the *current host* since the same file is not shared across across other machines. 
- Next, `(ALL) ALL` tells us that **pluralsight** will be allowed to run *all* commands as any user. Functionally speaking, this is equivalent to `(root) ALL`.

### Step 3 (Optional): Create Command Aliases
An alternative to using the wide permissions outlined above, we can restrict the list of commands that can be executed by a given user by grouping them into sets known as *aliases*. For example, we may want to allow user **pluralsight** to only use `adduser` and `usermod`, but not other commands. We have to choices here:
- List the commands one by one (using the corresponding absolute paths) at the end of the same entry, such as
```
pluralsight    ALL=(root) /usr/sbin/adduser, /usr/sbin/usermod
```
**or**

- define an *alias* (which we can name as we wish as long as it's all upper case, for example **USERMANAGEMENT**):
```
Cmnd_Alias USERMANAGEMENT = /usr/sbin/adduser, /usr/sbin/usermod
pluralsight    ALL=(root) USERMANAGEMENT
```

While the latter requires two lines, it is often preferred instead of the former since it contributes to keep `/etc/sudoers` cleaner and more readable. In any event, **pluralsight** will not be able to execute any other commands as root other than those specified above.

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

If an account needs to be deleted for good, use `sudo userdel -r` followed by the corresponding username. In this example, the use of `-r` will ensure that all the user's files are removed as well. If you want to keep such files for some reason, omit that option.

## Groups
In Linux, groups can be defined as a way to organize users that need the same type of access to a directory or file. In this section we will explain how to create and remove groups, whereas in *Section 2 - Permissions* a more detailed discussion on user and group permissions will be given, along with file and directory ownership.

To create a new group named **finances**, do `addgroup finances`. To remove it from the system, use `delgroup finances`. The information of the new group is stored in `/etc/group`, where each line shows the name of the group and the user accounts that are associated with it. Similarly, a group can be deleted using `delgroup` followed by the group name.

>It is important that you practice the commands and examples outlined in this section until you feel confident using them. Then proceed with the next section where we will be adding and removing users to and from groups, and granting or preventing access to files and directories.

# Section 2 - Permissions

Every file, directory, and other system objects in Linux are assigned an owner and a group. This is the most basic, yet essential, part of system security that protects users from each other. Owners, users belonging to a group, and all others may be granted different types of access to read from, write to, or execute files. This is generally referred to as *file permissions* in Linux. 
To set permissions and manage ownership, we will use the following commands:
- `chmod`: change file permissions.
- `chown`: change file owner.
- `chgrp`: change group ownership.
- `id`: print user and group IDs.

## Users, Groups, and Everyone Else
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

The first group of 3 characters (`rw-`) indicate that the owner of the file (**pluralsight**) has *read* and *write* permissions, as it is the case with the group (as indicated by the next 3 characters). Finally, the last group (`r--`) means that the rest of the users can only *read* that file - but not *write* to it or *execute* it.

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

>The `-aG` combined option is short for *append to group*. Now **finances** is said to be a *secondary* or *supplementary* group for user **student**. The new access permissions will take effect the next time **student** logs in. At that point, we can use the `id student` command to verify that the account was added correctly to the group. If we're logged in as **student**, doing just `id` will do the trick.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8fa07297-9b0f-4dbe-9360-7dc505fa9c08.png)

If instead of adding **student** to the **finances** group, we wanted to make him the owner of `test1`, we can use `chown` followed by the user and file names, in that order:

```
sudo chown student test1
```
> Note that the above command will cause that **pluralsight** does not have access to `test1` anymore, as such account is now neither the owner of the file or has not been added to the **finances** group.

## Special Permissions

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

## Revisiting /etc/sudoers
Using `/etc/sudoers`, we can also allow **all** the users in a group to execute programs with superuser privileges. For example, the following line indicates that members of **finances** are allowed to run `updatedb` (or more precisely, `/usr/bin/updatedb`):

```
%finances ALL=(ALL) /usr/bin/updatedb
``` 

The only difference when compared to individual users is that the group name must be prefaced by the `%` sign. Command aliases can be used in this case as well.

> To list the allowed commands for the invoking user in `/etc/sudoers`, simply do `sudo -l` in the command line and press Enter.

# Section 3 - Processes

Every program that runs on a Linux system is managed as a process. As such, it is started, executed, and eventually terminates or continues running on the background. In this last case, it is often referred to as a *daemon* or *system service*. This cycle is automatically managed by the kernel, without need for user intervention under normal circumstances. However, a process may require action on our part if it is not behaving as expected, or if it needs to be restarted after changes have been made to its configuration, or when we need to assign more or less system resources to it before or during its execution. 

In this section we are going to introduce the following process management commands:
- `ps`: report a snapshot of the current processes.
- `pstree`: display a tree of processes.
- `top`: display processes and uptime information.
- `nice`: run a program with modified scheduling priority.
- `renice`: alter priority of running processes.
- `kill`: terminate processes.
- `uptime`: how long the system has been running.

## Introducing processes in Linux
When a process is started either manually or during the boot process, it is assigned an unique integer number as identifier called **Process ID** or **PID** for short. If a process **A** launches or starts a process **B**, it is said that **A** is the parent of **B**, or that **B** is the child of **A**. Under this scenario, the **PID** of **A** is the **Parent PID** (or **PPID**) of **B**.

>The first process started by the kernel at boot time (`PID=1`) is called **systemd** in modern Linux distributions. This process is responsible for starting several other processes, which in turn start more, until the operating system is fully functional.

Additionally, each process is associated with the user who started it. This *user* can be an actual person, or an account used by the system for its operation. What we discussed in the previous section (*Section 2 - Permissions*) applies here as well: a process only has access to the system resources that its associated user account is allowed to utilize. In other words, if a process **A** was started by `user1`, and such account does not have write permissions in a given file, the process will not be permitted to write to that file either.

To display a snapshot of all the processes currently running in our system, we will use `ps`. When executed without arguments, this command will return **only** the processes owned by the current user. More useful information is available by using the `aux` or `-ef` options. Since the output in both cases can be **very** long, we can pipe the output to a *pager* such as `less` to inspect it more easily:

```
ps aux | less
```
or
```
ps -ef | less
```
The above commands return more information than what we're usually interested in. Most often we will want to view the **PPID**, **PID**, the command associated with the process (or the absolute path to the executable file), and the percentage of system memory and CPU usage. Each of these fields (and others as well) are explained in detail under the *STANDARD FORMAT SPECIFIERS* section in `man ps`. 

To view only these fields in the output of `ps`, we will use the `-eo` combined option followed by the corresponding format specifiers. For example,

```
ps -eo ppid,pid,cmd,%mem,%cpu | less
```
During a regular inspection or system troubleshooting, it is useful to identify which processes are consuming the most memory. Fortunately, `ps` supports the `--sort` option. When used with the above command, and piped to `head` instead of less, we can easily get a list of the most memory-hungry processes as follows (see Fig. 1):
```
ps -eo ppid,pid,cmd,%mem,%cpu --sort -%mem | head -n 5
```

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/3cd0a77f-bec4-4740-8414-19194a335b6b.png)

Among other things, the above image shows that the **PID** of `snapd` is 940. This process was started by `systemd`, as we can tell from `PPID=1`, and is currently the process that is consuming the most memory.

>The minus sign in `-%mem` after `--sort` indicates that the output should be sorted in *descending form*. Using a plus sign (`+`) instead, will result in the output being sorted in *ascending* form, which is the default behavior.

### Process trees
Given the parent / child relationship between process, it is often necessary to visualize a hierarchical list of processes in the form of a tree. To do this, we will use a tool called `pstree`. If the command is followed by
- A **PID**, then the tree will be displayed with such process at its root. Otherwise, it is rooted at `PID=1`. 
- A username, `pstree` will return all trees own by the user.

In addition to displaying only the name of processes, `pstree` can also return their **PIDs** when used with the `-p` option. Fig. 2 shows all trees owned by **pluralsight**. The images on the left and the right show the output with and without the use of `-p`. The number inside parentheses is the **PID** of each process.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/bbce1994-50f2-41d5-a4d3-8346e64c4885.png)

On the right, we can see that `tmux` (`PID=1470`) is the parent of two `bash` process (`PID=1471` and `PID=1486`). The latter is in turn the parent of `pstree`. This simple example illustrates the usefulness of `pstree` to view at a glance the process trees of a given user.

>Terminating a parent process has the effect of terminating its children as well.

### Top and Uptime
Although it is useful to use `ps` to take snapshots of the process list, it may be necessary to monitor that list dynamically with refreshing intervals. Instead of doing `ps` several times, Linux provides another tool called `top` that is more fit for the job. Additionally, `top` shows the same information as `uptime` as we can see in Fig. 3:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/67080801-0b01-4815-89d0-588bbbc4dd2d.png)

Fig. 3 indicates that the current time is **19:23:57**. The system has been up for **1 hour and 10 minutes** and the load average over the last minute, 5 and 15 minutes has been **0.03** (3%), **0.02** (2%), and **0.00** (0%), respectively. These percentages represent the use of CPU by running processes over the specified time intervals. In a multi-CPU system, these numbers should be divided by the number of processors to find out the average CPU usage. If you consistently experience high CPU usage, it may be time to add hardware resources to the machine or move the more CPU-hungry services to another system.
> Remember to check out `man top` for further details on the use of this tool, particularly the *FIELDS / Columns* section, which describes the available information of processes that are shown by `top`. 

### Process Execution Priorities
Without any intervention on our part, the kernel will adjust automatically the CPU cycles that are allocated to a certain process. However, Linux also allows the system administrator to change this behavior through the `nice` (for new processes) and `renice` (for running processes) commands. Both tools use an integer value in the range of -20 to 19 to modify the *priority* that should be given to a process in terms of CPU usage. The lower the value, the less *nice* the application - which means it will take up more system resources.

> A regular user can only increase the *niceness* of a process he or she owns. He or she cannot either decrease it or change the priority of processes owned by other users. These are other administrative tasks that must be performed either as root or using `sudo` as explained earlier.

To *renice* a process, use `renice` followed by the new *niceness* value, and `-p [PID]` where `[PID]` is the **PID** of the process. For example, let's increase the priority of `snapd` from **0** (current) to **-5**. Fig. 4 shows the change of the *niceness* value, and the use of `ps` with another format specifier (`ni`) to return this information. Also, `--pid` followed by a **PID** allows us to view the process fields of the corresponding process only.

```
ps -o pid,cmd,ni --pid=940
sudo renice -5 -p 940
ps -o pid,cmd,ni --pid=940
```

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/a85a5f48-7bf6-4a1a-a98d-50718a00d2b7.png)

After changing the *niceness* value of `snapd`, the kernel will start allocating more CPU cycles for it.

### Killing processes
Processes can be forced to terminate by sending them *signals* using `kill` or `killall` commands. The former requires that the **PID** of the process that we want to manipulate, whereas the latter allows us to terminate processes by name or by specifying owning user. 

> Terminating a process manually should only be considered when it has failed to respond or is interfering with the normal operation of other processes. For example, an outdated or failed plugin running on a web browser can cause the browser -and ultimately the system- to crash. Under this scenario, the memory usage typically skyrockets. Before that happens, it is a good idea to terminate (aka *kill*) the browser process and uninstall the offending plugin as soon as possible.

To kill a process by PID (say `PID=1025`), do `sudo kill -SIGTERM 1025`. If it ignores this signal, you can resort to a more drastic measure by using `-SIGKILL` instead: `sudo kill -SIGKILL 1025`. Please note, however, that the use of **SIGTERM** is preferred as it gives the process a chance to close any files it may be using (aka *clean termination*). 

If we want to terminate all the processes owned by a particular user, `killall` is more appropriate. The command should be followed by `-u [USER]`, where `[USER]` is the owner of the processes. For example, `sudo killall -u www-data` will terminate all processes owned by **www-data**. By the way, this account is the default owner of the Apache web server on Debian, Ubuntu, and similar distributions.

# Section 4 - Shell Scripting With Bash

Shell scripts are plain text files that contain a sequence of commands that are run by a shell one after another. Since Bash is the default shell in most modern Linux distributions, in this section we will explain how to leverage its programming capabilities to create simple scripts. As we gain experience, we can use what we've learned to develop more robust programs. 

>System administrators often use shell scripts to automate routinary tasks. As a rule of thumb, if a task has to be performed periodically (even when it's only once a month), it needs to be automated. In *Section 5 - Scheduling And Running Tasks With Cron* we will learn how to accomplish this goal.

## Structure of a Shell Script
The first line in a shell script must indicate the shell (aka *interpreter*) that will be used to execute it. For Bash, this means
```
#!/bin/bash
```
The above line then must be followed by the commands that should be run by the shell. Although preferences between system administrators may vary, effective and well-maintained shell scripts often include the following sections.

### Header
The script header is a commented-out section where the developer can include items such as:
- Description / purpose of the script
- Revision history
- License terms and / or copyright notice

>The shell ignores blank and commented-out lines. The former are only meant as information for the author, other reviewers, and people using the program. To comment out a line, simply place a `#` sign at the beginning.

A common header looks as follows:

```
# ======================================================================
# SCRIPT NAME: systeminfo.sh

# PURPOSE: Demonstrate simple Bash programming concepts

# REVISION HISTORY:

# AUTHOR				DATE			DETAILS
# --------------------- --------------- --------------------------------
# Gabriel A. Cánepa	 2017-10-21	  Initial version

# LICENSE: CC Attribution-ShareAlike 4.0 International
# ======================================================================
```

### Body
The body of the script is where the sequence of commands is placed one per line. A command can be executed directly by the shell or have its output be saved into a container known as *variable*. You can think of variables as boxes where we can store a fixed value (such as text or a number), or the output of a command, for later reuse. As a best practice, system administrators often use comments in the body to indicate what a given line of code is supposed to do. This also serves as a reminder for himself / herself and others who will later work on the same file.

>To store the output of a command in a variable, enclose the command between parentheses and preface them with the dollar sign. Thus, in `MYVAR=$(command)`, the variable `MYVAR` contains the output of `command`, where `command` can be any command executed by the shell. To use the contents of `MYVAR` in the script, add `$MYVAR` wherever it is needed.

For example:

```
echo "Starting to run the script..."
# VARIABLE ASSIGNMENT
# Show hostname:
HOST=$(hostname)
# User executing the script:
CURRENTUSER=$(whoami)
# Current date:
CURRENTDATE=$(date +%F)
# Host IP address:
IPADDRESS=$(hostname -I | cut -d ' ' -f1)

# SHOW MESSAGES
echo "Today is $CURRENTDATE"
echo "Hostname: $HOST ($IPADDRESS)"
echo "User info for $CURRENTUSER:"
grep $CURRENTUSER /etc/passwd
```

Let's now put everything together (`#!/bin/bash`, header and body) and save as `systeminfo.sh`. Next, make the file executable and run it:
```
chmod a+x systeminfo.sh
./systeminfo.sh
```
Fig. 1 shows the output of the script:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/d2407a59-986a-4962-9f65-362d553ebbfe.png)

>As you can see in the above image, the `echo` Bash builtin allows to embed variables and fixed text inside double quotes. Thus, the content of the given variable(s) will be part of the output, along with the text.

## Conditional Statements
When you need to execute different series of commands depending on a condition, the shell provides a standard `if`/`elif`/`else` construct, similar to most programming languages. If you realize there are several conditions to be examined, you may want to use a `case` construct instead, as explained later.

The syntax of a basic conditional statement is shown in *pseudo-code* as follows. Note that there must be a space after the opening bracket **and** before the closing one:

```
if [ condition 1 ]
then
	commands
elif [ condition 2 ]
	commands
else
	commands
fi
```

> The `elif` section is optional and used only when needed to test for a specific alternative to the condition specified through the `if`.

```
#!/bin/bash
CURRENTDAYOFTHEMONTH=$(date +%d)

if [ $CURRENTDAYOFTHEMONTH -le 10 ]
then
    echo "We are within the first 10 days of the month";
elif [ $CURRENTDAYOFTHEMONTH -le 20 ]
then
    echo "We are within the first 20 days of the month";
else
    echo "We are within the last 10 days of the month";
fi
```

The above script returns different messages depending on the current day of the month. The first `if` tests whether the current day is less or equal to 10. If this condition evaluates to true, the message `We are within the first 10 days of the month` is displayed, and the other conditions are not tested. Otherwise, `elif` checks if `$CURRENTDAYOFTHEMONTH` is less or equal to 20. If that is the case, `We are within the first 20 days of the month` will be returned instead. If none of the previous conditions are met, the default `else` block will apply, returning `We are within the last 10 days of the month`.

>If several alternative conditions need to be considered, we can still use several `elif` blocks. However, it's cleaner to use `case`, as explained next.

## Loops and Case
Once in a while, some tasks will need to be executed repeatedly either a fixed number of times or until a specified condition is satisfied. That is where the `for` and `while` *looping constructs* come in handy. Additionally, at times you will need a script to *choose* between different courses of action based on the value of a given variable - we'll use the `case` statement for that.

### The For Loop
This looping construct is used to operate on each element of a list of items. Such list can be specified explicitly (by listing each element one by one), or as the result of a command. The basic syntax of the `for` loop is illustrated in the following *pseudo-code*:

```
for variable-name in list
do
    Run command on variable-name as $variable-name
done
```

where `variable-name` is an arbitrary name that represents an item in `list` during each iteration.

For example, we can change the permissions on files `file1.txt`, `file2.txt`, and `file3.txt` to `640` using a `for` loop very easily:

```
for FILE in file1.txt file2.txt file3.txt
do
    chmod 640 $FILE
done
```

In the above example, the variable named `FILE` is used inside the loop (between the `do` and `done` lines) as `$FILE`. During the first, second, and third iteration, `$FILE` represents `file1.txt`, `file2.txt`, and `file3.txt`, respectively.

An alternative approach is to provide the list of files using the `ls` command and `grep` to only return the files where the names begin with the word `file`. Thus,

```
#!/bin/bash
for FILE in $(ls -1 | grep file)
do
    chmod 640 $FILE
done
```

produces the same result as the previous example.

### The While Loop
As opposed to the `for` loop, `while` is typically used when the number of iterations is not known beforehand, or when using `for` is impractical. Examples include, but are not limited to, reading a file line by line, increasing or decreasing the value of a variable until it reaches a given value, or responding to user input. 

The basic syntax of the `while` loop is:

```
while condition is true
do
    Run commands here
done
```

To illustrate, let's consider two examples. First off, let's read the `/etc/passwd` file line by line and return a message showing each username with its corresponding **UID**. This is an example of the case when we need to repeat an iteration an undetermined number of times.

>The **UID**, or **U**ser **ID**, is an integer number that identifies each user in the *third* field in `/etc/passwd`. It can be returned for each individual account using the `id` command followed by the username.

```
#!/bin/bash
while read LINE
do
    USERNAME=$(echo $LINE | cut -d':' -f 1)
    USERID=$(echo $LINE | cut -d':' -f 3)
    echo "The UID of $USERNAME is $USERID"
done < /etc/passwd
```

In this example, the condition that is checked at the beginning of each iteration is whether we have reached the end of the file. The variable named `LINE` represents each line in `/etc/passwd`. This file is set as the input to the loop by using the `<` redirection operator. When each line is read, the contents of the first and third fields are stored in `USERNAME` and `USERID`, respectively.

Fig. 2 shows the output of the while loop, which we have saved in a script named `users.sh` in the current working directory:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/53c8319e-5c70-4b57-8ad6-d4c08958ba5e.png)

Finally, let's write a simple number-guessing game in a script called `guess.sh`. When the script is launch, a random number between 1 and 10 is generated and stored in the variable `RANDOMNUM`. The script then will expect input from the user and indicate if the guess is correct, less than, or greater than the right number. This will continue until the user guesses the number correctly. In this example, `$NUMBER != $RANDOMNUM` is enclosed within square brackets to indicate accurately what is the condition to test.

```
#!/bin/bash
RANDOMNUM=$(shuf -i1-10 -n1)

NUMBER=0

while [ $NUMBER != $RANDOMNUM ]
do
    read -p "Enter a number between 1 and 10: " NUMBER
done
echo "Congratulations! Your guess was right!"
```

Fig. 3 shows our game in action:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/10ce3c16-3c0b-4f66-9e0d-c8f92315169f.png)

### Using Case

# Section 5 - Scheduling And Running Tasks With Cron



# Section 6 - Managing Software Using Yum Or Aptitude



# Section 7 - Setting Up And Securing An OpenSSH Server



# Section 8 - RAID



# Section 9 - LVM



# Section 10 - Disk Quotas