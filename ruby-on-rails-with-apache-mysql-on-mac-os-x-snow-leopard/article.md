![](https://s3.amazonaws.com/static.written.com/rails_with_mysql1416261120.jpg "rails_with_mysql")

### Install a Path Line Using Text Editor

You can use an editor such as TextMate, TextWrangler, BBEdit or vi.
(My favorite is TextMate and `mate` command or just use `vim` command).

### Remove Mysql 5.5 if you have (Mysql 5.1 is the most stable version)

For some reason I faced some problems with MySQL 5.5 so I installed the most stable version : MySQL 5.1 64 Bit.

Use mysqldump to backup your databases to text files! After that :

``` bash
$ sudo /usr/local/mysql/support-files/mysql.server stop
$ sudo rm /usr/local/mysql
$ sudo rm -rf /usr/local/mysql*
$ sudo rm -rf /Library/StartupItems/MySQLCOM
$ sudo rm -rf /Library/PreferencePanes/My*
$ rm -rf ~/Library/PreferencePanes/My*
$ sudo rm -rf /Library/Receipts/mysql*
$ sudo rm -rf /Library/Receipts/MySQL*
$ sudo rm -rf /var/db/receipts/com.mysql.*
$ mate /etc/hostconfig
```

Remove the line `MYSQLCOM=-YES-`

### Download and install Mysql 5.1

[http://www.mysql.com/downloads/mysql/5.1.html#downloads](http://www.mysql.com/downloads/mysql/5.1.html#downloads)




### Configure Path , download and Install Macports

``` bash
$ mate ~/.bash_profile
```

Add the following to the end of file and save

``` bash
export PATH=/opt/local/bin:/opt/local/sbin:$PATH
export MANPATH=/opt/local/share/man:$MANPATH
```

[http://www.macports.org/install.php](http://www.macports.org/install.php)

(MacPorts-1.9.2-10.6-SnowLeopard.dmg)

### Download and Install Ruby

``` bash
$ sudo port -v install ruby
```


### Download , Install and Update RubyGems

``` bash
$ mkdir ~/src
$ cd ~/src
$ curl -O http://production.cf.rubygems.org/rubygems/rubygems-1.3.7.tgz
$ tar xzvf rubygems-1.3.7.tgz
$ cd rubygems-1.3.7
$ sudo ruby setup.rb
$ sudo gem update --system
```


### Install rails, rake, bundle, rspec etc.

``` bash
$ sudo gem install rake rails thin tzinfo capistrano ruby-debug rspec
```

### Install Mysql Gem

``` bash
$ sudo env ARCHFLAGS="-arch x86_64" gem install --no-rdoc --no-ri mysql -- --with-mysql-config=/usr/local/mysql/bin/mysql_config
```

You can use `sudo gem uninstall mysql` to uninstall.

### Test mysql gem

``` bash
$ irb
$ irb > require 'rubygems'
$ irb > require 'mysql'
$ irb > Mysql.new('localhost', 'root', 'password', 'test')
```

### Install Sqlite gem

``` bash
$ sudo gem install sqlite3-ruby
```

### Install Passenger gem

``` bash
$ sudo gem install passenger
```

### Build passenger for apache

(Just hit enter on prompts)

``` bash
$ sudo passenger-install-apache2-module
```

### Include Passenger config file

``` bash
$ mate /private/etc/apache2/httpd.conf
```

Add the following to the end of file and save

``` bash
Include /private/etc/apache2/extra/httpd-passenger.conf
```

### Create Passenger config file for Apache:

My Passenger version is : 3.0.2 so you should edit the code below if yours is a different version

``` bash
$ mate /etc/apache2/extra/httpd-passenger.conf
```

Add the following to the file and save (Change passenger directory string if your have a different version. Learn it by typing `passenger-config --root`)

``` bash
LoadModule passenger_module /opt/local/lib/ruby/gems/1.8/gems/passenger-3.0.2/ext/apache2/mod_passenger.so
```

``` ruby
PassengerRoot /opt/local/lib/ruby/gems/1.8/gems/passenger-3.0.2
PassengerRuby /opt/local/bin/ruby
PassengerMaxPoolSize 6
PassengerMaxInstancesPerApp 2
RailsFrameworkSpawnerIdleTime 1800
RailsAppSpawnerIdleTime 600
PassengerPoolIdleTime 600
PassengerMaxRequests 1000
NameVirtualHost *:80
DocumentRoot "/Users/goksel/Sites/rails_app/public"
ServerName rails_app
Options -MultiViews
AllowOverride All
```


### Add your local domain to hosts:

``` bash
$ mate /etc/hosts
```

Add the following to the end of file and save

``` bash
127.0.0.1 rails_app
```

### Restart Apache

``` bash
$ apachectl configtest
```

(If you see “Syntax OK” everything is done right)

``` bash
$ sudo apachectl stop
$ sudo apachectl start
```

### Create a Mysql User if you need one

``` bash
$ mysql -u root -p
```

(If you dont have a root password you should create it)

``` sql
CREATE USER 'railsuser'@'localhost' IDENTIFIED BY 'pass';
FLUSH PRIVILEGES;
exit
```

### Create a Rails app

``` bash
$ cd ~/sites
$ rails new rails_app
```

### Configure database.yml

``` bash
$ cd rails_app
$ mate config/database.yml
```

Remove the content and add the following into file

``` ruby
development:
  adapter: mysql
  database: railsapp_dev
  username: root
  password: password
  host: localhost
test:
  adapter: mysql
  database: railsapp_test
  username: root
  password: password
  host: localhost
production:
  adapter: mysql
  database: railsapp_prod
  username: root
  password: password
  host: localhost
```

### Create an empty database

``` bash
cd ~/sites/rails_app
rake db:create
```

### Test your application

[http://rails_app/](http://test_app/)

