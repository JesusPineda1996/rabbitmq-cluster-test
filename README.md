## Introduction

This tutorial will walk you through the steps of setting up a RabbitMQ cluster hosted on Docker containers based on Ubuntu Xenial (16.04) image. In this guide, we discuss how to create an Ansible playbook to automate everything done previously.

## Create a New SSH Key Pair

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

## Install docker-ce package

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

1. Update the apt package index
```markdown
$ sudo apt-get update
```
2. Install the latest version of Docker CE
```markdown
$ sudo apt-get install docker-ce
```
To install a specific version, append the version string to the package name and separate them by an equals sign (=):
```markdown
$ sudo apt-get install docker-ce=<VERSION>
```

For more details see [Get Docker CE for Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/).

## Create Your Docker Container !

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

## Install rabbitmq-server

We now have to install ```rabbitmq-server``` (from the official repository) within each container. 

1. Run the following command to update the package list:
```markdown
$ sudo apt-get update
```
2. Install rabbitmq-server package:
```markdown
$ sudo apt-get install rabbitmq-server
```

## Setup the RabbitMQ cluster

In the following steps, we will name the nodes as node01, node02, node03, and so on.

1. Stop the RabbitMQ server on the nodes if it is already running:
```markdown
$ service rabbitmq-server stop
```
2. On node01, append all the IP-name bindings to ```/etc/hosts```, so it contains
something as follows (put your IP addresses here):
```markdown
192.34.79.203 node01
192.34.79.187 node02
192.34.79.15  node03
```
3. Copy the ```/etc/hosts``` file from node01 to all the other nodes in order to have the
hostname definitions aligned. Use the ```echo``` or ```scp``` command for this porpuse.
4. Copy the ```/var/lib/rabbitmq/.erlang.cookie``` file from node01 to all the other nodes. Use the ```echo``` or ```scp``` command for this porpuse.
5. Restart the RabbitMQ server on all the nodes:
```markdown
service rabbitmq-server start
```
6. Join all the nodes to node01. On all the nodes except node01, run the
following command:
```markdown
rabbitmqctl stop_app
rabbitmqctl join_cluster --ram rabbit@node01
rabbitmqctl start_app
```
This process has been thoroughly described by Boschi in the book titled [RabbitMQ Cookbook](https://www.amazon.com/RabbitMQ-Cookbook-Sigismondo-Boschi/dp/1849516502)

## Ansible playbook

# Configuring Ansible Hosts

We need to set up the ```/etc/ansible/hosts``` file first before we can begin to communicate with our other computers. Open the file with root privileges like this:
```markdown
sudo nano /etc/ansible/hosts
```

The hosts can be configured in a few different ways. The syntax we are going to use though looks something like this:
```markdown
[group_name]
alias ansible_ssh_host=server_ip_address
```

The ```group_name``` is an organizational tag that lets us refer to any servers listed under it with one word. Our IP addresses are ```192.34.79.203```, ```192.34.79.187```, and ```192.34.79.15```. We will set this up so that we can refer to these individually as ```rabbit1```, ```rabbit2```, and ```rabbit3```, or as a group as ```rabbits```.

This is the block that we should add to our hosts file to accomplish this:
```markdown
[rabbits]
rabbit1 ansible_ssh_host=192.34.79.203
rabbit2 ansible_ssh_host=192.34.79.187
rabbit3 ansible_ssh_host=192.34.79.15
```

Ping all of the servers you configured by typing:
```markdown
ansible -m ping all
```
![](Ping.png?raw=true)

Finally, Here is the entire ```deploy.yml``` file that ```installs docker-ce```, ```Creates a Docker container on each instance```, ```Installs rabbitmq-server from official RabbitMQ``` and ```Setup the RabbitMQ cluster```. Ansible is a great framework to orchestrate this. The entire ```php.yml``` file should now look like this:

```markdown
---
- hosts: rabbits
  become: yes

  tasks:
  
  - name: Uninstall Old Versions
    apt:
      pkg:          "{{ item }}"
      state:        absent
    with_items:
      - docker
      - docker-engine

  - name: Install dependencies
    apt:
      pkg:          "{{ item }}"
      state:        present
      update_cache: true
    with_items:
      - apt-transport-https
      - ca-certificates
      - curl
      - software-properties-common

  - name: Add Docker’s official GPG key
    script: AddGPGkey.sh

  - name: Add Docker repository
    apt_repository:
      repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable
      state: present

  - name: Install Docker
    apt:
      pkg:          docker-ce=17.03.0~ce-0~ubuntu-xenial
      state:        present
      update_cache: yes

  - name: Install docker-py
    pip:
      name: docker-py
      state: present

  - name: Pull ubuntu image
    docker_image:
      name: ubuntu
      state: present

  - name: creating container 01
    docker_container:
      name: rabbitmq
      hostname: rabbitmq01
      image: ubuntu
      network_mode: host
      restart_policy: always
      state: started
      command: /bin/bash
      interactive: yes
      etc_hosts:
        rabbitmq01: 192.34.79.203
        rabbitmq02: 192.34.79.187
        masterRabbitmq: 192.34.79.15
    when: inventory_hostname == ansible_play_hosts[0]

  - name: creating container 02
    docker_container:
      name: rabbitmq
      hostname: rabbitmq02
      image: ubuntu
      network_mode: host
      restart_policy: always
      state: started
      command: /bin/bash
      interactive: yes
      etc_hosts:
        rabbitmq01: 192.34.79.203        
        rabbitmq02: 192.34.79.187        
        masterRabbitmq: 192.34.79.15  
    when: inventory_hostname == ansible_play_hosts[1]

  - name: creating master container
    docker_container:
      name: rabbitmq
      hostname: masterRabbitmq
      image: ubuntu
      network_mode: host
      restart_policy: always
      state: started
      command: /bin/bash
      interactive: yes
      etc_hosts:
        rabbitmq01: 192.34.79.203        
        rabbitmq02: 192.34.79.187        
        masterRabbitmq: 192.34.79.15  
    when: inventory_hostname == ansible_play_hosts[2]

  - name: updating containers
    command: docker exec -i rabbitmq bash -c "apt-get update -y"

  - name: installing rabbitmq
    command: docker exec -i rabbitmq bash -c "apt-get install rabbitmq-server -y"

  - name: Copying cookies containers 1
    command: docker exec -i rabbitmq bash -c 'echo -n "PTKAICLBSVDSKHCUZZMX" > /var/lib/rabbitmq/.erlang.cookie '

  - name: Copying cookies containers 2
    command: docker exec -i rabbitmq bash -c 'chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie'

  - name: Copying cookies containers 3
    command: docker exec -i rabbitmq bash -c 'chmod 400 /var/lib/rabbitmq/.erlang.cookie'

  - name: Copying cookies containers 3
    command: docker exec -i rabbitmq bash -c 'service rabbitmq-server start'

  - name: Conecting slaves containers to master
    command: docker exec -i rabbitmq bash -c 'rabbitmqctl stop_app; rabbitmqctl join_cluster rabbit@masterRabbitmq; rabbitmqctl start_app'
    when: inventory_hostname != ansible_play_hosts[2]

  handlers:

  - name: stop Docker
    service: name=docker state=stopped

  - name: start Docker
    service: name=docker state=started
```



