serve
=====

`serve` is a small set of bash scripts which is used for easy deployment of a node server. It uses `ssh`, `git` and `n` to achieve this.

**This is highly experimental at best and we take no responsibility if you use this.**

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


## License 

(The MIT License)

Copyright (c) 2012 Robert Sk&ouml;ld &lt;robert@publicclass.se&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
