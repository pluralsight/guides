In this blog post, the aim is to introduce you to Docker, one of the most exciting and powerful open­source projects that has sprung up in the recent years. In a nutshell, Docker offers you the tools to package everything that forms an application, allowing you to effortlessly deploy the application across systems and machines (both virtual and physical). It allows you to setup once and then deploy anywhere. Docker makes it extremely easy for Consultants, Technical Account Managers, Solutions Architects and others to quickly test a piece of software. It's much quicker to use than a virtual machine, and it requires fewer resources. The average command takes under a second to complete!

![Docker Logo](https://d3oypxn00j2a10.cloudfront.net/0.14.4/img/nav/docker-logo-loggedout.png)

In the background, the Docker project uses mostly existing Linux functionality, such as LXC ([Linux containers](https://linuxcontainers.org/lxc/introduction/)), device­mapper and "aufs" ([a union file system](http://en.wikipedia.org/aufs)).

So why is every one raving about it? The short answer is that it is so light weight. Furthermore, it allows you to port applications across systems and hardware with ease, all while containing those applications and running them in their own secure sandboxed environments

### Ease of use

The whole process of porting applications is based on Docker containers which as just simple
directories that can be copied across to different systems or machines.

### Lightweight

Docker reuses mostly existing kernel functionality and Linux libraries, simply glueing the existing tools. It makes use of LXC technology which provides for virtualisation without the overhead of other virtualisation technologies.

### Security

Using LXC, it's is able to contain applications limiting resources, not allowing access beyond their own file­system as well as process isolation.

### Portability

By using containers, applications are able to work anywhere they are deployed to.

### Incremental Development

Because it is filesystem based it is easy to take snapshots, perform rollbacks similar to using a version control system

## How Docker works

The main parts of the deployment system are:

1. **Daemon**:­ This manages the LXC containers on a host
2. **Client**:­ This is used to interact with the Docker daemon
3. **Index**:­ This is a repository of Docker images

The above tools are used to create the applications which consist of:

1. **Containers**: ­These are directories containing everything needed to run an application.
2. **Files:** Scripts for automating the building of Docker applications.
3. **Images**: ­Snapshots of Docker applications or Base Operating Systems.

## Simple Use Cases

* I need to see the man page from a specific version of Ubuntu
* I need to quickly verify the command line options of a program
* I need to test the functionality of a specific version of software
* I need a scratch pad that is NOT my system
* I need a single daemon running, and I don't care what distribution of Linux it runs on

A lot more can be said about this program. To get the full low down, refer to the Docker documentation. To get started with using it on your system, you need to have kernel version 3.12.27 or later, which will be 'Docker'­enabled.

The system is supported on the following different distributions:

### Ubuntu:
* Ubuntu Trusty 14.04 (LTS) (64­bit)
* Ubuntu Precise 12.04 (LTS) (64­bit)

For Ubuntu 12.04 follow instructions [here](http://docs.docker.com/installation/ubuntulinux/#ubuntu­precise­1204­lts­64­bit).

#### Installation

``` bash
$ apt­-get update
$ apt­-get install docker.io lxc
$ ln ­sf /usr/bin/docker.io /usr/local/bin/docker
$ sed ­i '$acomplete ­F _docker docker' /etc/bash_completion.d/docker.io
```

### Centos

* CentOS­7 (64­bit)

#### Installation

``` bash
$ yum install docker
$ yum install docker­io lxc
$ systemctl start docker
$ systemctl enable docker
```


Once you install docker, start your first container, like this.

``` bash
$ docker run [image name] [command to run]
```
*note that this is usage syntax, not a command

``` bash
$ docker run ubuntu /bin/bash
```
Next up, we will see how to use its command line interface in detail. Stay tuned for the rest of the series!
