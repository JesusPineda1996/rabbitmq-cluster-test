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
For more details see [How to Install and Configure Ansible on an Ubuntu 12.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-an-ubuntu-12-04-vps). The string that is given back to you is what you need to paste in the second field in the Ormuco control panel:

![](KeyPanel.png?raw=true)

Click "Create SSH Key" to add your key to the control panel. 

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/JesusPineda1996/rabbitmq-cluster-test/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
