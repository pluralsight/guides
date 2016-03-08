>Docker experiment to build hello world image from scratch. Instead of using shell script, C based
application is used to make it comparable to custom application package/distribution.




Creating minimal docker image
-----------------------------



what you need :

-   docker installed on linux variant (tested on Ubuntu 14LTS)

-   gcc for making minimal application



docker can import archives such as tar, tar.gz, .bzip etc. you can package all
necessary files into an archive and import it into docker.



    tar cv --files-from /dev/null | sudo docker import - empty

This will create empty image in docker repository.



Now lets create Helloworld application to package it to docker.

file: a.c

    #include<stdio.h>

    void main() {

      printf("Hello from docker!!\n");

    }



We can compile with static linking to avoid shared library dependencies.

    gcc -static a.c -o a`

Verify that there are not dependencies with ldd

    ldd "a"
    >     not a dynamic executable



Create a Dockerfile with below entries (configuration for docker container that
we are about to build).

    FROM empty
    ADD ./a /a
    CMD ["/a"]



It says docker that seed the image from empty image that we have created and add
file a

to it as target path /a and startup command is to execute /a



Now build the image with

    $ sudo docker build -t "first_image" ./
    Sending build context to Docker daemon 880.6 kB
    Sending build context to Docker daemon
    Step 0 : FROM empty
     ---> 62f67e5fbc6a
    Step 1 : ADD ./a /a
     ---> 1236c4ce0129
    Step 2 : CMD ["/a"]
     ---> f6982bf61e13
    Successfully built f6982bf61e13


Now you can list images with

    $sudo docker images
    REPOSITORY   TAG    IMAGE ID     CREATED           VIRTUAL SIZE
    first_image  latest 9ba228119b81 54 seconds ago      877.4 kB



To Run the docker instance in background mode :

    $ sudo docker run first_image



You should see output like

    Hello from docker


> Recommended to checkout https://github.com/docker/docker.git and browse the source in your
favorite ide


