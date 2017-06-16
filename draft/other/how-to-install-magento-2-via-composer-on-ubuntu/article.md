# Install Magento 2 via Composer (ubuntu)

#### This Guide will help you to setup magento 2 for LAMP distributed machines as it will work for most of the ubuntu versions.

#### This guide assumes that lamp server is already installed on your ubuntu machine and have the basics understanding of apache/php/mysql and linux commands.

#### Prerequisite for installing Magento 2.
    ###### PHP version 5.6 and above (5.6.x or higher, 7.0.2–7.0.6 except for 7.0.5).
    ###### Mysql version 5.6 and above.
    ###### Apache version 2.2 and above.
    ###### Composer stable version.
    
#### Lets start with our INSTALLATION.

#### 1.PHP configuration
As magento is totally dependend on PHP configurations , so to setup such a heavy application(magento) we need to increase the memory limit(memory_limit).

First, open the php.ini config file with a text editor and copy the below code and press enter.

`sudo vim /etc/php(version)/apache2/php.ini`

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/9f4f2f83-bb24-4458-ba50-00c912695a64.png)


Second, Before you install Magento 2, make sure your system meets or exceeds the following requirements:
Required PHP extensions:

- PDO/MySQL
- mbstring
- mcrypt
- mhash
- simplexml
- curl
- gd2, ImageMagick 6.3.7 (or later) or both
- soap

On terminal execute the below commands one by one.

`sudo apt-get update`

`sudo apt-get install php7.0-gd php7.0-mcrypt php7.0-curl php7.0-intl php7.0-xsl php7.0-mbstring php7.0-openssl php7.0-zip`

once the php extensions are downloaded and configurations are set properly
we need to restart our apache server. Execute the below command.

`sudo service apache2 restart`

#### 2.Install Composer
Install composer with the following command:

`curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer`

To check if composer is installed properly , Execute the below code.

`composer list`

if you see the below screen

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c780844a-0c15-4605-bd9d-11a6157afdd8.png)

yes....Good to go.

#### 3.Mysql Setup
We need to create a database for our magento 2 project.
We can create either from terminal or from our browser phpmyadmin whichever is more feasible.

CREATE DATABASE magento2;

#### 4.Installing Magento 2

Navigate to the web directory.

$ cd /var/www
	
Clone the Magento Github repo.

$ git clone -b 2.0 https://github.com/magento/magento2.git magento2

once completed Navigate into the cloned folder.

i.e cd /var/www/magento2

now execute the code `composer install` .

This will download all the modules dependencies required for our magento 2 project.

Next to complete our installation process.

Execute the below code.
**Fill all the details given with the code**

`php bin/magento setup:install --backend-frontname=admin --db-name=dbname --db-password="pwd" --base-url=url --base-url-secure=url --admin-user=admin --admin-password=admin123 --admin-email=email --admin-firstname=name --admin-lastname=name --language=en_US --currency=USD --timezone=America/Chicago --use-rewrites=1`   

Your magento 2 installation process has started.

If you see the below screen.

````
Post installation file permissions check...

For security, remove write permissions from these directories: '/var/www/html/magento2/app/etc' `[Progress: 274 / 274]

[SUCCESS]: Magento installation complete.

[SUCCESS]: Admin Panel URI: /admin
````

Congratulations! Magento 2 installation has been successfully finished.

### 5. The Final Step (Create Virtual Host).

Execute the below code

`sudo vim /etc/apache2/sites-available/magento2.conf`

now copy the below code in your editor and save it
 
````
<VirtualHost *:80>
		ServerName local.magento2.com
		ServerAlias local.magento2.com
	    DocumentRoot /var/www/magento2/
		    <Directory /var/www/public/>
		        Options Indexes FollowSymLinks MultiViews
		        AllowOverride All
		    </Directory>
</VirtualHost>
````
Next, we have to tell Apache to use the new config file, and to ignore the default config file. Execute the following commands below one by one:

`sudo a2ensite magento2.conf`

`sudo a2dissite 000-default.conf`

Now execute the below code.

`sudo vim /etc/hosts` and make your host entry(i.e local.magento2.com)

copy the below code and save it.
````
127.0.0.1    local.magento2.com
````

Finally we need to reload our apache.
Execute the below command.

`sudo service apache2 reload`

Thats all FOLKS!!!
Clear magento **cache** and yup we are all set.

Your frontend url is **local.magento2.com** 

Your backend url using the previously used login and password. **local.magento2.com/admin**

--admin-user=admin

--admin-password=admin123

**NOTE** : we have installed Magento 2 without sample data. 



