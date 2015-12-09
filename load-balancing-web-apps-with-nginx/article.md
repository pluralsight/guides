![loadbalancing](https://hackhands.com/data/blogs/ClosedSource/assets/load-balancing-web-apps-nginx/loadbalancing.jpg)

Imagine you run a web app that serves a decent amount of traffic and has a loyal user base. One day, one of your users posts about your app on Reddit and boom! It hits the front page. You’re getting swamped with users coming to your app, and you can’t handle all the traffic. What do you do? Well, easy - just load balance your servers!

Load balancing is the process in which you utilize multiple servers to serve a single application. It helps scale your application to accurately and reliably match your capacity with the current level of demand. Load balancing web applications isn't a particularly difficult task, but the process can seem daunting. The advantage of manually setting up load balancing for your app is that it ends up being much cheaper and you get more control than services like Heroku provide. All you need to do is sync the files, use a shared database and everything else follows!

If we dissect a typical web application, nearly everything dynamic is done in the database or in-memory, and the rest is done on the filesystem.

We're going to go through the process of setting up two things:

* 2 Nginx load balancers
* 1 MySQL Master

You can add new load balancers whenever you want, but you might hit other constraints such as the database capacity.

I'm going to use Vultr (my choice of VPS host but there are many others out there) and Debian 7 Linux (Wheezy), but it shouldn't differ too much for Ubuntu Linux or any other Linux distribution.

For the Web Application I’m going to use everyone’s favorite blog, Wordpress!

## Getting started


 | Hostname | IP Address |
 | -------- | ----------- |
 | HostA | 10.0.0.5 |
 | HostB | 10.0.0.6 |


First, we need to install Nginx and PHP-FPM to each of these boxes and MySQL on one of them. PHP-FPM is the process manager for PHP, and it’s a replacement of php-fastcgi. We also need some users for syncing, and finally we need MySQL to store information about the wordpress install. So we run:


``` bash
root@hosta:~$ sudo apt-get install mysql-server nginx php5-fpm php5-mysql lsyncd
root@hostb:~$ sudo apt-get install nginx php5-fpm php5-mysql lsyncd
```

``` bash
root@hosta:~$ adduser wordpress
root@hostb:~$ adduser wordpress
```

## File syncing

Next up, we know Wordpress does indeed write to the underlying file system so we need to setup some file mirroring. I’m going to use [lsyncd](https://github.com/axkibe/lsyncd) for this because I prefer how easy it is to use, but there are many other tools out there.

We need to first create the wordpress directory, so we run:


``` bash
root@hosta:~$ sudo mkdir -p /var/www/wordpress; chown -R wordpress:www-data /var/www/
root@hostb:~$ sudo mkdir -p /var/www/wordpress; chown -R wordpress:www-data /var/www/
```


Additionally, we need to generate a pair of SSH keys for the wordpress user to allow the servers to transfer files:


``` bash
wordpress@HostA:~$ ssh-keygen
wordpress@HostB:~$ ssh-keygen
```


We then copy the keys across to each server:

``` bash
wordpress@HostA:~$ ssh-copy-id root@10.0.0.6
wordpress@HostB:~$ ssh-copy-id root@10.0.0.5
```


To get lsyncd to log, we need to create the log directory and log files:

``` bash
wordpress@HostA:~$ mkdir -p /var/log/lsyncd/; touch /var/log/lsyncd/lsyncd.{log,status}
wordpress@HostB:~$ mkdir -p /var/log/lsyncd/; touch /var/log/lsyncd/lsyncd.{log,status}
```


I’ve gone ahead and created a config file for lsyncd on Gist.  You can download it by running the following:


``` bash
wordpress@HostA:~$ mkdir /etc/lsyncd/; wget https://gist.githubusercontent.com/JamieH/c7f50ca56585c25e04fb/raw/22525f41c19bed8dea08905d8e14e1152f663d50/lsyncd.conf.lua -O /etc/lsyncd/lsyncd.conf.lua
wordpress@HostB:~$ mkdir /etc/lsyncd/; wget https://gist.githubusercontent.com/JamieH/c7f50ca56585c25e04fb/raw/22525f41c19bed8dea08905d8e14e1152f663d50/lsyncd.conf.lua -O /etc/lsyncd/lsyncd.conf.lua
```


**You need to edit each file and modify the IP of the corresponding server.** So on HostA, the “host” part should be : 10.0.0.6\.  On HostB the “host” part should be: 10.0.0.5

We also need to edit the /etc/init.d/lsyncd file so that the files are transferred across as the wordpress user.


``` bash
root@HostA:~$ wget https://gist.githubusercontent.com/JamieH/d9dafdd78df0324a37f4/raw/7a6829715480f8342f37709eb463ae6ec86fe4be/lsync -O /etc/init.d/lsyncd
root@HostB:~$ wget https://gist.githubusercontent.com/JamieH/d9dafdd78df0324a37f4/raw/7a6829715480f8342f37709eb463ae6ec86fe4be/lsync -O /etc/init.d/lsyncd
```


This revised startup script ensures that the daemon runs as the Wordpress user instead of root. Once you’ve done this we can safely restart the daemon for it by running:


``` bash
root@HostA:~$ sudo service lsyncd restart
root@HostB:~$ sudo service lsyncd restart
```


At this stage we should make sure that lsyncd starts at system startup, should one or both of our servers go down:


``` bash
root@HostA:~$ sudo update-rc.d lsyncd defaults
root@HostB:~$ sudo update-rc.d lsyncd defaults
```


Lets go ahead and test if this works by running:


``` bash
root@HostA:~$ touch HostA{0..10}
root@HostB:~$ touch HostB{0..10}
```


Be mindful that it can take around 30 seconds for this to sync. It could take longer depending on latency, network speed and the amount of content you are transferring.

## Downloading Wordpress on HostA and Configuring Nginx

So pat yourself on the back if you got file syncing working! Next up we need to install Wordpress on HostA only. So we need to set up nginx to serve the /var/www/wordpress folder. If you already have a site setup in nginx just change the root to our /var/www/wordpress folder. Otherwise you may need to do adjust your existing config.

We should download wordpress first so that we can give it time to sync, so lets run:


``` bash
wordpress@HostA:~$ wget -qO- https://wordpress.org/latest.tar.gz | tar xvz -C /var/www/
```


Ideally you should research how to do configure nginx yourself as it is quite important but to save some time, I’ve created a simple script you can locate [here](https://gist.githubusercontent.com/JamieH/88c7dd3469083c06e3fd/raw/664e09288370abb0a3925e0999c603694ab593cf/nginx.conf).  Be sure to delete the default config in /etc/nginx/sites-enabled/.  You can get the sample config file and delete the default one by running:


``` bash
root@HostA:~$ wget https://gist.githubusercontent.com/JamieH/88c7dd3469083c06e3fd/raw/664e09288370abb0a3925e0999c603694ab593cf/nginx.conf -O /etc/nginx/sites-enabled/wordpress; rm /etc/nginx/sites-enabled/default
```


Next, we need to edit the file to fit the specifics of our setup.  Make sure you replace the IP at the end of server_name with your domain name/IP.  We can restart nginx by running:


``` bash
root@HostA:~$ service nginx reload
```


Let’s push this to the other host as well:


``` bash
root@HostB:~$ wget https://gist.githubusercontent.com/JamieH/88c7dd3469083c06e3fd/raw/664e09288370abb0a3925e0999c603694ab593cf/nginx.conf -O /etc/nginx/sites-enabled/wordpress; rm /etc/nginx/sites-enabled/default
```


(You may need to remove the default config)

If for some reason you are not using a domain name you will need to edit the IP in the config file.

## Setting Up MySQL Server

Before we can actually install Wordpress we need to have MySQL working. We installed MySQL Server earlier so we should be able to connect to it using the password we setup earlier.


``` bash
root@HostA:~$ mysql -u root -p
```


Let’s go ahead and create a database and a user for Wordpress.


``` bash
mysql> CREATE DATABASE wordpress;
Query OK, 1 row affected (0.00 sec)

mysql> CREATE USER wordpress@10.0.0.5;
Query OK, 0 rows affected (0.00 sec)

mysql> SET PASSWORD FOR wordpress@10.0.0.5=PASSWORD("securepassword");
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON wordpress.* TO wordpress@10.0.0.5 IDENTIFIED BY "securepassword";
Query OK, 0 rows affected (0.00 sec)

mysql> SET PASSWORD FOR wordpress@10.0.0.6=PASSWORD("securepassword");
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON wordpress.* TO wordpress@10.0.0.6 IDENTIFIED BY "securepassword";
Query OK, 0 rows affected (0.00 sec)
```


## Installing Wordpress

So we’ve got ourselves a working web server. Now we install Wordpress. Head to the domain name / IP you set up in Nginx and install Wordpress. You’ll need to make sure that you use the same MySQL user that you allowed access to both HostA and HostB on the MySQL Server. You also need to make sure you put the MySQL server's external IP address when you add the user for the HostB box so that HostB can reach it.

If you get an error such as “Error establishing a database connection”, you’ll need to make sure the MySQL Server is listening on the external IP address.  Just edit the /etc/mysql/my.cnf and comment out the following lines:


``` bash
bind-address = 127.0.0.1
skip-networking
```


If the skip-networking line doesn’t exist, add it and comment it out. Once you’ve done this restart MySQL to fix the error.


``` bash
root@HostA:~$ sudo service mysql restart
```


If you get the error, “Sorry, but I can’t write the wp-config.php file.” -- this is a permissions issue, fixable as follows:


``` bash
root@HostA:~$ sudo usermod -a -G wordpress www-data
root@HostB:~$ sudo usermod -a -G wordpress www-data
```



``` bash
root@HostA:~$ sudo chgrp -R wordpress /var/www
root@HostA:~$ sudo chmod -R g+w /var/www
```


Alternatively, you could run lsyncd as the www-data user and just do it that way.  But it’s better to run lsyncd as its own user, rather than give the web user more permissions.

Afterwards, refresh the page.  If you get the issue again, just make the file yourself with the Wordpress user.

Once we’ve done the above, all we need to do is go to HostA and update the config to contain this:


``` bash
upstream wploadbalance{
        least_conn;
        server hosta:90;
        server hostb;
}

server {
       listen 80;
       server_name www.example.com;
       location / {
           proxy_pass http://wploadbalance;
    }
}
```


Lastly, change the listen port on the other server instance to be on port 90, and port our DNS to HostA.  Voila - you now have a load balanced nginx setup!

## Conclusion

Load balancing gives us the ability to serve more visitors at once as we “balance” the load between two different servers. It’s not so much about improving the speed of your app, but rather, it prevents your server from slowing down when there is too much traffic hitting your server at once. Load balancing can help if you are receiving increasing amounts of traffic or expect to receive a sudden spike in traffic. Like when one of your faithful users promotes you on Reddit or Hacker News. If that ever happens and you’re stuck trying to load balance your app, feel free to reach out to me through my profile. I’d be happy to lend a hacking hand!

