### Introduction
Ruby on Rails is an extremely popular open-source web framework that provides a great way to write web applications with Ruby.

This tutorial will show you how to install Ruby on Rails on CentOS 7, using rbenv.

### Prerequisites
Before installing rbenv, you must have access to a superuser account on a CentOS 7 server.

### Install rbenv
Install the rbenv and Ruby dependencies with yum:
```shell
sudo yum install -y git-core zlib zlib-devel gcc-c++ patch readline readline-devel libyaml-devel openssl-devel make bzip2 autoconf automake libtool bison curl sqlite-devel
```
---
Now we are ready to install rbenv. The easiest way to do that is to run these commands, as the user that will be using Ruby:
```shell
cd
git clone git://github.com/sstephenson/rbenv.git .rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
exec $SHELL

git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bash_profile
exec $SHELL
```
This installs rbenv into your home directory, and sets the appropriate environment variables that will allow rbenv to the active version of Ruby.

Note that you might need to reboot the server machine in order for the new environment variables to take effect.

### Install Ruby
Before using rbenv, determine which version of Ruby that you want to install. We will install the latest version as of now, which is Ruby 2.3.0.
```shell
rbenv install -v 2.3.0
rbenv global 2.3.0
```
Verify that Ruby was installed properly with this command:
```shell
ruby -v
```
It is likely that you will not want Rubygems to generate local documentation for each gem that you install, as this process can be lengthy. To disable this, run this command:
```shell
echo "gem: --no-document" > ~/.gemrc
```
You will also want to install the bundler gem, to manage your application dependencies:
```shell
gem install bundler
```
Now that Ruby is installed, let's install Rails.

### Install Rails
As the same user, install Rails 4.2.5 with this command:
```shell
gem install rails -v 4.2.5
```
Whenever you install a new version of Ruby or a gem that provides commands, you should run the `rehash` sub-command. This will install *shims* for all Ruby executables known to rbenv, which will allow you to use the executables:
```shell
rbenv rehash
```
Verify that Rails has been installed properly by printing its version:
```shell
rails -v
```
If it's installed properly, you will see this output: `Rails 4.2.5`

# Install Javascript Runtime
A few Rails features, such as the asset pipeline, depend on a Javascript runtime. We will install Node.js to provide this functionality.
<br><br>
Add the EPEL yum repository.
```shell
sudo yum -y install epel-release
```
Then install the Node.js package:
```shell
sudo yum install nodejs
```
**Note:** This will probably not install the latest release of Node.js, as Enterprise Linux does not consider it to be "stable". If you want to install the latest version, feel free to build it on your own.
<p>
Congratulations! Ruby on Rails is now installed on your system.

# Configuring Git
A good version control system is essential when coding applications. Follow the [How To Set Up Git](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-centos-7#set-up-git) section of the How To Install Git tutorial.

# Setting Up a Database

Rails ships with sqlite3 as its default database. Chances are you will not want to use it because it's stored as a simple file on disk and not suitable for a bigger application. You'll probably want something more robust like MySQL or PostgreSQL.
<p>
In my experience (it might sound opinionated), PostgreSQL is better than MySQL in terms of performance and scalability, so I'm going to cover installation of PostgreSQL only in this tutorial.

CentOS's default repositories contain Postgres packages, so we can install them without a hassle using the `yum` package system.
```shell
sudo yum install postgresql-server postgresql-contrib
```
<p>
Accept the prompt by responding with a `y`.
<p>
Now that our software is installed, we have to perform a few steps before we can use it.
<p>
Create a new PostgreSQL database cluster:
```shell
sudo postgresql-setup initdb
```
<p>
By default, PostgreSQL does not allow password authentication. We will change that by editing its host-based authentication (HBA) configuration.
<p>
Open the HBA configuration with your favorite text editor. We will use vim:
```shell
sudo vim /var/lib/pgsql/data/pg_hba.conf
```
<p>
Find the lines that looks like this, near the bottom of the file:
```
host    all        all        127.0.0.1/32        ident
host    all        all        ::1/128             ident
```
<p>
Then replace "ident" with "md5", so they look like this:
```
host    all        all        127.0.0.1/32        md5
host    all        all        ::1/128             md5
```
<p>
If you want to use password authentication when connecting to postgres server in terminal, you need to make the following changes as well:
<p>
`original`
```
local    all        all                           peer
```
<p>
`updated`
```
local    all        all                           md5
```
<p>
In most cases, we are installing postgres on remote server (VPS etc) and we might want to connect to the server using postgres client software, such as Navicat.
<br>
In order to enable remote access to PostgreSQL database server, we have to the following things:

**Add the following line to `pg_hba.conf` file**
```
host    all        all        0.0.0.0/0           md5
```
`0.0.0.0/0` indicates that postgres server will allow remote access from any IP addresses.
You can instead write specific IP addresses if you don't want to allow arbitrary access.
<p>
**Modify `postgresql.conf` file**
```shell
vim /var/lib/pgsql/data/postgresql.conf
```
Find the line `listen_addresses='localhost'` and change it to `listen_addresses='*'`.
Or just bind to specific IP addresses, separated by space.
<p>
**Open the port 5432**
```shell
sudo iptables -I INPUT -p tcp --dport 5432 -j ACCEPT
```
----------
Now we can start and enable PostgreSQL:
```shell
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

# Install postgres gem to our Rails app
```shell
sudo yum install postgresql-devel
gem install pg
```
Now we can use PostgreSQL with our Rails application.

# Create a Test Application
```shell
cd ~
rails new myapp -d postgresql
cd myapp
rake db:create
rails s -b <SERVER's PUBLIC IP>
```
Rails starts the server at the default port 3000. In order to allow remote client to access to the server, we need to modify iptables just like postgres.
```shell
sudo iptables -I INPUT -p tcp --dport 3000 -j ACCEPT
```
And then restart our rails server.
<p>
Now on your favorite browser, hit `http://<SERVER's PUBLIC IP>:3000`.
<p>
If you see the Rails "Welcome aboard" page, your Ruby on Rails installation is working properly! Hoorah!

# Conclusion
You're now ready to start developing your new Ruby on Rails application. Good luck! And happy coding!
<p>
;)