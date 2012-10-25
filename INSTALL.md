How to prepare a server for `serve`
==================================

This is a step-by-step guide on how to prepare a server to use the scripts in this repository for managing a node service with ease.

It has been developed on a Linode 512 VPS loaded with Ubuntu 12.04 LTS. 

## Install SSH, curl, git & nginx

    root> apt-get install openssh-server curl git-core nginx

## Install Node JS
    
    root> apt-get install python-software-properties
    root> apt-add-repository ppa:chris-lea/node.js
    root> apt-get update
    root> apt-get install nodejs npm make g++
    root> npm install -g n semver

## Add deploy user
    
    root> adduser --home /var/app deploy

## Install serve commands

    deploy> git clone https://github.com/publicclass/serve.git local \
        && chmod +x local/bin/*

## Prepare for N
    
    deploy> mkdir -p local/{lib,include}


## Configure Ubuntu 12.04 LTS

### A sudoers file for restarting node services

Copy it into `/etc/sudoers.d/` (linking does not work?):

    root> cp /var/app/local/sudoers.d-node /etc/sudoers.d/node
    root> chmod 0440 /etc/sudoers.d/node

### Configure SSH 

Don't forget to upload SSH key first.

    PermitRootLogin without-password
    PasswordAuthentication no
    AllowAgentForwarding yes

### An nginx app conf

Add this line to `/etc/nginx/nginx.conf`:

    include /etc/nginx/sites-enabled/*; // <-- after this line
    include /var/app/*/server.conf;


### An upstart app conf

Link it to upstart:

    root> ln -s /var/app/local/node.conf /etc/init/node.conf

