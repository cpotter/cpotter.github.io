---
title: Ghost Render Blog
published_at: 2015-12-28 1:20 PM
author: Chris Potter
image: /2015/12/28/images/blog-image.png
---
This blog is an attempt to keep track of and organize a variety of my personal
projects.  I'm hoping that by starting this blog it will encourage me to finish
what I actually start!  This blog is the first on my list.

A few of the things on my list:
 - [Ghost/Github Blog](https://github.com/cpotter/cpotter.github.io)
 - Deck of Cards API vanilla node app
 - Record Cataloger Meteor App
 - Meteor 1000 Books by Kindergarten Meteor App
 - Sudoku Solver
 - Freeflow Solver
 - Crafty Spending App with [Charlestronauts](https://github.com/Charlestronauts)

## How was this blog built?
This blog was built using [ghost-render](https://github.com/mixu/ghost-render).  
Using github's pages to render the final blog, locally I am writing these posts
in markdown, which is ideal.  It was rather tricky to set up however.  There were
a couple unintuitive things I needed to do in order to get the repo working perfectly.
I will try and review the process I went through.  Some steps may be basic, but
it will be best to be thorough for all audiences. (that's being optimistic!)

### Step 1: Create username repo
Since I wanted to host the blog with github, this step was the easiest.  Creating
a repo called `username`.github.io will automatically created a hosted static website
to display your website.

### Step 2: Setting up the local environment
```
git clone <username.github.io repo>
cd <username.github.io repo>
```
Once inside this repo follow the instructions for [ghost-render](https://github.com/mixu/ghost-render).
I decided to do the entire process within a folder called chris_potter_blog/ in the repo.  
I will recount them here for good measure.

```
npm install -g ghost-render
ghost-render --init > settings.json
```

#### Themes
I decided the theme(s) should be organized into their own folder for cleanliness.
I decided to try and jump between themes for testing.  Each theme I decided to fork
into my personal repo so that I could make changes and commit those changes to my
own theme.

#### Build process
I decided to write my own bash script to compile my blog. Creating the script is
simple:

```
touch compile
chmod +x compile
```
within that file I added the following code:
```
#!/bin/bash

ghost-render --input ./blog/ --settings ./settings.json --theme ./themes/Swaggy-Bastard --output ./rendering
```
running `sh compile` within the chris_potter_blog folder will create the rendering/
folder with the contents of the blog.  Once this is complete, from the base folder
of the repo, run
```
ln -s chris_potter_blog/rendering/* .
touch .nojekyll
```

I decided to symlink the information in the rendering folder to try and keep the
repo relatively clean and tidy for the build process.  If you feel this is overkill,
then don't do it!

The final issue lies with the themes repos declared under the themes folder. Git
considers these sub-repos of the blog.  So I needed to `touch .gitmodules` and within
that file have:
```
[submodule "chris_potter_blog/themes/Swaggy-Bastard"]
    path = chris_potter_blog/themes/Swaggy-Bastard
    url = https://github.com/cpotter/Swaggy-Bastard.git
```
