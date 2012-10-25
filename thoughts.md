Deployment of node
==================

This was the step by step thoughts on why and how of the bash scripts in this repo.

## Using NPM

The idea is that the server has npm installed and set up to use the same private repo. Then to setup a new website it's added using `npm i $NAME` on the server. 

  1. Publish to private NPM registry

      npm publish

  2. Deploy on server

      ssh -A deploy@server "npm up $NAME"


## Using deploy.sh + foreman

  1. Add Procfile

      # Procfile
      web: n npm 0.6.18 start
      redis: redis-server
      es: elasticsearch
      couch: couchdb

  2. Add deploy.conf

      [production]
      user deploy
      repo git@publicclass.org:publicclass.git
      path /var/node/publicclass
      ref origin/master
      post-deploy npm i --production && foreman start


## Using a deploy 'service'

It's technically a set of bash scripts:

* `bin/setup <name> [hostnames]`

  Easily create a deployable repository using `ssh deploy@host bin/setup test`.

  - `create_app(name)`
    Creates the app directory using `mkdir -p /var/app/$name`. If $hostnames is set it will not fail if the directory already exists.

  - `create_repo(name)`
    Creates a bare repository using `git init --bare /var/app/$name.git`. And copies in bin/post-receive into repo/.git/hooks. Skip this if repo already exists.

  - `create_routes(hostnames)`
    Will add routes for the app into /var/app/$NAME/routes.conf. 

      # hostnames.conf
      example.com
      test.example.com
      www.example.com

    the proxy should then on reroute() assign a port to these hostnames




* `bin/post-receive` 
  
  Is symlinked/copied to the a repository which wants to be auto-deployed.

  - `main()`

      while read oldrev newrev refname
      do
        old=`git rev-parse --short $oldrev`
        new=`git rev-parse --short $newrev`
        checkout $old $new $refname
      done

  - `checkout(old,new,ref)`
    Simply clones the repository into /tmp/{name}/{ref}, cd into it and checks out `ref`. 

      GIT_WORK_TREE=/tmp/$name/$new git checkout -f $new

  - `node_version()`
    Tries to figure out the node version of the repository by reading the `package.json` or `.node_version` and running it through `semver` along with a list of the possible node versions. And call `n $version` to make sure it's installed.

      test $node_version || node_version=`cat .node_version 2> /dev/null`
      test $node_version || node_version=`node -p -e 'require("./package.json").engines.node' 2> /dev/null`
      test $node_version || err could not find node version
      /usr/local/bin/n $node_version
      return $?

  - `create_shims(node_version)`
    Creates `bin/node` and `bin/npm` in the app repository which points to the correct node version. Something like this should work:
      
      echo '#!/bin/bash' > bin/node
      echo "`which n` use $node_version \$@" >> bin/node

      echo '#!/bin/bash' > bin/npm
      echo "`which n` npm $node_version \$@" >> bin/npm

  - `install_modules()`
    Installs modules from npm.

      bin/npm install --production

  - `build_archive(name,ref)`
    Creates an archive of the repo into /var/app/{name}/{ref}.tar.gz

      tar -czf /var/app/{name}/{ref}.tar.gz --exclude .git/ .

  - `cleanup()`
    Remove the temporary directory. Also run in case of any errors from the earlier steps.

  - `deploy()`
    Configurable? Executes `bin/deploy`. 


* `bin/deploy <name> <ref>`
  - `extract_archive(name,ref)`
    Extracts the archive 

      tar -xzf /var/app/$name/$ref.tar.gz

  - `next_port()`
    Finds the next available port.

  - `start(name,ref,port)`
    Starts the app 

      start node NAME=$name REF=$ref PORT=$port

  - `test(port)`
    See if the app is running with 

      curl -s localhost:$port

  - `reroute(port)`
    Updates /var/app/routes.json. But where do we get the hosts for the app?




Compiling to a package on git deploy which includes node/npm and is then 

1. (l) git push deploy
2. (r) post-receive hook
  2.1 git checkout the ref
  2.2 find node-version in package.json and create node and npm shims for it in bin/
  2.3 bin/npm install --production
  2.4 compile an archive with the modules and all to /var/app/{name}/{ref}.tar.gz (+ checksum?)
  2.6 a deployment "service" watches /var/app/*/*.tar.gz and takes that archive and untars it to /var/app/{name}/{ref} 
      and run `start node NAME={name} REF={ref} PORT={port}` where port is the next available port (see below).
  2.7 if it starts and responds without fail it will update it's route in /var/app/routes.json and `restart node-router`
  2.8 otherwise it will 

