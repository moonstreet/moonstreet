---
title: "Finding files quickly"
date: 2024-03-29T08:44:56+01:00
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
- mouseless
---

# Finding files with fzf ripgrep and nnn

This is the next articial in the series 'mouseless'. Find previous articles here: 

[Terminal workflow]({{< ref "terminal-workflow" >}})  
[Nice vim commands]({{< ref "nice-vim-commands" >}})

Why? Because navigating through files in can take a lot of time, especially if you are faced with a big git repo. However, tools like ripgrep and fzf can assist you here. 

Let's keep it simple and easy to remember!

## nnn
[nnn](https://github.com/jarun/nnn) a full-featured terminal file manager.
Here are some commands you will need:

| Shortcut | Description                     |
|----------|---------------------------------|
| !        | Open folder in terminal         |
| e        | Open selected file in vim       |
| q (or 2x esc        | Exit nnn                        |

<img src="/nnn.png" width="400">

## fzf and ripgrep

[fzf](https://github.com/junegunn/fzf) is a tool that can help you deal with searching for files and then filter the results.
Just typing `fzf` in a folder will display all files and folders, and then you can carry on typing to narrow the results.

<img src="/fzf.png" width="400">

[ripgrep](https://github.com/BurntSushi/ripgrep) is a tool that recursively searches the current directory for a regex pattern. It is blazingly fast. 

## fzf and rg in vim

Wouldn't it be nice to edit the selected file immediately in vim?
Yes it would. Install the [fzf.vim](https://github.com/junegunn/fzf.vim) plugin. 

At this to your .zshrc or .bashrc:

```sh
function vimrg() {
  local file=$(rg --line-number --no-heading --color=always "$1" | fzf --ansi --delimiter=':' --preview 'rg --color=always --context 2 {1} {2}' --preview-window=hidden --no-height )
  if [[ -n $file ]]; then
    local file_path=$(echo "$file" | cut -d':' -f1)
    local line_number=$(echo "$file" | cut -d':' -f2)
    vim "+$line_number" "$file_path"
  fi
}

```
When running vimrg "encryption" it will start fzf to all files with 'encryption' somewhere in the text. It will open the file in vim upon selecting the entry you want to edit.

### fzf in vim
When installing fzf in vim, it will actually use ripgrep under the covers to search the folder.
It allows you to type `:Rg <searchterm>` and then it will open a nice fzf preview window.

<img src="/rg-vim.png" width="600">

## What is next

I think I should write more, but I really want to go for a run now and do some other work.

