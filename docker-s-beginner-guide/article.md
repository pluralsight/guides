Docker is a hot product on the market right now, i think most of you heard of it, if not let me explain a little what it means.

Docker allows you to package an application with all of its dependencies into a standardized unit for software development. That unit is called container. 

Docker it runs only above **Linux** but you can use it with **Mac OS** or **Windows** using **Kitematic**. Kitematic is a Docker tool that creates for you **VirtualBox Machines** in order to let you run Docker on them. Basically for development part is ok to run Docker on Mac/Windows but for production is desirable to run Docker only on Linux machines.

Let me explain some terms used in Docker:
- **dockerfile** - is a file that describes your steps in order to create a Docker image. Is like a recipe where you tell all ingredients and steps in order to obtain your dish.
- **image** - images on Docker are like the snapshot of a virtual machine, but way more lightweight, they are the building-block of the containers.
- **container** - From images you can create containers, this is the equivalent of creating a VM from a snapshot, but again, way more lightweight. Containers are the ones that run applications.

These terms are the most used terms in Docker's world. I will cover only these terms for the beginner's guide.

I will cover the creating and packaging of a **Java** application that runs on **Tomcat Server**.

**_

### 0. Installation
_**


You can install Kitematic for Mac OS and Windows from here [https://kitematic.com/](https://kitematic.com/) and for Linux you can type this command 
```
curl -sSL https://get.docker.com/ | sh
```
_If you are using Mac or Windows i will suggest you to not use Kitematic interface in the beginning, it will be ok if you start to create and manipulate everything from command line in this way you will learn more._



**_

### 1. First Step
_**

You start by creating your Dockerfile that describes the steps in order to create the base image, here is an example (comments on each line that describes the code):
```

# All Dockerfiles needs to start from a base linux image.
# Docker has a hub (http://hub.docker.com) where you can see all images submitted by users and 
# you can submit your own images. Something like github for Docker images
# we will start from java:8-jre that is an ubuntu machine with java 8 installed on it
# the keyword here is FROM
FROM java:8-jre

# The KEYWORD ENV it let us specify some linux environment variables
# Here we will set catalina_home (the home of the tomcat server)
ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH


# One key command is RUN that let you run linux commands inside the image.
# Here we navigate into /usr/local then download tomcat from the server
# then uncompress it and then rename it to /tomcat
# the final path to tomcat will be /usr/local/tomcat
RUN 
     cd /usr/local/ && 
     wget http://mirrors.m247.ro/apache/tomcat/tomcat-8/v8.0.32/bin/apache-tomcat-8.0.32.tar.gz && 
     tar xzf apache-tomcat-8.0.32.tar.gz &&
     mv apache-tomcat-8.0.32/ tomcat/


# the EXPOSE command it will tell to your future container to expose 8080 port to outside
# 8080 is the default port for tomcat
EXPOSE 8080

# CMD it sets the first command that will be run when you will start containers from the resulting image, here we will tell that we will start the tomcat when the container is started
CMD ["/usr/local/tomcat/catalina.sh", "run"]
```

**

### 2. Step Two
**
After you create this Dockerfile you need to create your Image.
You need to navigate into your directory where the Dockerfile was saved (navigate through command line) and type
```
docker build -t tomcat .
```

What this mean: **docker build** - _Build a new image from the source code at PATH_, in our case the PATH is '.' (current folder). -t means  Name and optionally a tag the image (format name:tag), in our case just the name tomcat.

After all steps that will be executed you will check if everything was ok by typing
docker images and you will see your image there: tomcat.

**

### 3. Step three
**
You can now start a container from an images is by running this
```
docker run -p 8080:8080 --name=tomcat tomcat
```

Let me explain, **docker run** - _Run a command in a new container_. -p is very important  because, usually applications that use some ports in order to run, like tomcat 8080, mysql 3306, apache 80, in a docker container those ports will be restricted only in that container. What this means? it means that if you start a container with mysql (port 3306) and you want to access that mysql instance from a Java application you will not be able to access it because mysql port is restricted to be open just in the Docker container that you started. 
But we have a solution here by using -p parameter - _Publish a container's port(s) to the host_.By using **port1:port2** you want to say that: i want  **port1** to match with **port2**. After that you can use **port2** in order to access the service that is opened on **port1** inside the container.

**

### 4. Step four
** 
After you run a container you can check that by typing 
```
docker ps
```
This command will show to you all running containers, if this will not show something than it means your container creation failed. To check that type
```
docker ps -a
```
That will display all containers.

**

### 5. Extras
**

After you started a container you can stop it by using **docker stop [name]**, in the future you can start again the container by typing **docker start [name]**.
Another important command is **docker exec** that let's you run a command inside your docker **running** container. Be aware just in running containers not in the stopped ones.

For a complete list of commands you can go to [https://docs.docker.com/engine/reference/commandline/cli/](https://docs.docker.com/engine/reference/commandline/cli/). I will recommend you to read all of those commands.

Hope that this will help you. For any question just write me on twiiter [@grvmariobyn](https://twitter.com/grvmariobyn)
Thank you. Marius.