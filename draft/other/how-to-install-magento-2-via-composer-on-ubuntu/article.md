# Install Magento 2 via Composer

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

First, open the php.ini config file with a text editor and copy the below code and enter.

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

On terminal copy the below commands one by one and press enter

`sudo apt-get update`

`sudo apt-get install php7.0-gd php7.0-mcrypt php7.0-curl php7.0-intl php7.0-xsl php7.0-mbstring php7.0-openssl php7.0-zip`

once the php extension are downloaded and configuration set properly
we need to restart our apache server.copy the below code and press enter.

`sudo service apache2 restart`

#### 2.Install composer

`curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer`

To check if composer is installed properly , type in your terminal

`composer list`

if you see the below screem

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c780844a-0c15-4605-bd9d-11a6157afdd8.png)

Good to go.

#### 3.Mysql Setup
We need to create a database for our magento 2 project.
We can create either from terminal or from our browser phpmyadmin whichever is more feasible.

CREATE DATABASE magento2;

#### 4.Installing Magento 2

CD to the web directory.

$ cd /var/www
	
Clone the Magento Github repo.

$ git clone -b 2.0 https://github.com/magento/magento2.git magento2

once completed CD into the cloned folder.

i.e cd /var/www/magento2

type `composer install` and press enter .
This will download all the module dependencies required for our magento 2 project.

Next to complete our installation process

copy the below code and fill proper details as per requirement. 

php bin/magento setup:install --backend-frontname=admin --db-name=dbname --db-password="pwd" --base-url=url --base-url-secure=url --admin-user=admin --admin-password=admin123 --admin-email=email --admin-firstname=name --admin-lastname=name --language=en_US --currency=USD --timezone=America/Chicago --use-rewrites=1   and press enter

Your magento 2 installation is started.

If you see the below screen.

Post installation file permissions check...

For security, remove write permissions from these directories: '/var/www/html/magento2/app/etc' `[Progress: 274 / 274]

[SUCCESS]: Magento installation complete.

[SUCCESS]: Admin Panel URI: /admin

Congratulations! Magento 2 installation has been successfully finished.

Now you can enter backend using the previously used login and password.

--admin-user=admin

--admin-password=admin123




