# docker_tutorial

## Prerequisites

* Install [Docker for Desktop](https://docs.docker.com/engine/installation/#desktop)

[Docker](https://www.docker.com/what-container) has great tutorials for beginners as well as advanced users. We're assuming some basic Docker knowledge in this tutorial but the basics are worth reviewing. In her own words:

> A container image is a lightweight, stand-alone, executable package of a piece of software that includes everything needed to run it: code, runtime, system tools, system libraries, settings. Available for both Linux and Windows based apps, containerized software will always run the same, regardless of the environment. Containers isolate software from its surroundings, for example differences between development and staging environments and help reduce conflicts between teams running different software on the same infrastructure.

![Docker donated $25,000 to the Oceanic Society in Ross, California and in exchange the organization denoted one of the whales, Molly Dock, that its affiliated scientists have been tracking for years as being sponsored by Docker. hence the logo](https://github.com/012345/docker_tutorial/blob/master/Screen%20Shot%202017-11-14%20at%207.33.49%20PM.png)



## Docker Command Line Basics

Make sure Docker is running. Open a terminal window and see what commands are available. If this errors then you need to start the docker daemon. 

```docker```

List images which have been pulled

```docker images```

If you see a bunch of our competitors listed you can easily remove them with (this is a joke. kind of): 

```docker rmi %image_id%```

Now we need to get the DataStax Images!
## What is Available

Use DataStax provided Docker images in non-production environments for development, to learn DataStax Enterprise, OpsCenter and DataStax Studio, to try new ideas, or to test and demonstrate an application. The following images are available:

Docker Store:
* [DataStax Enterprise](https://hub.docker.com/r/datastax/dse-server/): The best distribution of Apache Cassandra™ with integrated Search, Analytics, and Graph capabilities

Docker Hub:
* [DataStax Studio](https://hub.docker.com/r/datastax/dse-studio/): An interactive developer’s tool for DataStax Enterprise which is designed to help your DSE database, Cassandra Query Language (CQL), DSE Graph, and Gremlin Query Language development.
* [DataStax Enterprise OpsCenter](https://hub.docker.com/r/datastax/dse-opscenter/): The web-based visual management and monitoring solution for DataStax Enterprise (DSE)

## Get the Images

DataStax Enterprise - DataStax Enterprise is located in Docker Store. That means that there are a couple extra steps to download the image. 
1. Register an account with [Docker Store](https://store.docker.com/) (free)
2. Subscribe to [DataStax Enterprise](https://hub.docker.com/r/datastax/dse-server/) (also free)
3. You're now subscribed to the image and can always pull it with no extra steps

DataStax Studio and Opscenter are in Docker hub so getting them is straight forward. 

Pull DataStax Enterprise, Studio, and OpsCenter

```docker pull store/datstax/dse-server:latest```

```docker pull datastax/dse-studio```

```docker pull datastax/dse-opscenter```

## Running the Containers


Option | Description
------------- | -------------
`-e` | (**Required**) DS_LICENSE=accept Sets [Environment variables](#using-environment-variables) to accept the licensing agreement and <BR> (optional) change the initial configuration.
`-d` | (Recommended) Starts the container in the background.
`-p` | Publish container ports on the host computer to allow remote access to DSE, OpsCenter, and Studio. See [Exposing DSE public ports](#exposing-dse-public-ports)
`-v` | Bind mount a directory on the local host to a DSE Volume to manage configuration or preserve data. See [Volumes and data](#volumes-and-data). 
`--name` |Assigns a name to the container.

These are the most commonly used `docker run` switches used in deploying DSE.  For a full list please see [docker run](https://docs.docker.com/engine/reference/commandline/run/) reference.

### Enabling advanced functionality

By default, the DSE server image is set up as a transactional (database) node.
To set up the node with DSE advanced functionality, add the option that enables feature to the end of the `docker run` command.

DSE start switches | Description
------------- | -------------
-s | Enables and starts DSE Search.
-k | Enables and starts Analytics.
-g | Enables and starts a DSE Graph.

You can combine the options to run more than one feature. For more examples, see the Starting [DSE documentation](http://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/operations/startStop/startDseStandalone.html).

Let's start a single node of DSE 

```docker run -e DS_LICENSE=accept --name my-dse -d datastax/dse-server```

We started this container in the background by using the ```-d``` option. Use the following command to get an interactive shell to your DSE node.

```docker exec -it my-dse bash```

If you're sitting down and totally ready to be amazed. Move on to the next step and get yourself a cql shell :

```cqlsh```

Easy right? Okay, we have more to learn. Exit that shell. We'll come back to it.

```exit``` 

```exit```

Let's take a quick look at the docker logs just so you know how:

```docker logs my-dse```

Cool right? But lets say that you just want to see the output of a common status command without getting that interactive shell. It's easy:

```docker exec -it my-dse nodetool status```

Lets stop our running container. You don't have to stop the running container but we're starting with a clean slate is the easiest way to learn as is repetition. as is repetition. as is repetition. :

```docker stop my-dse```

```docker rm /my-dse```


### Creating an OpsCenter container

Next we'll create an OpsCenter container. OpsCenter exists to manage DSE, so we'll need a dse-server node also. 

Create an OpsCenter container:

```docker run -e DS_LICENSE=accept --name my-opscenter -d -p 8888:8888 datastax/dse-opscenter```

Create DataStax Enterprise server nodes, providing the link to the OpsCenter container using the Docker name.

```docker run -e DS_LICENSE=accept --link my-opscenter:opscenter --name my-dse -d datastax/dse-server```

Get the IP Address of your your new DSE node. 

```docker inspect my-dse | grep '"IPAddress":'```

Open a browser and point to `http://DOCKER_HOST_IP:8888`. The *Create the new connection* dialog displays.
1. Click **Manage existing cluster**.
2. Enter the IP Address for the DSE Node (from previous step) 
3. Choose **Install agents manually**.

The agent is already available on the DSE node, no installation is required.

See [OpsCenter documentation](http://docs.datastax.com/en/opscenter/6.1/) for further info on usage/configuration. Note that LCM is not supported and should not be used for containerized environments. Other tasks like scheduling commitlog backups would require writing a file in DSE which isn't possible in immutable containers. These tasks much be handled manually using DSE configuration files. 


### Creating a Studio container

1. Create a Studio container:

```
docker run -e DS_LICENSE=accept --link my-dse --name my-studio -p 9091:9091 -d datastax/dse-studio
```

2. Open your browser and point to `http://DOCKER_HOST_IP:9091`, create the new connection using my-dse as the hostname. 

Check [Studio documentation](http://docs.datastax.com/en/dse/5.1/dse-dev/datastax_enterprise/studio/stdToc.html) for further instructions.

##Docker Compose - Building a Simple Cluster

Okay. Lets keep going. We need a cluster.

>Fun fact. Until the 'thing' is running it's an image. When running it's a container. Stale conversation at a party? Might I suggest this opener "Isn't it interesting that you don't actually call something a 'container' until it's running. Up to that point it's just an image.

Okay back to work. 


Bootstrapping a multi-node cluster with OpsCenter and Studio can be elegantly automated with [Docker Compose](https://docs.docker.com/compose/). To get sample `compose.yml` files visit the following links.  

* [DSE](https://github.com/datastax/docker-images/blob/master/docker-compose.yml)  

* [OpsCenter](https://github.com/datastax/docker-images/blob/master/docker-compose.opscenter.yml)  

* [Studio](https://github.com/datastax/docker-images/blob/master/docker-compose.studio.yml)


**3-Node Setup**

```
docker-compose up -d --scale node=2
```

**3-Node Setup with OpsCenter**

```
docker-compose -f docker-compose.yml -f docker-compose.opscenter.yml up -d --scale node=2
```

**3-Node Setup with OpsCenter and Studio**

```
docker-compose -f docker-compose.yml -f docker-compose.opscenter.yml -f docker-compose.studio.yml up -d --scale node=2
```

**Single Node Setup with Studio**

```
docker-compose -f docker-compose.yml -f docker-compose.studio.yml up -d --scale node=0
```

##Next you should 
* Check out the full docs for examples of how to how to modify configurations with the snazzy [conf directory](https://github.com/datastax/docker-images/blob/master/README.md#using-the-dse-conf-volume) or [environment variables](https://github.com/datastax/docker-images/blob/master/README.md#using-environment-variables)
* Try a more advanced tutorial from [DataStax Academy](academy.datastax.com)
