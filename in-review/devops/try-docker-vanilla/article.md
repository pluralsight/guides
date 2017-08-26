>Docker experiment to build hello world image from scratch. Instead of using shell script, C based application is used to make it comparable to custom application package/distribution.


Creating minimal Docker image
-----------------------------


#### What you need :

* Docker installed on linux variant (tested on Ubuntu 14LTS)
* gcc for making minimal application


#### Let's create Helloworld application to package it to Docker:

file: helloWorld.c

```c
#include<stdio.h>

void main() {

  printf("Hello from Docker!!\n");

}
```


We can compile with static linking to avoid shared library dependencies.

```bash
gcc -static helloWorld.c -o helloWorld`
```

Verify that there are no dependencies with ldd
```bash
ldd "helloWorld"
>     not a dynamic executable
```


Create a Dockerfile as follows (configuration for Docker image that
we are about to build).

```
FROM scratch
ADD ./helloWorld /helloWorld
CMD ["/helloWorld"]
```

It tells Docker to seed the image from the `scratch` image, which is completely empty, and add the helloWorld executable to it as target path /helloWorld and the startup command is to execute /helloWorld.


Now build the image with

```bash
$ docker build -t "helloWorld" .
Sending build context to Docker daemon 880.6 kB
Sending build context to Docker daemon
Step 0 : FROM empty
 ---> 62f67e5fbc6a
Step 1 : ADD ./a /a
 ---> 1236c4ce0129
Step 2 : CMD ["/a"]
 ---> f6982bf61e13
Successfully built f6982bf61e13
```

Now you can list images with
```bash
$ docker images
REPOSITORY   TAG    IMAGE ID     CREATED           VIRTUAL SIZE
first_image  latest f6982bf61e13 54 seconds ago      877.4 kB
```


To Run the docker image as a container:  
```bash
$ docker run helloWorld
```

You should see output like:
```
Hello from docker
```

> Recommended to checkout https://github.com/docker/docker.git and browse the source in your
favorite ide


