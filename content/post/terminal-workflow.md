---
title: "Terminal Workflow"
date: 2021-03-06T09:44:56+01:00
draft: false
featured: true
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

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e4/Tmux_logo.svg/1280px-Tmux_logo.svg.png" width="400">

This is a post about my workflow when editing files in the terminal.

When editing files quickly I usually resort to vim.
It requires a lot of muscle memory but hey, one needs to train their reptile brain, right.


## Tmux config

So I use tmux and vim so you need those two installed.
My tmux config is really simple. I changed the leader key to ctrl+q for easy reach on my keyboard.
I used to have ctrl+a but that conflicted with my habit to use that keyboard combo to go to the beginning of a command.

Create or edit ~/.tmux.conf and add this:

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

<img src="/termimal1.png" width="800">

Then in the upper half of the screen I open the files I want to work with

```
vim -O file1 file2
```

Then you will end up with something like this:

<img src="/termimal2.png" width="800">

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


And some other useful navigation commands. It might take some practice because you need to memorize two sets of commands.
But the reward is big, I promise.

| Command    | What it does    |
| :------------- | :----------: | 
| u | Undo |
| ctrl+r | Redo |
| ctrl+ww | Navigate to buffer  |
| :vsplit | Split vertically  |
| ctrl + w _ | Set height of split to max  |
| ctrl + w \| | Set width of split to max  |

Here some tmux commands:

| ctrl+q arrows | Navigate windows (tmux) |
| ctrl+q % | Split vertically (tmux) |
| ctrl+q " | Split horizontally (tmux) |
| ctrl+q z | Toggle window full screen (nice one!) |

Finally some handy vim productivity boosters:

| Command    | What it does    |
| :------------- | :----------: | 
| v | Enter visual mode per character |
| V | Enter visual mode per line |
| ZZ | Write file, if modified, and quit Vim |
| ( | jumps to the previous sentence |
| ) | jumps to the next sentence |
| { | jumps to the previous paragraph |
| } | jumps to the next paragraph |
| [[ | jumps to the previous section |
| ]] | jumps to the next section |
| [] | jump to the end of the previous section |
| ][ | jump to the end of the next section |
| a | Insert text after the cursor |
| A | Insert text at the end of the line |
| i | Insert text before the cursor |
| o | Begin a new line below the cursor |
| O | Begin a new line above the cursor |
| :%s/wrong/right/gc | Find and replace |
| /foo | Search and highlight foo |
| :noh | Stop highlihghting foo |
| :r | Replace current character |
| :R | Replace current character and stay in insert mode |

My favorites

| Command    | What it does    |
| :------------- | :----------: | 
| ci " | Change text between quotes works with {, [ and so on |
| . | Repeat | 
