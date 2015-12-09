Lets play around the docker installation we did earlier. Docker has a straightforward CLI (command line interface) that allows you to do almost everything you could want to a container. All of these commands use the image id (ex. be29975e0098), the image name (ex. coolsvap/test_webapp) and the container id (ex. 72d468f455ea) interchangably depending on the operation you are trying to do. This is confusing at first, so pay special attention to what you're using.

## Launching a Container

Launching a container is simple as docker run + the image name you would like to run + the command to run within the container. If the image doesn't exist on your local machine, docker will attempt to fetch it from the public image registry. Later we'll explore how to use docker with a private registry. It's important to note that containers are designed to stop once the command executed within them has exited. For example, if you ran /bin/echo hello world as your command, the container will start, print "hello world" and then stop:

``` bash
$ docker run ubuntu /bin/echo hello world
```

Let's launch an Ubuntu container and install Apache inside of it using the bash prompt:

``` bash
$ docker run -t -i ubuntu /bin/bash
```

The `-t` and `-i` flags allocate a pseudo-tty and keep stdin open even if not attached. This will allow you to use the container like a traditional VM as long as the bash prompt is running.

Install Apache with

``` bash
$ apt-get update && apt-get install apache2
```

You're probably wondering what address you can connect to in order to test that Apache was correctly installed...we'll get to that after we commit the container.

## Committing a Container

After that completes, we need to commit these changes to our container with the container ID and the image name. To find the container ID, open another shell (so the container is still running) and read the ID using

``` bash
$ docker ps
```

The image name is in the format of username/name. We're going to use coolsvap as our username in this example but you should sign up for a docker.com user account and use that instead. It's important to note that you can commit using any username and image name locally, but to push an image to the public registry, the username must be a valid docker.com user account. Commit the container with the container ID, your username, and the name apache:

``` bash
$ docker commit 72d468f455ea coolsvap/apache
```

The overlay filesystem works similar to git: our image now builds off of the ubuntu base and adds another layer with Apache on top. These layers get cached separately so that you won't have to pull down the ubuntu base more than once.

## Keeping the Apache Container Running

Now we have our Ubuntu container with Apache running in one shell and an image of that container sitting on disk. Let's launch a new container based on that image but set it up to keep running indefinitely. The basic syntax looks like this, but we need to configure a few additional options that we'll fill in as we go:

``` bash
$ docker run [options] [image] [process]
```

The first step is to tell docker that we want to run our coolsvap/apache image:

``` bash
$ docker run [options] coolsvap/apache [process]
```

### Run Container Detached

When running docker containers manually, the most important option is to run the container in detached mode with the `-d` flag. This will output the container ID to show that the command was successful, but nothing else. At any time you can run `docker ps` in the other shell to view a list of the running containers. Our command now looks like:

``` bash
$ docker run -d coolsvap/apache [process]
```

After you are comfortable with the mechanics of running containers by hand, it's recommended to use systemd units and/or fleet to run your containers on a cluster of docker host machines. Do not run containers with detached mode inside of systemd unit files. Detached mode prevents your init system, in our case systemd, from monitoring the process that owns the container because detached mode forks it into the background. To prevent this issue, just omit the `-d` flag if you aren't running something manually.

### Run Apache in Foreground

We need to run the apache process in the foreground, since our container will stop when the process specified in the docker run command stops. We can do this with a flag `-D` when starting the apache2 process:

``` bash
$ /usr/sbin/apache2ctl -D FOREGROUND
```

Let's add that to our command:

``` bash
$ docker run -d coolsvap/apache /usr/sbin/apache2ctl -D FOREGROUND
```

#### Network Access to Port 80

The default apache install will be running on port 80\. To give our container access to traffic over port 80, we use the `-p` flag and specify the port on the host that maps to the port inside the container. In our case we want 80 for each, so we include `-p 80:80` in our command:

``` bash
$ docker run -d -p 80:80 coolsvap/apache /usr/sbin/apache2ctl -D FOREGROUND
```

You can now run this command on your docker host to create the container. You should see the default apache webpage when you load either `localhost:80` or the IP of your docker host.