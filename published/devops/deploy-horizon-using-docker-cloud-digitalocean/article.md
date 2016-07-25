This tutorial is an end-to-end summary of how you can build and deploy a Horizon Application using GitHub, Travis-CI, Docker, Docker Compose, Docker Hub, Docker Cloud, and DigitalOcean. That's a lot of Dockers but don't worry, it's not complicated!

----

## Create a Horizon Application

For this tutorial I'm going to use an Express application with an [embedded Horizon server](http://horizon.io/docs/embed/). Embedding Horizon in an Express application will give you a little more flexibility.

Initialize npm with two dependencies, Express and @horizon/server.

``` bash
mkdir horizon-with-docker
cd horizon-with-docker
npm init
npm install --save @horizon/server
npm install --save express
```

Next, create a small Express application that will contain the Horizon server. The folks over at Horizon have included [an example](http://horizon.io/docs/embed/) on how to do this in their documentation. I'm going to copy their example and make some minor changes.

Note that I'm specifying the `rdb_host` and `rdb_port` as environment variables. These options specify how the embedded Horizon server connects to RethinkDB. I've given them some default values which you can use for now. (More on RethinkDB environment variables later.)

horizon-with-docker/server.js

``` javascript
'use strict'

const express = require('express');
const horizon = require('@horizon/server');
const path = require('path');

const app = express();
app.use(express.static('dist'));
app.get('*', function(req, res) {
  res.sendFile(path.join(__dirname, 'dist/index.html'));
});

const httpServer = app.listen(8081, function(err) {
  if (err) {
    console.log(err);
  } else {
    console.log('Listening on port 8081.');
  }
});

const horizonServer = horizon(httpServer, {
   auto_create_collection: true,
   auto_create_index: true,
   project_name: 'HorizonWithDocker',
   permissions: false,
   rdb_host: process.env.RDB_HOST || 'localhost',
   rdb_port: process.env.RDB_PORT || 28015,
   auth: {
     allow_anonymous: true,
     allow_unauthenticated: true,
     token_secret: 'HorizonWithDockerIsSecret'
   }
});


```

### Create a front-end for the application. 

Note in the above code the horizon client is being served by the Horizon server. Horizon does however offer the Horizon client as a separate npm package. One could technically serve it at another location if need be. I'll copy the front-end from this [example](http://horizon.io/docs/getting-started/).

horizon-with-docker/dist/index.html

``` html

<!doctype html>
<html>
  <head>
    <meta charset="UTF-8">
    <script src="/horizon/horizon.js"></script>
    <script>
      var horizon = Horizon();
      horizon.onReady(function() {
        document.querySelector('h1').innerHTML = 'Horizon With Docker!'
      });
      horizon.connect();
    </script>
  </head>
  <body>
   <marquee><h1></h1></marquee>
  </body>
</html>

```

Putting it all together, insert an npm start script that will run the Horizon application.

horizon-with-docker/package.json

``` json 
{
  "name": "horizon-with-docker",
  "version": "1.0.0",
  "description": "A sample application to use in a tutorial on how to deploy Horizon to Docker/DigitalOcean",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "author": "Christopher Asche",
  "license": "MIT",
  "dependencies": {
    "@horizon/server": "^1.1.3",
    "express": "^4.14.0"
  }
}
```

### Test the application. 

If you have not done so already, you will have to [install](https://www.rethinkdb.com/docs/install/) and [start](https://www.rethinkdb.com/docs/start-a-server/) RethinkDB. Once RethinkDB is started, start the Horizon application using the npm start script.

``` bash
$ npm start

> horizon-with-docker@1.0.0 start /path/to/horizon-with-docker
> node server.js

Listening on port 8081.
info: Connecting to RethinkDB: localhost:28015
info: Index metadata synced.
info: Groups metadata synced.
info: Collections metadata synced.
info: Metadata synced with database, ready for traffic.
```

Then visit [http://localhost:8081](http://localhost:8081) and voila!

## Dockerize the Horizon Application

If you have not already done so, [install docker](https://docs.docker.com/engine/installation/). We can create a docker image that will contain our Horizon application. For this specific application, it's fairly straight forward.

Create a dockerfile in the base repository that installs the node dependencies, copies the application to `/user/src/app`,and calls the previously created start script.

horizon-with-docker/Dockerfile

``` Dockerfile 

FROM nodesource/trusty:6.2.0

WORKDIR /usr/src/app

COPY package.json /usr/src/app/package.json

RUN npm install
COPY . /usr/src/app

EXPOSE 8081

CMD [ "npm", "start" ]

```

Test the dockerfile by building your applications image. I've named mine `horizon-width-docker` but you're free to choose any name.

``` bash 

docker build -t horizon-with-docker .

```

Once your image has finished building, test it by running it inside a container. If you want to view the images on your host, you can always recall them via the *docker images* command.

``` bash

docker run -i -t horizon-with-docker:latest

> horizon-with-docker@1.0.0 start /usr/src/app
> node server.js

Listening on port 8081.
info: Connecting to RethinkDB: localhost:28015

```

Uh oh! Looks like the application started fine, but could not finish connecting to RethinkDB. 

Perfect!

If you're using Docker Machine like I am, your container is running inside a virtual machine. You can inspect the IP of this virtual host with the command *docker-machine ip* -- for instance, my IP is 192.168.99.100.

The default settings for Horizon are to connect to a RethinkDB at localhost:28015. The instance I started earlier is running on *localhost:28015*. So the application is not able to finish starting because it cannot connect to RethinkDB. __Let's fix this problem with a tool called Docker Compose.__

## Orchestrate with Docker Compose

[Docker Compose](https://docs.docker.com/compose/) is a tool for orchestrating multiple docker containers. Docker Compose will start our application *and* RethinkDB, each in their own docker container.

For this to work, a configuration file called `docker-compose.yml` is necessary. The file will define two services, `web` and `db`. They will correspond with the Horizon application and the RethinkDB database.

Remember the `rdb_host` and `rdb_port` environment variables from `server.js`? They can be utilized now because they're needed to tell the embedded Horizon server how to connect to RethinkB. The service name, which arbitrarily points to the host, can be used with Docker Compose.

horizon-with-docker/docker-compose.yml

``` yml

version: "2.0"
services:
 db:
   image: rethinkdb
   ports:
     - "28015:28015"
     - "8080:8080"
 web:
   build: .
   environment:
    RDB_HOST: db
    RDB_PORT: 28015
   depends_on:
     - db
   ports:
     - "8081:8081"

```

To start the services, use the `docker-compose up` command.

``` bash 

$ docker-compose up
Starting horizonwithdocker_db_1
Starting horizonwithdocker_web_1
Attaching to horizonwithdocker_db_1, horizonwithdocker_web_1
web_1  |
web_1  | > horizon-with-docker@1.0.0 start /usr/src/app
web_1  | > node server.js
web_1  |
web_1  | Listening on port 8081.
web_1  | info: Connecting to RethinkDB: db:28015
db_1   | Running rethinkdb 2.3.4~0jessie (GCC 4.9.2)...
db_1   | Running on Linux 4.4.12-boot2docker x86_64
db_1   | Loading data from directory /data/rethinkdb_data
db_1   | warn: Cache size does not leave much memory for server and query overhead (available memory: 656 MB).
db_1   | warn: Cache size is very low and may impact performance.
db_1   | Listening for intracluster connections on port 29015
db_1   | Listening for client driver connections on port 28015
db_1   | Listening for administrative HTTP connections on port 8080
db_1   | Listening on cluster addresses: 127.0.0.1, 172.19.0.2, ::1, fe80::42:acff:fe13:2%176
web_1  | info: Index metadata synced.
db_1   | Listening on driver addresses: 127.0.0.1, 172.19.0.2, ::1, fe80::42:acff:fe13:2%176
db_1   | Listening on http addresses: 127.0.0.1, 172.19.0.2, ::1, fe80::42:acff:fe13:2%176
db_1   | Server ready, "6c696e22ceab_d6t" f24ca1b0-0c28-40d8-8fb7-2b15acfed12d
web_1  | info: Index metadata synced.
web_1  | info: Index metadata synced.
web_1  | info: Index metadata synced.
web_1  | info: Collections metadata synced.
web_1  | info: Index metadata synced.
web_1  | info: Groups metadata synced.
web_1  | info: Collections metadata synced.
web_1  | info: Metadata synced with database, ready for traffic.

```

Now, with everything synced up on the web, visit http://DOCKER_MACHINE_IP:8081. You should see your Horizon app!

## Building with Travis-Ci

To build the application, I'm going to use __Travis-CI.__ 

Travis-CI integrates very nicely with GitHub and provides a nice Docker Service. Go ahead and create a repository for the source code. I'm going to use [casche/horizon-with-docker](http://github.com/casche/horizon-width-docker).

In this application, the build is going to be trivial. Travis-CI is just going to run *npm install* and report back the status to GitHub. To accomplish this, create a `.travis.yml` file:

horizon-with-docker/.travis.yml

``` yml 

language: node_js
node_js:
- '5'

```

Commit the changes and push them to GitHub. Note: you may have noticed a couple directories that were generated in our project, `node_modules` and `rethinkdb_data`. These should be added to the `.gitignore` because they're not needed under version control.

Now head over to [travis-ci.org](http://travis-ci.org) and add your repository. You can log into Travis-CI with your GitHub credentials. Click the '+' button to the left to add add your repository and enable your build. If you don't see your repository, try 'syncing' your account. This will query GitHub and get a new snapshot of your repositories.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/9829abce-64c5-46e3-b9f0-7ef8d2cb694c.png)


That's it! Now when changes are pushed to GitHub, Travis-CI will automatically build your application and report back the status on the commit. Go ahead and try it to see changes in real-time.

## Pushing to Docker Hub

The next objective is to modify the Travis-CI build so that it creates and pushes a docker image containing the application to Docker Hub. This allows Docker Hub to pull the image and deploy it to a DigitalOcean droplet. If you don't already have a Docker Cloud account, [sign up for one on Docker Cloud](https://cloud.docker.com).

Edit the *.travis.yml* file to build and push the docker image after a successful build. To do this, Travis-Ci needs to login to your Docker Cloud account. Create [encrypted environment variables](https://docs.travis-ci.com/user/environment-variables/) to store the login credentials in the *.travis.yml*.

``` bash 
gem install travis
travis encrypt DOCKER_EMAIL=<docker-cloud-email> --add env.global
travis encrypt DOCKER_USER=<docker-cloud-user> --add env.global
travis encrypt DOCKER_PASS=<-docker-cloud-pass> --add env.global
```

Now, add an *after_success* clause that will:

 1. Log-in to Docker Cloud
 2. Build the image
 3. Tag the image
 4. Push it to Docker Hub.

To enable this, add the [docker service](https://docs.travis-ci.com/user/docker/) to the *.travis.yml*.

> Don't forget to change the DOCKER_IMAGE_NAME environment variable to your name/horizon-with-docker!

``` yml 
language: node_js
services:
- docker
env:
  global:
  - DOCKER_IMAGE_NAME=casche/horizon-with-docker
  - COMMIT=${TRAVIS_COMMIT::8}
  - secure: mYeRDOjcU+0vz+MxG094IsFusddmHKtUtPt6CM6NZmtvzY3OynTyNE4YYZwUPqe/WFiOuZ79MGUehjJeStE9+voTaVY3hb+iMFStmimErGireRoysIYHfFc1HKoqSbXTM/NZ935Y3PQk6MGIrOmgEbw4A0WZrq8lX55hsV0WfFsel8B08IBf2RcHnfM1QYXtdSpZtUR2B42Y9lo96C30krF3ywMPCfJZK2W8o1vG5sHytl+Vh5QZTDH7heCO064qxRaq218vw+YPXFnecZlp8kILTaWDdhPOGACMBwQZZOAKNi6RWyvxYpHPGCLYTCtwJ+dCoYWYSBonTH4ASgabVlnwzcVz1WDEmX6KBfLLnD0lDkI6uavxl5fQ85okXGUJ8jYnhw45t+4lMGW0ZzPWO7dUYSkj39GrT4+sOZp50DtQKBvaPxBzQ2P9N/KDNyUXu5Xs8RD7z4HA1FvTpAdFCpHp0vhWB6iQoY4sXgf6x7q2iLzQRCvk3VXjV8iniDL9dk58TIUuZGKn+yKs1kTy9sO1qOEle/O5SJ2l64GcUQlxV1mF+pKOGAA/qRT4qjie8Oid6A2UxaerNKUFaiG/TBrVa8KMZLdXxISXjKEBFVJc3ryEeF5qajodTE96GcqE+7yn2DZCQWFtpc9HBNEpZ/c7PSDUSDV2LPucoeo5x6Y=
  - secure: AsWrqOJRZcpojCGivqRn7p9wc7JH/hB3Qbkj4hzqTKOI3pMoYjrGW28mDvanfY32j5INccLWdEXYOohUY+iLM5+WdX7FSx2Vx4DuXtRzxCfqpsFD5z80J6DSSGBUk/3soTX3ip91No0gnisbTCRgXARy1P9gwvqT+nh1SmDA/Q/KxocvbQ6p6gOQrFY87XjtXryFuVjAPR+a0lHryH/r+qWYIJbtfYJZadnXuBzZY1uLsndmOax7xHdldB1LOkSedKrFT7yVSc9i7IOEPX1Fx0mG1k6Q1u/Hhrz5vA17vqG58+sVxQ6XQkgxu2N1ESEMoMD+l8cs1IiOC7boMCNLRHIoVolnazXLXhUwSCUYpOdYMn0KcUB1BYP4SbHWYaeDFgVa3eaZMnf958QFzqjXupcef7HIysyBK/oitsrrVpMp17C3o+25pp9YXIKiwOKz4E6N7sNhcrLqfRyTcljeb/iZtNkdqyWQFvpz3uxrAwz67GIWrTzUy73QSjLGkwmDOGbthl1fIUN80/Y8VdszJMSfeEh8p9CHmHT+oCvBXamw04YgcdTR1aJ6mcDXl4C+cZpbQyy9UDwU2GW6VA5sYVT07vruVf5fwsNgWx7pcUECaQnCUTBykGlLPiTD5/0qx+q9ZZCSiEixpz7B032AmLYwKtxGJvbKnMOKDKmnKoM=
  - secure: sEUbqdbnD7te3Q4LfzEoRNOwcMyze6tUdhpMvtG9ld5ohj3iOEaz9b7IYqYnmv7iIgX6QReldxZJg0bpRaO0sFUX82ejmK5FNDpGVsGCevh4K3lbTOnieDjxvyrHg40KdR3RU0WWMKzhlKc938FcVBSEplYR4RtWlgdlKvgQquNJCi5qC3gObvMfuN1cOHG2/Ztf5U+DxZo0c/5KULuJxuPIumzhnd8dZLgGNiBrXbSgq0IZBfRGc0jMX6HcNCjRY9SirS1/yccF59BE6iu0wWPzt5pHTlG+OxN1Gj+zoZ5sqlZbkC1DqIP2EQusT5XJtZIySkYqbQrukQVUzfcszTS/Lf+RS6k/fElsMBwPDQUZSTsqjds7Q5JhufYgsiiU1zp6P87r18O9ccI10/rmZHQDib9xMfz3s8O2hiR1cakhYd5UlWiKzN3lF36siBhTa7AAqH6m+pE/WKj4b2GS/I7CY5KvedLAjWfraYCkSNbTuiU/16D1iiVhfvQKLYUBWdo3x7lPjeAwh1ANhhSzn+qdlYA8ZbYa0wOAbmmoJrxfI07ouoVwYACYLwiudpq9UN62ZObcvzdo9Alv57aRIGnrSZJ34IjRbdRZTWthChkg5cxkO/15UgGJhwEtFSQNXWJPrinvNZ39Ug/O7BP6seZM48hqIk/u5BWClZYyIEo=
after_success:
 - docker login -e="$DOCKER_EMAIL" -u="$DOCKER_USER" -p="$DOCKER_PASS"
 - docker build -f Dockerfile -t $DOCKER_IMAGE_NAME .
 - if [ ! -z "$TRAVIS_TAG" ]; then
      docker tag $DOCKER_IMAGE_NAME:latest $DOCKER_IMAGE_NAME:$TRAVIS_TAG;
    fi
 - if [ "$TRAVIS_BRANCH" == "master" ]; then
      docker push $DOCKER_IMAGE_NAME;
    fi

```

Commit and push the changes. Once Travis-CI builds the application successfully, it will do the same for the docker image. If the build was triggered by a commit to master, the image is pushed to Docker Hub. This may take a little bit of time so be patient.

## Deploying to DigitalOcean with Docker Cloud <a name="deploy"></a>

To continue with this step you will need an account on [DigitalOcean](https://cloud.digitalocean.com/registrations/new), a cloud-based software infrastructure specific to software development. We'll deploy the application using a __droplet__. The smallest one can be used but a cost will still be incurred. Don't worry, the cost is [minimal](https://www.digitalocean.com/pricing/) - less than a cent.

> Don't forget to destroy the droplet when you're done!

After this, log into [Docker Cloud](https://cloud.docker.com/login) and link your DigitalOcean account. To do this, click on 'Cloud Settings' at the bottom left. You should now see a list of Cloud Providers on the page. Click on the plug icon next to DigitalOcean to link your accounts. Note that, at the time of writing, there is a $20 credit added to your DigitalOcean account when linked with Docker Cloud. For the purposes of our demonstration the costs will now be free!


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/d0ffee77-f47c-41ac-942d-3d02f8b3d42f.png)


Once your accounts are linked, create a new DigitalOcean Node Cluster. I'm going to call mine, *horizon-with-docker* in the region *Toronto 1*, but you're free to choose any name and region.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/82fcfbc6-5e4c-4038-8ce4-152c54bbf711.png)


The newly created node cluster can be used to run a stack. A stack is a collection of services and each service is a collection of containers. Stacks are created with the [*stack-yaml* file](https://docs.docker.com/docker-cloud/apps/stack-yaml-reference/).

Remember the *docker-compose.yml* used for Docker Compose to launch our application? That wasn't *really* necessary for deployment to DigitalOcean. Although, it's great for testing and local development.

The compose file isn't needed here and but it's going to be very similar to the stack yaml. Click stacks, then create. It will then ask for the stack yaml:

stack.yml

``` yaml 

db:
  image: 'rethinkdb:latest'
  ports:
    - '8080:8080'
    - '28015:28015'
web:
  environment:
    - RDB_HOST=db
    - RDB_PORT=28015
  image: 'casche/horizon-with-docker:latest'
  links:
    - db
  ports:
    - '80:8081'

```

> Don't forget to change the tag to the tag you used inside your *.travis.yml*. That is, the tag of your docker image on Docker Hub.

The port of the Horizon application has been changed to the default, http port 80. Go ahead and give it a meaningful stack name, such as 'Horizon-with-Docker':


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/9109de97-3698-4f44-9d10-15b7e5e6970d.png)


Once created, revisit the node cluster created earlier to grab the IP address of your DigitalOcean droplet - my droplet IP is 159.203.61.66. Go ahead and visit your freshly deployed Horizon application at the IP address. Congratulations!

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/19a7f3c5-6a35-4935-8a81-82539b522a4a.png)


But wait, there's just one more thing: __automatic deployments__. Each time the Travis-CI build succeeds on master, Docker Cloud should redeploy the applications image.

Click on services, then click on the service that's running your Horizon application. If you've kept a similar naming scheme, it will be called *web*. Scroll down until you see 'Triggers'. Create a new redeployment Trigger called 'image deploy'.  


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ff4cb118-9f52-40d9-8461-d87c88091f37.png)


Return to Docker Hub and select your docker image for the Horizon application. Go to the Web Hooks tab and create a new webhook called 'digitalocean-deploy'. Use the web address from the Trigger created above and append it to https://cloud.docker.com/.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/68d20f18-f0bf-4fc9-ae3c-cad2b576e0d4.png)


Now, each time an image is deployed to Docker Hub, Docker Cloud will redeploy the container with the new image to DigitalOcean! Awesome!

## Closing Thoughts

This tutorial has outlined a development flow for Horizon on DigitalOcean using Docker Cloud. It is by no means complete and should be adjusted for production usage. For more on adapting for productive deployments, you can check out the official Horizon documentation [here](http://horizon.io/docs/docker/).

Thanks for reading. Please let me know your comments and feedback through the discussion section below. You can also follow me on twitter @devcasche. 