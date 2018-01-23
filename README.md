## Introduction

This tutorial will walk you through the steps of setting up a RabbitMQ cluster hosted on Docker containers based on Ubuntu Xenial (16.04) image. In this guide, we discuss how to create an Ansible playbook to automate everything done previously.

### Create a New SSH Key Pair

If you do not already have an SSH key pair, create an RSA key-pair by typing:

```markdown
ssh-keygen
```
You will be asked to specify the file location of the created key pair, a passphrase, and the passphrase confirmation. Press ENTER through all of these to accept the default values. Your new keys are available in your user's ~/.ssh directory. The public key (the one you can share) is called id_rsa.pub. type this to get the contents of your public key:

```markdown
cat ~/.ssh/id_rsa.pub
```
For more details see [How to Install and Configure Ansible on an Ubuntu 12.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-an-ubuntu-12-04-vps). The string that is given back to you is what you need to paste in the second field in the [Ormuco](https://ormuco.com) control panel:

![](KeyPanel.png?raw=true)

Click "Create" to add your key to the control panel. 

### Install docker-ce package

Now it’s time for some Docker! First, let's create three instances with at least 1 CPU and 1GB of memory and attach a floating IP to each instance. Before you install Docker CE for the first time on a new host machine, you need to set up the Docker repository:

1. Update the apt package index:

```markdown
$ sudo apt-get update
```

2. Install packages to allow apt to use a repository over HTTPS:

```markdown
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

3. Add Docker’s official GPG key:

```markdown
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

4. Docker CE has two update channels, stable and edge. Use the following command to set up the **stable** repository. 

```markdown
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

The lsb_release -cs sub-command below returns the name of your Ubuntu distribution, such as xenial.

**Install Docker CE**

5. Update the apt package index

```markdown
$ sudo apt-get update
```

6. Install the latest version of Docker CE

```markdown
$ sudo apt-get install docker-ce
```

To install a specific version, append the version string to the package name and separate them by an equals sign (=):

```markdown
$ sudo apt-get install docker-ce=<VERSION>
```

For more details see [Get Docker CE for Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/).

### Create Your Docker Container !

We must choose which image to download and use to create our Docker container. Use the pull command to download an image. Let’s download the basic ubuntu 16.04 image:

```markdown
$ docker pull ubuntu:16.04
```

You can check this has downloaded the image to your local store with the above **docker images command**.

![](DockerImages.png?raw=true)

The next step is to create a container and make the required changes. Creating a container is done with the **run command**. The basic docker run command takes this form:

```markdown
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

When you run a container, you can use the --network flag to specify which networks your container should connect to. The ```host``` network adds a container on the host’s network stack. In line with this, The docker run command takes this form:

```markdown
$ docker run --network=host -itd --name=container 2a4cca5ac898  /bin/bash
```

Now, each container in the network can immediately communicate with other containers in the network. And that’s all there is to it! You have created a new Docker container.

### Install rabbitmq-server

We now have to install ```rabbitmq-server``` (from the official repository) within each container. 

1. Run the following command to update the package list:

```markdown
$ sudo apt-get update
```

2. Install rabbitmq-server package:

```markdown
$ sudo apt-get install rabbitmq-server
```

