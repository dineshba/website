---
title: "Gitfetcher"
date: 2020-06-26T15:38:55+05:30
---

#### Why I will not do `git fetch` or `git pull` ?

Because I have **automated** it.

![gitfetcher.gif](/gitfetcher.gif)

Highly inspired from [bash-git-prompt](https://github.com/magicmonty/bash-git-prompt). If you are already using it (or equivalent of it), below custom script may be useful to you.

**Problem Statement:** 

- Whenever I jump into the git directory, I do `git pull`
- And sometimes I forget to do it which might create conflicts

So created a script, which will do `git fetch`

- whenever I jump into git directory
- if we havent fetched for some time (say 5 min)

```sh
$ cat /path/to/gitfetcher.sh
#!/usr/bin/env bash

function fetch_in_background() {
  {
    eval "git fetch --quiet" &> /dev/null
  }&
}

GIT_REPO="$PWD/.git"
if [[ -e $GIT_REPO ]]; then
  FETCH_HEAD="$GIT_REPO/FETCH_HEAD"
  if [[ -e "${FETCH_HEAD}" ]]; then
    
    FETCH_TIMEOUT=5 #minutes after which check again
    
    perl -e '((time - (stat("'"${FETCH_HEAD}"'"))[9]) / 60) > '"${FETCH_TIMEOUT}"' && exit(0) || exit(1)'
    x="$?"
    if [[ $x == 0 ]]; then
      if [[ -n $(git remote show) ]]; then
        (
          fetch_in_background
          disown -h
        )
      fi
    fi
  else 
    # echo ".git/FETCH_HEAD not found"
    fetch_in_background
    disown -h
  fi
fi

```

And include the above script as `prompter` in your bash/zsh/fish configfile like

```sh
### Fetch in background
export PATH="$PATH:path/to/gitfetcher.sh" # configure gitfetcher.sh in the path
export PROMPT_COMMAND="gitfetcher.sh; $PROMPT_COMMAND"
### Fetch in background
```

You can find the source code of above script [here](https://github.com/dineshba/dotfiles/blob/master/gitfetcher.sh) and [sample](https://github.com/dineshba/dotfiles/blob/master/.bashrc#L33-L36) to see how I configured in my bashrc file