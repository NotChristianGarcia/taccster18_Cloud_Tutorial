# Introduction to Cloud Computing


This module will be partially interactive. Please follow along on your own computer.


## What is a cloud?

A "cloud" is a computer system that provides users with shared access to on-demand computing resources via the internet.


<center><img src="../../resources/cloud.jpg" style="height:300px;"></center>


### Goals/Benefits
  * Resource utilization
  * Scalability and Elasticity
  * Reproducibility by way of Programmability
  * Reliability through redundancy


### Cloud Means Different Things to Different Groups

Different kinds of people use clouds for different purposes.
  * System Administrators - use cloud to automate operations.
  * Software Developers - build applications on cloud servers and platforms.
  * Computational scientists - Write codes to analysis scientific data in the cloud.

In this course we will try to exposure you to each of these views.

### Cloud service models

Clouds can offer different service models:
  * Infrastructure-as-a-service (Iaas) - virtual servers, networks, firewalls, etc. (AWS, Azure)
  * Platform-as-a-service (Paas) - deploy applications without managing virtual servers (Google App Engine, Heroku)
  * Software-as-a-service (Saas) - Ready to use software application (Gmail, Office365)

Also: emerging models such as functioni-as-a-service.

### Infrastrcture-as-a-service

Key concepts:
  * Virtual Machines (VMs) - Simulate a physical computer through software.
  * Software defined networking: Routers, networks and subnets - used to connect VMs to other computers.
  * Security groups - firewall rules enabling or disabling network traffic to/from ports on the VMs.

### Hands on: Launching a VM on the JetStream cloud

1. Navigate to the JetStream Horizon interface: https://tacc.jetstream-cloud.org
2. Login with your tacc User Name and Password and enter "tacc" for the Domain.
3. Make sure you see our Training project (TG-TRA170023) in the projects drop down at the top and select it if it is not already selected (check mark):
   <center><img src="../../resources/jetstream_horizon_projects.png" style="height:300px;"></center>
4. Select Access & Security -> Key Pairs -> Create Key Pair. Give your key a name and click "Create Key Pair". When prompted, download the key file to your local desktop or click the Download link.
5. Instances -> Launch Instance.
  * Details: Name your instance something like your_name01 and click Next (Don't click launch instance)
  * Source: Click the "No" under "Create New Volume"
  * Source: Enter "Ubuntu 18.04 Devel and Docker" in the search and click the plus (+) to select that image.
  * Flavor: Click the plus next to m1.medium
  * Networks: Choose the TG-TRA170023-subnet by clicking plus.
  * (Skip Network Ports by clicking next)
  * Security Groups: Select default security group.
  * Key Pair: Select the key you just created in step 4.
  * Click Launch instance.
6. Once the VM has been spawned, click the arrow at the far right to select "Associate Floating IP" from the available instance actions.
<center><img src="../../resources/jetstream_horizon_instance_actions.png" style="height:300px;"></center>
7. Select an IP in the drop down (if there is not one, click the plus (+) symbol to allocate one) and click Associate.


### Hands on: verifying connectivity to the VM

Use the SSH key and IP address you generated in steps 4 and 7 above, respectively to test connectivity to your new VM.

OSX/Linux:

```
 Open the application 'Terminal'
  # make sure your key file has the correct permissions:
  $ chmod 0600 chmod 0600 ~/Downloads/<key_name>.pem
  # connect to the VM over SSH
  $ ssh -i ~/Downloads/<key_name>.pem root@<IP address>
```

Windows:

```
 Open the application 'PuTTY'
  enter Host Name: <IP address>
  (click 'Open')
  (enter "ubuntu" username)
  (select your key)
```

If all goes well you should see a command prompt that looks something like:

```
Last login: Sat Jul 22 17:52:45 2017 from dhcp-146-6-176-22.tacc.utexas.edu
root@test-jfs1:~$
```


## Intro to Docker

Intro slides: https://docs.google.com/presentation/d/1RVyQ1CDamNLRw9mIcUec-IoTcH0-wbQcXsjbRB5NAws/edit?usp=sharing


### Initial setup
Typically, accessing the docker daemon requires root or to be in the docker group. For the purposes of this introduction,
 we can simply do everything as the root user:
```
$ sudo su - root
```

Make sure you can access the docker daemon; you can verify this by checking the version:
```
$ docker version
Client:
 Version:      1.13.1
 API version:  1.26
 Go version:   go1.6.2
 Git commit:   092cba3
 Built:        Thu Nov  2 20:40:23 2017
 OS/Arch:      linux/amd64

Server:
 Version:      1.13.1
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.6.2
 Git commit:   092cba3
 Built:        Thu Nov  2 20:40:23 2017
 OS/Arch:      linux/amd64
 Experimental: false

```

Create a test directory to contain your docker work:
```
$ mkdir docker; cd docker
```

### Docker Images and Tags, Docker Hub, and Pulling Images
A Docker image is a container template from which one or more containers can be run. It is a rooted filesystem that,
by definition, contains all of the file dependencies needed for whatever application(s) will be run within the
containers launched from it. The image also contains metadata describing options available to the operator running
containers from the image.

One of the great things about Docker is that a lot of software has already been packaged into Docker images. One source
of 100s of thousands of public images is the official docker hub: https://hub.docker.com.

The docker hub contains images contributed by individual users and organizations as well as "official images". Explore
the official docker images here: https://hub.docker.com/explore/

Most modern programming languages offer an official image. For example, there is an official image for the Python programming language: https://hub.docker.com/_/python/

Docker supports the notion of image tags, similar to tags in a git repository. Tags identify a specific version of an
image.

The full name of an image on the Docker Hub is comprised of components separated by slashes. The components include a
"repository" (which could be owned by an individual or organization), the "name", and the "tag". For example, an image
with the full name
```
jstubbs/test:0.1
```
would be the "test" in the "jstubbs" repository and have a tag of "0.1". TACC maintains multiple repositories on the Docker Hub
including:
```
tacc
taccsciapps
agaveapi
abaco
```
Official images such as the python official image are not owned by a repository, but all other images are.

To pull an image off Docker Hub use the `docker pull` command

```
$ docker pull python
Using default tag: latest
latest: Pulling from library/python
cc1a78bfd46b: Pull complete
. . .
```

As indicated in the output, if no tag is specified the "latest" tag is pulled. You can verify that the image is
available on your local machine using the `docker images` command:
```
$ docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
jstubbs/test           latest              9dfe5a2c4b43        52 minutes ago      81.2 MB
python                 latest              a5b7afcfdcc8        3 hours ago         912 MB
```

### Running a Docker Container
We use the `docker run` command to run containers from an image. We pass a command to run in the container. Similar
to running other progams on Unix systems, we can run containers in the foreground (attached) or in the backgrouns.

#### Running and Attaching to a Container
To run a container and attach to it in one command, use the `-it` flags. Here we run `bash` in a container from the ubuntu image:
```
docker run -it ubuntu bash
```
#### Running a Container in Daemon mode ####
We can also run a container in the background. We do so using the `-d` flag:
```
docker run -d ubuntu sleep infinity
```
Keep in mind that the command given to the `docker run` statement will be given PID 1 in the container, and as soon as this process exits the container will stop.


### Installing and Running the Containerized Classifier Application

We have packaged up a self-contained image classifier application based on the TensorFlow library. Our image is
available from the DockerHub. To download it, issue the following commnand:

```
$ docker pull taccsciapps/classify_image
```

What you are downloading is a complete application that will run an image classifier algorithm on an image
that is publicly available from the internet.

Once the download is complete, we can run one or more containers from our image. This particular image requires
an environment variable, `$URL`, containing the address to a publicly available image, to be passed in at run time.

For example, we can run our image classifier program on the `dog.jpg` image contained in the github repo for this turotial with:

```
$ docker run -it --rm -e URL=https://raw.githubusercontent.com/TACC/taccster18_Cloud_Tutorial/master/classifier/data/dog.jpeg  taccsciapps/classify_image
```

### Additional Remarks on Running Containers

#### Running Additional Commands in a Running Container ####
Finally, we can execute commands in a running container using the `docker exec` command. First, we need to know the container id, which we can get through the `docker ps` command:

```
~ $ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
a2f968b8443f        ubuntu:16.04        "sleep infinity"    9 seconds ago       Up 8 seconds                            awesome_goldwasser
```

Here we see the container id is `a2f968b8443f`. To execute `bash` in this container we do:
```
docker exec -it a2f968b8443f bash
```
At this point we are attached to the running container. If our bash session exits, the container will keep running because the `sleep infinity` command is still running.

*Note: The `docker ps` command only shows you running containers - it does not show you containers that have exited. In order to see all containers
on the system use `docker ps -a`.


### Removing Docker Containers ###
We can remove a docker container using the `docker rm` command (optionally passing `-f` to "force" the removal if the container is running). You will need the container name or id to remove the container:
```
$ docker rm -f a2f968b8443f
```


### Building Images From a DockerFile
We can build images from a text file called a Dockerfile. You can think of a Dockerfile as a recipe for creating images.
The instructions within a dockerfile either add files/folders to the image, add metadata to the image, or both.

#### The FROM instruction
We can use the `FROM` instruction to start our new image from a known image. This should be the first line of our Dockerfile. We will start our image from an official Ubuntu 16.04 image:

```
FROM ubuntu:16.04

```

#### The RUN instruction
We can add files to our image by running commands with the `RUN` instruction. We will use that to install `wget` via `apt`. Keep in mind that the the docker build cannot handle interactive prompts, so we use the `-y` flag in `apt`. We also need to be sure to update our apt packages.

The Dockerfile will look like this now:
```
FROM ubuntu:16.04

RUN apt-get update && apt-get install -y wget
```

#### The ADD instruction
We can also add local files to our image using the `ADD` instruction. We can add a file `test.txt` in our local directory to the `/root` directory in our container with the following instruction:

```
ADD test.txt /root/text.txt
```

A complete Dockerfile for the classify_image application as well as the necessary scripts and supporting files is available from the Tutorial gihub repo:
https://github.com/TACC/taccster18_Cloud_Tutorial/tree/master/classifier


#### Building the Image
To build an image from a Dockerfile we use the `docker build` command. We use the `-t` flag to tag the image: that is, give our image a name. We also need to specify the working directory for the buid. We specify the current working directory using a dot (.) character:
```
docker build -t classify_image .
```

Note that this command requires all the files in the `classifier` directory in the Tutorial github repo (https://github.com/TACC/taccster18_Cloud_Tutorial/tree/master/classifier)