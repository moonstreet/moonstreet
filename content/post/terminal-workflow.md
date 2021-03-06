---
title: "Terminal Workflow"
date: 2021-03-06T09:44:56+01:00
draft: false
showShare: false
codeMaxLines: 50 
codeLineNumbers: true 
figurePositionShow: true 
categories:
- Technology 
tags:
- linux
- vim
---

This is a post about my workflow when editing files in the terminal.

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e4/Tmux_logo.svg/1280px-Tmux_logo.svg.png" width="400">


When editing files quickly I usually resort to vim.
It requires a lot of muscle memory but hey, one needs to train their reptile brain.


## Tmux config

So I use tmux and vim so you need those two installed.
My tmux config is really simple. I changed the leader key to ctrl+q for easy reach on my keyboard.
I used to have ctrl+a but that conflicted with my habit to to use that keyboard combo to go to the beginning of a command.


```sh
set-option -g prefix C-q
set -g mouse off

set -g status-bg black
set -g status-fg white
```

## Workflow


First start tmux and then split the screen horizontally.
Then decrease the size of the lower screen.

```
tmux
ctrl+q "
ctrl+q  arrow down #hold ctrl+q while pressing arrow down key
```
You will end up with something like this:

<img src="/termimal1.png" width="600">

Then in the upper half of the screen I open the files I want to work with

```
vim -O file1 file2
```

Then you will end up with something like this:

<img src="/termimal2.png" width="600">

As you can see you can edit files in the upper part of the screen and issue commands in the lower part.


## Some basic commands

If you want to open another buffer (aka file in vim) you can do:

```shell
ctrl+p #assuming you have that installed in vim, else use :e
```

Then you can just choose which buffer you want to display with
:bd, :bn and so on. If you want to delete a buffer then just type :bd

| Command    | What it does    |
| :------------- | :----------: | 
| :bn | Display next buffer   |
| :bp | Display previous buffer |
| :bd | Delete buffer |
| :ls| List buffers |


And some other useful commands. It might take some practice because you need to memorize two sets of commands.
But the reward is big, I promise.

| Command    | What it does    |
| :------------- | :----------: | 
| u | Undo |
| ctrl+r | Redo |
| ctrl+ww | Navigate to buffer  |
| ctrl+q arrows | Navigate windows (tmux) |
| :vsplit | Split vertically (vim) |
| ctrl+q % | Split vertically (tmux) |
| vspl+q " | Split horizontally (tmux |
