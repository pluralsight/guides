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

### 2.Install composer

`curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer`

We need to generate a keypair to clone the Magento repo. Since this requires authentication, see the Magento developer documentation for the latest instructions.

Next, place these keys in the auth file for composer using the command below.

`sudo nano /root/.composer/auth.json`
	
Copy and paste the contents below into the file.

{
"http-basic": {
      "repo.magento.com": {
         "username": "your public key",
         "password": "your private key"
      }
   }
}	

To check if composer is installed properly , type in your terminal

`composer list`

if you see the below screem


