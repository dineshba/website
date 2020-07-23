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

Put this openrc file in the path (so that we can invoke this from anywhere) Eg: `/usr/local/bin/openrc`

```sh
$ cat /usr/local/bin/openrc
#!/usr/bin/env bash

gitUrl=$(git open -p)
fileContent=$gitUrl
file=".openrc"
if [ -f "$file" ]
then
	echo $gitUrl | cat $file - |rg ^http | fzf --select-1 -m | xargs -I{} open {}
else
    open $gitUrl
fi
```

> Note: Above script depends on [git-open](https://github.com/paulirish/git-open)

Have a file `.openrc` with list of urls in your repo directory.

![openrc.gif](/openrc.gif)

##### My usecase:
Want to see the result of jenkins pipeline after `git push`
So, my `.openrc` will be like

```sh
$ cat .openrc
http://myjenkins.com/job/my-repo/job/master/
```

> Note: Url should start with http:// or https:// so that `open` command opens in default browser.

So after pushed, I will do 
```sh
$ openrc # which opens my jenkins pipeline url for the master branch
```

#### Why not plain [git-open](https://github.com/paulirish/git-open) ?
- I used it a lot when my pipelines were in Gitlab. When we are having pipelines in different url other than git url, git-open is not very useful.

So started using this `script` which makes my life easily.

#### How the script evolved of the script/alias:

- Initial version:

```sh
alias openrc="cat .openrc | xargs -I{} open {}"
```

- Use grep/[ripgrep](https://github.com/BurntSushi/ripgrep) instead of cat to remove the empty files
```sh
$ alias openrc="grep . .openrc | xargs -I{} open {}"
$ # or
$ alias openrc="rg . .openrc | xargs -I{} open {}"
```

- Use [fzf](https://github.com/junegunn/fzf) to select from mutilple urls
    - Auto select if only one url present (--select-1 flag)
    - Select more than one url using <TAB> (-m flag)
```sh
$ alias openrc="grep . .openrc | fzf --select-1 -m | xargs -I{} open {}"
```

- Consider only the valid urls (starts with http(s))

```sh
$ alias openrc="grep ^http .openrc | fzf --select-1 -m | xargs -I{} open {}"
```

- Sometimes, I have to think to between typing `openrc or git open`. So removing that unnecessary thinking by below thinking 😛. Appending the output of `git open` to `.openrc` so that we can always type `openrc`
```sh
$ alias openrc="echo \$(git open -p) | cat .openrc - |rg ^http | fzf --select-1 -m | xargs -I{} open {}"
```

Note: The above script will print the error message if the `.openrc` file is not found. But still it will open the git url in the repo. To fix this, I moved from alias to script.