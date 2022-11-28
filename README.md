# [GEUS G&K technical documentation and guides](https://geus-glaciology-and-climate.github.io)

[![LICENSE](https://img.shields.io/badge/license-MIT-lightgrey.svg)](https://raw.githubusercontent.com/mmistakes/minimal-mistakes/master/LICENSE)
[![Jekyll](https://img.shields.io/badge/jekyll-%3E%3D%203.7-blue.svg)](https://jekyllrb.com/)
[![Ruby gem](https://img.shields.io/gem/v/minimal-mistakes-jekyll.svg)](https://rubygems.org/gems/minimal-mistakes-jekyll)

This is a space for sharing technical documentation and guides that are of interest to GEUS Glaciology and Climate.

It builds on the [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) Jekyll theme for building personal sites, blogs, and portfolios. As the name implies, styling is purposely minimalistic to be enhanced and customized.


## How to contribute

### Make a new branch of this repository
Navigate the dropdown menu on the Github repo and create a new branch by writing the name you want it to have followed by clicking "Create branch". 

If you would rather create a new branch through the command line then you can do so like this:

```
git checkout -b my-branch
```

### Write your post
You can either do this online through Github's code editor or on your local computer. To create a new file and edit it online, navigate to `_posts` in your branch of this repo and click "Add file". To do this locally, clone this repo, checkout your branch and then navigate and to `_posts` and create a new file.

```
git clone https://github.com/GEUS-Glaciology-and-Climate/geus-glaciology-and-climate.github.io.git
cd geus-glaciology-and-climate
git pull
git checkout my-branch
cd _posts
touch 2022-10-28-my-new-post.md
```

Name your file with the following convention `YYYY-MM-DD-title-of-post`, where you define the date as the date you intend to publish your post and the title as a hyphenated short version of your post title. 

You will be writing your post as a markdown file (.md). If you are unfamiliar with the markdown format then it is pretty easy to pick up. [This page](https://www.markdownguide.org/basic-syntax/) is a great resource for helping with the syntax. 


#### Adding a header section to your post
Once you have written your post, you need to add a header section to the top of your markdown file so that it is properly formatted and presented. It should look something like this:

```
---
title: "Linking a GitHub repo to the GEUS Dataverse"
author: Penelope How
date: 2022-11-28 16:00
classes: wide
categories:
  - Guides
tags: 
  - dataverse
  - github
---
```

You can use this one as a template, editing all of the information after each colon to fit your post. We currently support two `category` types:
- `Guides`
- `Documentation`
If your post does not fall under either of these categories then feel free to make up your own. 

Please limit `tags` to a maximum of five words, otherwise the tags and descriptions can get pretty lengthy. 

In order to add yourself as a recognised `author` (with an author profile associated with the post) then please see the next section. If you do not want to be recognised as an author then comment out the `author` field with a `#` at the beginning of the line, i.e.

```
# author: Penelope How
```

#### Adding yourself as an author
To add yourself as a recognised `author`, you need to add your information to `_data/authors.yml` in the repo. It should look something like this:

```
Penelope How:
  name: "Penelope How"
  bio: "Data scientist"
  avatar: "https://avatars0.githubusercontent.com/u/28731816?s=460&v=4"
  links:
    - label: "Email"
      icon: "fas fa-fw fa-envelope-square"
      url: "mailto:pho@geus.dk"
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/PennyHow"
    - label: "Twitter"
      icon: "fab fa-fw fa-twitter-square"
      url: "https://twitter.com/DrPennyHow"
```

Add a line break below the last author and edit this information with your author details (the header, `name`, `bio`, `avatar`, `email url` `github url`, `twitter url`). If you do not have all of these details, or don't want to disclose them, then remove the relevant fields. 

To use your Github avatar, you can easily generate the URL [here](https://cwestblog.com/2020/02/14/hack-to-get-a-users-github-avatar/).

Once you have created your author profile once, you will never have to do it again.

#### Adding images to your post
You can add images to your post from a URL like this:

```
![GEUS logo](https://www.geus.dk/media/6708/geus_dk_logo_header.png)
```

![GEUS logo](https://www.geus.dk/media/6708/geus_dk_logo_header.png)

However, there may be instances where you would like to use an original image that is not available via URL. You can place your image in the `assets/images/` directory of the repo and then link to it like this:

```
![GEUS logo](https://raw.githubusercontent.com/GEUS-Glaciology-and-Climate/geus-glaciology-and-climate.github.io/master/assets/images/gk_logo.png)
```

![GEUS logo](https://raw.githubusercontent.com/GEUS-Glaciology-and-Climate/geus-glaciology-and-climate.github.io/master/assets/images/gk_logo.png) 


### Push the post to your branch
Once your post is finalised, you need to push this to your branch if you have been working on it from a local computer.

```
# Add all changed files
git add .

# Commit these changes with a message describing the changes
git commit -m "My new post"

# Push the changes to the branch
git push
```

If you created everything on the Github editor portal then you do not need to do this step.


### Create a pull request
Once your post and files are on your branch of the repo, make a pull request to merge your branch with the master branch. This can be done on the Github portal by clicking "Pull requests" and then "New pull request" on the repo page. As this is a forked repo, you will see many options for the location of the base repository. Make sure you select `GEUS-Glaciology-and-Climate/geus-glaciology-and-climate-github.io:master`.

Or you can do this from command line as follows:

```
git pull GEUS-Glaciology-and-Climate/geus-glaciology-and-climate-github.io:master myBranch
```

#### Wait for the changes to be implemented
Once your pull request has been accepted and your post (and other file changes) has been added to the master branch, you need to wait for Github Actions to build and deploy the new pages. This can take up to 10 minutes or so. You can check [here](https://github.com/GEUS-Glaciology-and-Climate/geus-glaciology-and-climate.github.io/actions) to see how the deployment is going.
