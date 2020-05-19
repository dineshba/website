---
title: "Openrc"
date: 2020-05-19T21:31:36+05:30
---

#### Problem Statement:

After doing `git push` how many of us wanted to see the
    
- status of the pipeline
- new commits in github/gitlab ui
- new artifacts in the nexus repo
- wiki page of the git repo


To be generic, you want to open a particular url after an action. How can we achieve it easily ?

#### Solution:

```sh
alias openrc="cat .openrc | xargs -I{} open {}" # in your .bashrc/.zshrc file
```

Have a file `.openrc` with list of urls in your repo directory.

##### My usecase:
Want to see the result of jenkins pipeline after `git push`
So, my `.openrc` will be like

```sh
$ cat .openrc
http://myjenkins.com/job/my-repo/job/master/
```

So after pushed, I will do 
```sh
$ openrc # which opens my jenkins pipeline url for the master branch
```

#### Why not [git-open](https://github.com/paulirish/git-open) ?
- I used it a lot when my pipeline is in Gitlab. When we are having pipeline in different url other than git url, git-open is not very useful.

So started using this `alias` which makes my life easily.