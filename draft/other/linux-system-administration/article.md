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



# Section 2 - Permissions



# Section 3 - Processes



# Section 4 - Shell Scripting With Bash



# Section 5 - Scheduling And Running Tasks With Cron



# Section 6 - Managing Software Using Yum Or Aptitude



# Section 7 - Setting Up And Securing An OpenSSH Server




# Section 8 - RAID




# Section 9 - LVM



# Section 10 - Disk Quotas