serve
=====

`serve` is a small set of bash scripts which is used for easy deployment of a node server. It uses _ssh_, _git_ and _n_ to achieve this.

## Example
 
Setting up and deploying to an app named "bogey" which will be hosted at _bogey.com_. Assumes we have a git repository already for a functioning node project with a healthy `package.json`.

```
$ ssh deploy@server serve setup bogey bogey.com

  Your app "bogey" has been set up! add it as a git remote with:
    
    git remote add deploy deploy@1.2.3.4:bogey.git

$ git remote add deploy deploy@server:bogey.git
$ git push deploy master
Counting objects: 5, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 509 bytes, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: -----> Installing node 0.6.21
remote: -----> Installing dependencies with npm 1.1.37
remote:        Dependencies installed
remote: -----> Building archive
remote: -----> Deploying bogey (d024f28)
remote:        Successfully deployed on port 5000
To deploy@server:bogey.git
   02dc119..d024f28  master -> master

```

