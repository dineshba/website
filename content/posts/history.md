---
title: "Terminal: History Matters"
date: 2020-07-12T13:32:13+05:30
---

For more productive, it is better to reuse the commands in history. In order to do that

- Save more history
- Optimize the history
- Able to find and reuse history easily

##### Save more history

```sh
# bashrc/zshrc/etc

#### increase history size
export HISTSIZE=1000000
export HISTFILESIZE=1000000
####

```

> Note: You can specify how big you want

##### Optimize the history

- Remove the duplicates in the histroy file
- Keeps the last entry when there is a duplicate (to maintain the order)

I wrote a simple scirpt in [nim programming language](https://nim-lang.org/) for the bash_history file. You can find the script [here](https://github.com/thecasualcoder/prunehistory). If you are using other than bash, feel free to raise PR for other shells.

> Note: For oh-my-zsh refer the comments section below which contains the helpful links to achieve the same. Thanks to Prabhu for the comments

##### Able to find and reuse history easily

Using tools like fzf to do fuzzy search and find the older commands and use. [Refer my another blog](https://dineshba.github.io/posts/fzf/) to see the easy usage of fzf.