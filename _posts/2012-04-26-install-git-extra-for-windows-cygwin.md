---
title: Install git-extras for windows cygwin
author: Michael
layout: post
permalink: /2012/04/26/install-git-extra-for-windows-cygwin/
categories:
  - software development
tags:
  - git
---
I recently discovered git-extras when searching to see if the brew package manager (for Mac) had a package for git

    > brew search git
    

<!--more-->

Once I looked at it and read the documentation on github, I decided it would be nice to have. https://github.com/visionmedia/git-extras

I also use git on my Windows machine so I followed the instructions for installation

    > curl https://raw.github.com/visionmedia/git-extras/master/bin/git-extras | INSTALL=y sh
    

It installed without a hitch. But the first time I tested a command it puked:

     $ git extras
     /usr/local/bin/git-extras: line 2: $'\r': command not found
     /usr/local/bin/git-extras: line 4: $'\r': command not found
     /usr/local/bin/git-extras: line 5: syntax error near unexpected token `$'{\r''
     'usr/local/bin/git-extras: line 5: `update() {
    

I understood it was a dos vs unix vs mac line endings issue. And I quickly realized that its because I have the following setting: `core.autocrlf=true` set. So I was stuck. I didn't want to unset that globally. And I had no idea how to clone a git repository and specify an option on the command line. So I just brute forced it. I cloned the repository:

    git clone https://github.com/visionmedia/git-extras.git 
    

Then I changed into the directory

    cd git-extras
    

Then I locally modified the autocrlf setting

    git config core.autocrlf false
    

Then when I did a `git status` it showed me that every file appeared to be modified. I then did a

    git checkout .
    

To recheck out every file. And then finally installed

    make install
    

After which all the commands worked.
