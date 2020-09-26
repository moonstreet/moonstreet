---
title: "Moonstreet"
date: 2020-09-19T08:36:08+02:00
draft: false
---

Welcome to yet another Hugo blog!
<!--more-->

Moonstreet is the name of the street where I live, together with my lovely girlfriend.
I am a devops engineer by day and a Linux distro hopper by night. By the time coding or scripting results in configuration I usually get bored.

## About this website

These are my notes about the things I learn and think I worthy of sharing with the world. 
I am writing these notes on a modern Linux distro or my MacBook which I use as a daily driver so there will be hardly any Windows content (but never say never).
This website is made with the following tools:

- [Hugo](https://gohugo.io/), a static site generator
- [Github](https://github.com), for version control
- [Netlify](https://www.netlify.com/), for deployment
- [Noteworthy](https://github.com/kimcc/hugo-theme-noteworthy), the theme. 

Here are the steps to create this site:

```sh
sudo dnf install -y hugo
mkdir ~/blog && cd ~/blog
hugo new site moonstreet
cd themes
git init
git clone https://github.com/kimcc/hugo-theme-noteworthy.git
cd ..
echo 'theme = "hugo-theme-noteworthy"' >> config.toml
hugo serve -D
```

Then in a new terminal window:

```sh
hugo new posts/hugo.md
```

And then paste this content in hugo.md.

## Push to Github and publish with Netlify

When publishing, make sure your post is not set to 'draft'.
Run a final `hugo` to build the site.
Then commit and push.