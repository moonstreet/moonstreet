---
title: "Over about"
aliases: ["about-us", "about-hugo", "contact"]
date: 2020-12-28T17:10:13+01:00
draft: false
showShare: false
---

Welcome to yet another Hugo blog!
<!--more-->

Moonstreet is the name of the street where I live, together with my lovely girlfriend.
I am a devops engineer by day that just can't stop thinking about technology in the night. 

The opinions expressed in this blog are my own personal opinions and do not represent my employer’s view in any way.

## About this website

These are my notes about the things I learn and think I worthy of sharing with the world.
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

## On a new computer

```sh
git clone # this repo
cd moonstreet
git submodule update --init --recursive
hugo server -D
```

## Some markdown pointers

On how to include and resize images.

```markdown
![tmux](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e4/Tmux_logo.svg/1280px-Tmux_logo.svg.png)
 
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e4/Tmux_logo.svg/1280px-Tmux_logo.svg.png" width="400">

<img src="/termimal2.png" width="600">

![plaatje](/termimal2.png)
```

Written in Go, Hugo is an open source static site generator available under the [Apache Licence 2.0.](https://github.com/gohugoio/hugo/blob/master/LICENSE) Hugo supports TOML, YAML and JSON data file types, Markdown and HTML content files and uses shortcodes to add rich content. Other notable features are taxonomies, multilingual mode, image processing, custom output formats, HTML/CSS/JS minification and support for Sass SCSS workflows.

Hugo makes use of a variety of open source projects including:

* https://github.com/yuin/goldmark
* https://github.com/alecthomas/chroma
* https://github.com/muesli/smartcrop
* https://github.com/spf13/cobra
* https://github.com/spf13/viper

Hugo is ideal for blogs, corporate websites, creative portfolios, online magazines, single page applications or even a website with thousands of pages.

Hugo is for people who want to hand code their own website without worrying about setting up complicated runtimes, dependencies and databases.

Websites built with Hugo are extremelly fast, secure and can be deployed anywhere including, AWS, GitHub Pages, Heroku, Netlify and any other hosting provider.

Learn more and contribute on [GitHub](https://github.com/gohugoio).
