* installation

``` bash
$ git clone https://github.com/ac1965/hugo-blog.git && cd hugo-blog
$ git submodule init
$ git submodule update
$ git submodule foreach git pull origin master
```

* post
hugo new blog/hoge.md

* generation
hugo -D

* server (and generation)
hugo server (-D)
