---
title: "My personal vim cheatsheet"
date: 2021-03-05T09:44:56+01:00
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

Here is  my vim & tmux cheatsheet.

## tmux

This is my .tmux.conf. I prefer to set the meta key to **q**.

```sh
set-option -g prefix C-q
set -g mouse off

set -g status-bg black
set -g status-fg white
```

| Command    | What it does    |
| :------------- | :----------: | 
| ctrl+q arrows | Navigate windows (tmux) |
| ctrl+q % | Split vertically (tmux) |
| ctrl+q " | Split horizontally (tmux) |
| ctrl+q z | Toggle window full screen (nice one!) |
| [ | Enter scroll mode |
| q | Leave scroll mode |


 ## Open multiple files at once

```shell
vim file1 file2
vim -O file1 file2 # split vertically
```

## Buffers

| Command    | What it does    |
| :------------- | :----------: | 
| :bn | Display next buffer   |
| :bp | Display previous buffer |
| :bd | Delete buffer |
| :ls| List buffers |


## Windows


| Command    | What it does    |
| :------------- | :----------: | 
| u | Undo |
| ctrl+r | Redo |
| ctrl+ww | Navigate to buffer  |
| :vsplit | Split vertically  |
| ctrl + w _ | Set height of split to max  |
| ctrl + w \| | Set width of split to max  |

## Productivity boosters

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
| Go | Add a new line at the end of the file |
| :%s/wrong/right/gc | Find and replace |
| /foo | Search and highlight foo |
| :noh | Stop highlihghting foo |
| :r | Replace current character |
| :R | Replace current character and stay in insert mode |

## Favorites

| Command    | What it does    |
| :------------- | :----------: | 
| ci " | Change text between quotes works with {, [ and so on |
| . | Repeat | 
