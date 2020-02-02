+++
title = "About Moonstreet"
date = "2019-01-01"
menu = "main"
+++

Moonstreet is the name of the street where I live, together with my lovely girlfriend and all sorts of laptops and computers.

These are my notes about Automation, Devops and SRE. I use the following techniques:

- [Hugo](https://gohugo.io/), a static site generator
- [Github](https://github.com), for version control
- [Netlify](https://www.netlify.com/), for deployment


## About Automation

I like to automate all the things. Cloud provisioning, configurations, application deployment, container deployment and orchestration.. In this blog I will write things down not only because I have a tendency to forget stuff but also because I like to share what I know. And learn as I go.

## About Hugo

Install Hugo from a binary.

https://gohugo.io/getting-started/installing/


```sh
wget https://github.com/gohugoio/hugo/releases/download/v0.63.2/hugo_0.63.2_Linux-64bit.deb && sudo dpkg -i hugo_*.deb
hugo new site luthorcorp
```
This will make  a new folder named luthorcorp.

Go look for a theme here: https://gohugo.io/getting-started/quick-start/

```
cd luthorcorp
git init
git submodule add https://github.com/alanorth/hugo-theme-bootstrap4-blog.git themes/hugo-theme-bootstrap4-blog
```

Change the config.toml that came with the theme with the one that is in the root of your site.
Just read the docs from the theme or else nothing gets displayed.

Then create a new post.

```
hugo new posts/my-first-post.md
hugo new posts/gardening/planting-a-tree.md
```

