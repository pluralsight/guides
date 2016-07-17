Gone are the days when a developer had to make a mess of their base OS by installing loads of software, because Docker has come to save the day. In this article I'm going to talk about using MySql docker container instead of installng the database server in your system. 

##### Some advantages of using Docker in this scenario

 * Does not start or stop with your system by default. You run it only when you need, it frees up your system's memory and also improves your system start up time.
 * Does not make any changes to your base OS.
 * You can upgrade or downgrade MySql any time you want to without worrying whether the version is in your OS package repository or not.
 * Portable. Easily transferrable from one system to another.
 * Cross-platform. Run the same container on OSX or Linux or Windows
 
I can't think of any more at the moment, although the people at Docker has put up some amazing documentation on their site. Feel free to have a look if you want to know [more](https://docs.docker.com/).


> I'll be using OSX for this guide. These commands should be usable on Linux as well. For Windows users I'd try my best to provide the appropriate links but you might need to make some tweaks before you can use this.

### Prerequisites
* **Docker** - Please follow the [installation guide](https://docs.docker.com/engine/installation/) for assistance or you can get it by using the direct links below
    * [Mac](https://download.docker.com/mac/beta/Docker.dmg)
    * [Linux](https://docs.docker.com/engine/installation/linux/)
    * [Windows](https://download.docker.com/win/beta/InstallDocker.msi)
* **docker-compose** - You can install it by following the instructions [here](https://docs.docker.com/compose/install/).
* **make** - I'd be using Makefile to control the docker container. If you don't have `make` installed please follow the resepctive link
    * [Mac](http://stackoverflow.com/a/11494872/2894655)
    * [Linux](http://www.cyberciti.biz/faq/debian-linux-install-gnu-gcc-compiler/)
    * [Windows](http://gnuwin32.sourceforge.net/packages/make.htm)

### Set it all up

I usually prefer to keep all my work related files in the location `~/Workspace`. So I'd be using this location through out the guide. If you prefer to keep them somewhere else please make the appropriate changes.

I've also created a [github repo](https://github.com/redmoses/mysql-docker) to help you along, if you want to skip the steps.

The first step would be to create the directory
```bash
mkdir ~/Workspace/mysql-docker
```

The directory structure would look like below at the end of this guide -
```bash
.
├── Makefile
├── config
│   ├── custom.cnf
│   └── mysql.sh
└── docker-compose.yml
```

##### `docker-compose.yml`

Inside the `mysql-docker` directory create `docker-compose.yml` file  

```
mysql:
  image: mysql:latest
  container_name: mysql-db01
  ports:
    - "3306:3306"
  environment:
    - MYSQL_ROOT_PASSWORD=root1234
    - MYSQL_DATABASE=testdb
    - MYSQL_USER=testuser
    - MYSQL_PASSWORD=testpass
  volumes:
    - ./config/mysql.sh:/root/mysql.sh
    - ./config/custom.cnf:/etc/mysql/conf.d/custom.cnf
    - ./data:/var/lib/mysql
```

For detailed information regarding the `docker-compose` file please see [here](https://docs.docker.com/compose/compose-file/).

* `mysql` The name of the docker-compose service
* `image` The image to start the container from. I'm using the `latest` tag here which means it will get the latest mysql version available. If you want to work with some specific version you can check out the list of available tags [here](https://hub.docker.com/r/library/mysql/tags/).
* `container_name` A custom container name
* `environment` Environment variables for the container
* `volumes` Volumes to mount when running the container

The `environment` variables
* `MYSQL_ROOT_PASSWORD` The root password of your database server
* `MYSQL_DATABASE` The name of the default database to start the container with. You can ignore this if you don't want the server to start with an already created database.
* `MYSQL_USER` If you've created a default database then the user defined here will have all permissions over that database.
* `MYSQL_PASSWORD` The default user's password

More information regarding the mysql docker image can be found [here](https://hub.docker.com/_/mysql/)

##### `Makefile`

Inside the `mysql-docker` directory create a file called `Makefile` 
```
up:
	docker-compose up -d

down:
	docker-compose down

start:
	docker-compose start

stop:
	docker-compose stop

restart:
	docker-compose restart

status:
	docker-compose ps

logs:
	docker-compose logs

shell:
	docker exec -ti mysql-db01 bash -e /root/mysql.sh
```

If you closely look into the contents of `Makefile` then you'd see that all the commands are just running different `docker-compose` commands except the `shell` command. This is why the `Makefile` exists because of the inability of `docker-compose` to execute commands on a running container.

* `up` command will download the mysql docker image if it's not already there, then it will start it up according to the `docker-compose.yml` configuration. The `-d` argument ensures that the container will run in daemon mode (in the background)
* `down` command will stop the container and delete it
* `start`, `stop`, and `restart` commands will control the container accordingly
* `status` command will show the current status of the container
* `logs` command will show the current logs of the container
* `shell` command will log into the mysql shell with root credentials

##### `mysql.sh`
Inside the `mysql-docker` directory create a directory called `config`. Now create a file called `mysql.sh` inside the `config` directory you just created

```bash
#!/bin/sh
mysql -uroot -p$MYSQL_ROOT_PASSWORD
```
As we are defining the mysql root password in the environment variable `MYSQL_ROOT_PASSWORD` the script uses this to log into the mysql shell using root credentials. This script is mounted on `/root/mysql.sh` and is executed by the `Makefile` command `shell`.

##### Create an alias
This is just to simplify things for us. I use `zsh` as my shell so I'd add the following line in `~/.zshrc` file, but if you use `bash` you should be adding to `~/.bashrc`
```bash
alias mysql='cd ~/Workspace/mysql-docker/ && make '
```

Once you've created the alias load it in your environment
```bash
$ source ~/.zshrc
```
---
**Initialize MySql**

Now we are all set up. So lets start using the mysql docker. The first time you run the docker you have to run the initialize command. This command will download mysql docker image if it's not already there. Then it will start it up according to the `docker-compose.yml` configuration.
```
$ mysql up
```

Once you've initialized the docker you can open up your terminal and use the following commands accordingly.

**Start MySql**

Start up the mysql container
```
$ mysql start
```
**Stop MySql**

Stop the mysql container
```
$ mysql stop
```
**Restart MySql**

Restart the mysql container
```
$ mysql restart
```
**MySql logs**

Show the logs of the running container
```
$ mysql logs
```
**MySql shell**

To connect to mysql shell
```
$ mysql shell
```

**Data location**

The data will be stored in the location `~/Workspace/mysql-docker/data`

**MySql configurations**

If you want to have custom mysql settings you can edit the file `config/custom.cnf` according to your needs.


**Refreshing the database**

If you want to start fresh and delete all old data you can do the following -
```bash
$ mysql down && sudo rm -rf ~/Workspace/mysql-docker/data && mysql up
```

---

Hope this guide helps you in making your coidng days easier.
