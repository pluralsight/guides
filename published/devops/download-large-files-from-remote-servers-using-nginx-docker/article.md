There are times when we have to download large files such as database backups and huge log files from our servers. We typically use SSH (Secure Shell) or SCP (Secure Copy) in these scenarios. But if the server is in a remote location, these protocols become extremely slow in comparison to downloading in HTTP.


So why not use HTTP for these purposes? The solution is pretty simple: just use a server like [Nginx](https://en.wikipedia.org/wiki/Nginx) and configure it to serve the files you need. Then simply download the file using [aria2](https://aria2.github.io/) or some other download accelerator from your local machine. However, these situations don’t necessarily arise on a regular basis, so our solution would need to be temporary  —  something that we could easily remove after use.

[Docker](https://www.docker.com/) to the rescue! 

_Side note: if you're new to using docker, be sure to check out our [Beginner's Guide to docker](http://tutorials.pluralsight.com/devops/docker-a-beginner-guide) first._

We’ll first build an Nginx image (check out this [discussion about images and containers](http://stackoverflow.com/questions/23735149/docker-image-vs-container) for more) with our custom configuration file and run it. That way, when we need to download something, we’ll simply start the image and stop it once our download is complete. 

As a docker image, this download process also becomes portable; you can use this on any of your servers without touching the existing system.

### Prerequisites

The solution requires you to have the following packages (preferrably the latest versions) installed on your server:

* Docker — [installation guide](https://docs.docker.com/engine/installation/)
* Docker-compose — [installation guide](https://docs.docker.com/compose/install/)

### Nginx configuration file

> It is strongly recommended to use SSL (Secure Socket Layer) for encryption if you use this guide on your production servers. Here's an [article](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04) that might be helpful.

```
server {
  listen 80;
  server_name localhost;
  # serve the static files on port 80
  location /downloads/ {
    alias /files/;
  }
}
```

The above configuration tells Nginx to listen on port 80 and serve our files when a request is made on **/downloads/** url.

### Docker-compose file

The next step would be to create the `docker-compose.yml` file. This file tells docker how to run a specific container.



> It must be noted that the docker-compose.yml file below assumes that you have created the directories `conf` and `files` where `docker-compose.yml` resides. Moreover it also assumes that the Nginx configuration file is inside the `conf` directory and the files you want to download are in the `files` directory.

```
nginx:
  image: nginx:latest
  volumes:
    - "./files:/files"
    - "./conf:/etc/nginx/conf.d"
  ports:
    - "8080:80"
```

The above file tells docker to run a container using the `nginx:latest` image, mount the directories `files` and `conf` from the host machine, and expose port 80 to host machine’s port 8080.

### Controlling the container

To start the container for the first time:
```
docker-compose up -d
```

To start the container repeatedly:
```
docker-compose start
```

To stop the container:
```
docker-compose stop
```

To restart the container:
```
docker-compose restart
```

To stop the container and remove the images from your host machine:
```
docker-compose down
```

If you find out that the `down` command is not working, you probably lack the latest `docker-compose.yml` and need to upgrade. 

After you’ve used the `down` command, the next time you want to start the container, you’ll have to use the `up` command to recreate it  —  in other words, you will have to boot up the container as if starting for the first time.

### Downloading your files

Once you’ve started your container, you can start downloading your files. At first you’ll need to put your files in the appropriate location,

> I’m assuming you’ve kept the files in the `~/downloads` directory. For this step, you need to download a file called `db.tar.gz`

```
cp db.tar.gz ~/downloader/files/
```

Then you can simply download your file from the URL below:
```
http://your_server_ip:8080/downloads/db.tar.gz
```

I hope this simple, portable solution will meet your needs, especially when it comes to occasionally downloading large content from servers. Let me know your thoughts and feedback in the comments section below! :)