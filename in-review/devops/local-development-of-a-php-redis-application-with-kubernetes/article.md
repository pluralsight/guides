The conventional approach to developing applications on Kubernetes, an open-source tool for deploying containerized applications, requires developers to write code, commit to GitHub, build a Docker container, and deploy that container into Kubernetes.

In this tutorial, I'll introduce you to a simpler, sleeker approach. We'll edit a PHP application locally, and run a *virtual copy* of the PHP application as a cloud-hosted Kubernetes instance. We'll be running Redis in Kubernetes. The best part about this: the PHP application will have **full access** to the Kubernetes resources, but you'll be able to edit it locally.

## Prerequisites

### Technologies used

* [Kubernetes](https://kubernetes.io)
* [Google Container Engine](https://cloud.google.com/container-engine/)
* [Telepresence](http://www.telepresence.io)
* [PHP](http://www.php.net/) and [Redis](https://redis.io/)

This tutorial has been tested on a modern version of Linux and Mac OS X. We'll also use the Google Container Engine as our Kubernetes cluster, although any Kubernetes installation will work.

We're going to use the [Guestbook](https://cloud.google.com/container-engine/docs/tutorials/guestbook) sample application, which runs on basic PHP and Redis, in our example.

### Setup

We'll start by setting up the Command Line Interface (CLI) for the Google Container Engine (GKE) locally. Follow the instructions at [the Cloud Downloads page](https://cloud.google.com/sdk/downloads) to download and install the latest Cloud SDK. Then, install `kubectl`:

```
% sudo gcloud components update kubectl
```

We need to install Telepresence, which will proxy your locally running service to GKE. Note that this tutorial uses 0.26 of Telepresence. To use the latest version, visit http://www.telepresence.io.

```
% curl -L https://github.com/datawire/telepresence/raw/0.26/cli/telepresence -o telepresence
% chmod +x telepresence
```

Move Telepresence to somewhere on your $PATH:

```
% mv telepresence /usr/local/bin

**OR**
export /path/to/telepresence:$PATH
export $PATH
```

You'll also need to install `torsocks`. On Mac OS X, use `brew`:

```
% brew install torsocks
```

Or, if you're on Ubuntu, use `apt`:

```
% sudo apt install --no-install-recommends torsocks
```

We'll also configure a local development environment for PHP that can run the Guestbook application. We'll install the [PEAR package manager](https://pear.php.net/manual/en/installation.getting.php), and then install the Predis library which the PHP application uses.

```
% curl -O https://pear.php.net/go-pear.phar
% php go-pear.par
% pear channel-discover pear.nrk.io   # You may need to add pear to your path, if so, use one of the methods above used with telepresence
% pear install nrk/Predis
```

This tutorial uses several Kubernetes configuration files. You can optionally clone the [Telepresence GitHub repo](https://github.com/datawire/telepresence/) to get a local copy of them all:

```
% git clone https://github.com/datawire/telepresence.git
```

All example files are in the [`examples/guestbook`](https://github.com/datawire/telepresence/tree/master/examples/guestbook) directory.

### Setting up Kubernetes in Google Container Engine

We're going to use Google Container Engine as our Kubernetes cluster. Start by going to [the Cloud Console page](https://console.cloud.google.com). Then choose the Google Container Engine option from the menu and Create a Cluster.

Now, use the command line to authenticate to the cluster:

```
% gcloud container clusters get-credentials CLUSTER_NAME
% gcloud auth application-default login
```

### The Guestbook application

We're now going to start installing the Guestbook application. 

First, we need to set up the Redis master deployment ([config](https://github.com/datawire/telepresence/blob/master/examples/guestbook/redis-master-deployment.yaml)), the Redis master service ([config](https://github.com/datawire/telepresence/blob/master/examples/guestbook/redis-master-service.yaml)), the Redis slave deployment ([config](https://github.com/datawire/telepresence/blob/master/examples/guestbook/redis-slave-deployment.yaml)), and the Redis slave service ([config](https://github.com/datawire/telepresence/blob/master/examples/guestbook/redis-slave-service.yaml)). If you don't want to download each of these files manually, these files are in the [`examples/guestbook`](https://github.com/datawire/telepresence/tree/master/examples/guestbook) directory of the Telepresence repository.

```
% kubectl create -f redis-master-deployment.yaml
% kubectl create -f redis-master-service.yaml
% kubectl create -f redis-slave-deployment.yaml
% kubectl create -f redis-slave-service.yaml
```

You can verify that everything is running by using the `get pods` command:

```
% kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
redis-master-343230949-lpw91   1/1       Running   0          1d
redis-slave-132015689-dpp46    1/1       Running   0          1d
redis-slave-132015689-v06md    1/1       Running   0          1d
```

### Connecting the Guestbook frontend to Redis

We're going to use [Telepresence](http://www.telepresence.io) to create a special pod in the remote Kubernetes cluster that proxies your local code. This way, the PHP application will be able to talk to remote cloud resources, and vice versa.

We'll start by deploying the Telepresence proxy onto the Kubernetes cluster using the [`telepresence-deployment.yaml`](https://github.com/datawire/telepresence/blob/master/examples/guestbook/telepresence-deployment.yaml) deployment file.

```
% kubectl create -f telepresence-deployment.yaml
```

We need to start an external load balancer. The [`frontend-service.yaml`](https://github.com/datawire/telepresence/blob/master/examples/guestbook/frontend-service.yaml) uses a selector for `app:guestbook` and `tier:backend`. We've used these labels in the `telepresence-deployment.yaml`, so the load balancer will talk directly to our proxy.

```
% kubectl create -f frontend-service.yaml
```

We're going to start the local Telepresence client, which creates a special shell that connects to the proxy in GKE.

```
% telepresence --deployment telepresence-deployment --expose 8080 --run-shell
```

In this shell, `cd` into the `examples/guestbook` directory and start the frontend application. To do this, you need to locate the directory where Predis is installed. You can figure this out by typing:

```
% pear config-get php_dir
```

Now, in the `examples/guestbook` directory, and start PHP:

```
% php -d include_path="PATH_TO_PEAR_DIR" -S 0.0.0.0:8080
```

It's time to view our app in the browser. Let's look up the IP address of our external load balancer using `get services`:

```
% kubectl get services
NAME           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
frontend       10.7.251.109   106.196.217.34   80:30563/TCP   25m
redis-master   10.7.247.217   <none>           6379/TCP       5d
redis-slave    10.7.244.48    <none>           6379/TCP       5d
```

Enter the external IP address of your load balancer into a browser like Google Chrome. In the above example, the `EXTERNAL-IP` is 106.196.217.34. You'll see the Guestbook application running. Type a message, and you'll see it saved into Redis.

### Edit your code

Open `index.html` from your shell and try renaming the "Submit" button to something else. Save and reload. You'll immediately see your changes take effect on the external IP address.

Stop the PHP process, and type `exit` to terminate the Telepresence proxy.

### How this works

Telepresence works by building a two-way network proxy between a special pod running inside GKE and a process running locally. Your incoming request goes to the external load balancer, which then sends the request to the Telepresence proxy, which then proxies those requests to the local Telepresence client, which then passes the request to the PHP application.

## More reading

* The [Microservices Architecture Guide](https://www.datawire.io/guide) covers design patterns and HOWTOs in setting up an end-to-end microservices infrastructure
* The [Kubernetes tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/) gives a good walk-through of using Kubernetes, or visit the [Google Container Engine Quickstart](https://cloud.google.com/container-engine/docs/quickstart)

## Conclusion

Developers love fast dev cycles. In this tutorial, we've shown how you can edit code locally, save changes, and immediately run that code in Kubernetes, all without waiting for a Docker build cycle. We did this with a simple webapp, but such an approach is **even more useful** when you adopt microservices and build up applications. Imagine you have dozens of services in your application; all services can run in your remote Kubernetes cluster, while you still develop locally.

____ 

Happy coding! Don't forget to add this article to your favorites. Please leave all comments and feedback in the discussion section below.
