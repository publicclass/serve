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

    deploy> git clone https://github.com/publicclass/serve.git \
        && chmod +x local/bin/*

## Prepare for N
    
    deploy> mkdir -p /var/app/local/{lib,include}


## Configure Ubuntu 12.04 LTS

### A sudoers file for restarting node services

Create a sudoers-conf for the deploy user:

    deploy> vim /var/app/local/sudoers.d-node 

        deploy     ALL=NOPASSWD: /sbin/restart node *
        deploy     ALL=NOPASSWD: /sbin/stop node *
        deploy     ALL=NOPASSWD: /sbin/start node *
        deploy     ALL=NOPASSWD: /etc/init.d/nginx reload


Then copy it into `/etc/sudoers.d/` (linking does not work?):

    root> cp /var/app/local/sudoers.d-node /etc/sudoers.d/node
    root> chmod 0440 /etc/sudoers.d/node

### Configure SSH 

Don't forget to upload SSH key first.

    PermitRootLogin without-password
    PasswordAuthentication no
    AllowAgentForwarding yes

### An nginx app conf

Create the `server.conf` which will be copied into each app directory:

    deploy> mkdir -p /var/app/local/skel
    deploy> vim /var/app/local/skel/server.conf 

        server {
          server_name {hostnames};

          location / {
            proxy_pass http://127.0.0.1:{port};
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP  $remote_addr;
          }
        }


Then add this line to `/etc/nginx/nginx.conf`:

    include /etc/nginx/sites-enabled/*; // <-- after this line
    include /var/app/*/server.conf;


### An upstart app conf

Create the `node.conf`:

    deploy> vim /var/app/local/node.conf 

        description 'node upstart script'
        author 'robert'
         
        start on (local-filesystems and net-device-up)

        instance "Node - $NAME - $REF"

        respawn 
        respawn limit 5 60

        console output

        script
          # Debugging:
          exec 2>>/dev/.initramfs/myjob.log
          set -x
          
          test "$PORT" || (echo "missing PORT" && exit 1)

          DIR="/var/app/$NAME/$REF"
          VARS=`cat /var/app/$NAME/.env 2> /dev/null | tr '\n' ' ' | sed -E 's/[^ ]+/export &;/g'` 
          VARS="$VARS export PORT=${PORT};"

          exec su -s /bin/sh -c "${VARS}"'exec "$0" "$@"' deploy \
            -- ${DIR}/bin/npm start --production ${DIR} 2>&1 \
            | /usr/bin/tee ${DIR}.log \
            | /usr/bin/logger -plocal7.notice -t${NAME}-${REF}
        end script

Then link it to upstart:

    root> ln -s /var/app/local/node.conf /etc/init/node.conf



