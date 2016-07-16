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

##### Docker 
To follow this guide you must have Docker already installed in your system. Please follow the [installation guide](https://docs.docker.com/engine/installation/) for assistance or you can get it by using the direct links below

* [Mac](https://download.docker.com/mac/beta/Docker.dmg)
* [Linux](https://docs.docker.com/engine/installation/linux/)
* [Windows](https://download.docker.com/win/beta/InstallDocker.msi)

##### docker-compose
One more requirement is to have ```docker-compose``` installed as well. You can install it by following the instructions [here](https://docs.docker.com/compose/install/).

##### make
I'd be using Makefile to control the docker container and everything else and tt just makes life easier.
* [Mac](http://stackoverflow.com/a/11494872/2894655)
* [Linux](http://www.cyberciti.biz/faq/debian-linux-install-gnu-gcc-compiler/)
* [Windows](http://gnuwin32.sourceforge.net/packages/make.htm)


### Directory Structure

The directory structure should look like below at the end of this guide -
```bash
.
├── Makefile
├── README.md
├── config
│   └── login.sh
└── docker-compose.yml
```

#### ```docker-compose.yml```