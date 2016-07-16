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
* `Docker` Please follow the [installation guide](https://docs.docker.com/engine/installation/) for assistance or you can get it by using the direct links below
    * [Mac](https://download.docker.com/mac/beta/Docker.dmg)
    * [Linux](https://docs.docker.com/engine/installation/linux/)
    * [Windows](https://download.docker.com/win/beta/InstallDocker.msi)

* `docker-compose` You can install it by following the instructions [here](https://docs.docker.com/compose/install/).

* `make` I'd be using Makefile to control the docker container
    * [Mac](http://stackoverflow.com/a/11494872/2894655)
    * [Linux](http://www.cyberciti.biz/faq/debian-linux-install-gnu-gcc-compiler/)
    * [Windows](http://gnuwin32.sourceforge.net/packages/make.htm)

### Set it all up

I usually prefer to keep all my work related files in the location `~/Workspace`. So I'd be using this location through out the guide. If you prefer to keep them somewhere else please make the appropriate changes.

##### Create the directory
```bash
mkdir ~/Workspace/mysql-docker
```

The directory structure would look like below at the end of this guide -
```bash
.
├── Makefile
├── config
│   └── mysql.sh
└── docker-compose.yml
```

##### `docker-compose.yml`

Inside the `mysql-docker` directory create `docker-compose.yml` file with the below contents 

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
    - ./data:/var/lib/mysql
```
**Explanation**

For detailed information regarding the `docker-compose` file please see [here](https://docs.docker.com/compose/compose-file/).

* `mysql` The name of the docker-compose service

* `image` The image to start the container from

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

Inside the `mysql-docker` directory create a file called `Makefile` with the below contents 
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

**Explanation**

If you closely look into the contents of `Makefile` then you'd see that all the commands are just running different `docker-compose` commands except the `shell` command. This is why the `Makefile` exists because of the inability of `docker-compose` to execute commands on a running container.

* The `up` command will download the mysql docker image if it's not already there, then it will start it up according to the `docker-compose.yml` configuration. The `-d` argument ensures that the container will run in daemon mode (in the background)

* The `down` command will stop the container and delete it

* The `start`, `stop`, and `restart` commands will control the container accordingly

* The `status` command will show the current status of the container

* The `logs` command will show the current logs of the container

* The `shell` command will log into the mysql shell with root credentials
