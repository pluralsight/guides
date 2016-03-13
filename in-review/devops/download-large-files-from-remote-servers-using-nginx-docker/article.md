There are times when we have to download files like database backups or huge log files from our servers and usually we use SSH or SCP for this. But if the server is in a remote location than these protocols starts having hiccups as they are extremely slow compared to HTTP.
So I figured why not use HTTP for these purposes. The solution is pretty simple just use a server like Nginx and configure it to serve the files you need. Then simply download the file using aria2 or some other download accelerator from your local machine. But however, these situations usually don’t arise on a regular basis, so it has to be something temporary which can be removed as soon as we are done.

This is when [docker](https://www.docker.com/) comes to rescue. We’ll first build an Nginx image with our custom configuration file and then run it. So when we need to download something we’ll just start the image and then stop it when we are done. Being a docker image the solution also becomes portable, which means you can use this on any of your servers without much of a hassle or touching the existing system.

### Prerequisites

The solution requires you to have the following packages (preferrably the latest versions) installed on your server,

* Docker — [installation guide](https://docs.docker.com/engine/installation/)
* Docker-compose — [installation guide](https://docs.docker.com/compose/install/)

### Nginx configuration file

```
server {
  listen 80;
  server_name localhost;
  # serve the static files
  location /downloads/ {
    alias /files/;
  }
}
```

The above configuration tells Nginx to listen on port 80 and serve our files when a request is made on **/downloads/** url.

### Docker-compose file

The next step would be to create the docker-compose.yml file. This file tells docker how to run a specific container.



> It must be noted that the below docker-compose.yml file assumes that you have created the directories conf and files where docker-compose.yml resides. Moreover it also assumes that the nginx configuration file is inside the conf directory and the files you want to download are in the files directory.

```
nginx:
  image: nginx:latest
  volumes:
    - "./files:/files"
    - "./conf:/etc/nginx/conf.d"
  ports:
    - "8080:80"
```

The above file tells docker to run a container using the nginx:latest image, mount the directories files and conf from the host machine and finally expose port 80 to host machine’s port 8080.

### Controlling the container

To start the container for the first time,
```
docker-compose up -d
```

To start the container on a regular basis,
```
docker-compose start
```

To stop the container,
```
docker-compose stop
```

To restart the container,
```
docker-compose restart
```

To stop the container and remove the images from your host machine,
```
docker-compose down
```

If you find out that the down command is not working this probably means that you don’t have the latest version of docker-compose and need to upgrade. Moreover once you’ve used the down command the next time you want to start the container you’ll have to use the up command to recreate it.

### Downloading your files

Once you’ve started your container you can start downloading your files. At first you’ll need to put your files in the appropriate location,

> I’m assuming you’ve kept the files in the ~/downloads directory and you need to download a file called db.tar.gz

```
cp db.tar.gz ~/downloader/files/
```

Then you can simply download your file from the below URL,
```
http://your_server_ip:8080/downloads/db.tar.gz
```

I hope this simple solution will come of some use to you and comments are more than welcome :)