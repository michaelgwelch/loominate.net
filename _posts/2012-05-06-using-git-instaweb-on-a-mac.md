---
title: Using git instaweb on a Mac
author: Michael
layout: post
permalink: /2012/05/06/using-git-instaweb-on-a-mac/
categories:
  - Mac
  - software development
tags:
  - brew
  - git
---

<!-- cSpell:ignore instaweb lighttpd httpd symlinks pcre unic openssl sbin aligncenter -->

Today I discovered `git instaweb`. This is a very cool feature that allows you to instantly view your local repository (the one in your current working directory) in a web browser. <!--more--> It was incredibly easy to setup on my Mac. I did some googling and found out not everyone thinks it is so easy to setup. This is because they either don't have a web server installed or the web server that is installed (apache2) isn't installed on a Mac the way `git instaweb` expects.  

As you will see shortly, I was in good shape to use this feature because I was already using the [Homebrew Package Manager][1].

Here is what happened the first time I tried using it:

```bash
$ git instaweb
lighttpd not found. Install lighttpd or use --httpd to specify another httpd daemon.
```

Like I said I use homebrew, so whenever I see an error like this the first thing I try doing is installing the necessary software using brew:

```bash
$ brew install lighttpd
Error: You must 'brew link pkg-config' before lighttpd can be installed
```

Ok, I thought, this might be a problem, but let's just do what it is asking me to do:

```bash
$ brew link pkg-config
Linking /usr/local/Cellar/pkg-config/0.25... 2 symlinks created
```

and now let's try again:

```bash
$ brew install lighttpd
==> Installing lighttpd dependency: pcre
==> Downloading ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.30.
######################################################################## 100.0%
######################################################################## 100.0%==> ./configure --prefix=/usr/local/Cellar/pcre/8.30 --enable-utf8 --enable-unic
==> make test
==> make install
/usr/local/Cellar/pcre/8.30: 130 files, 3.2M, built in 34 seconds
==> Installing lighttpd
==> Downloading http://download.lighttpd.net/lighttpd/releases-1.4.x/lighttpd-1.
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/lighttpd/1.4.30 --with-openssl --with
==> make install
Warning: /usr/local/sbin is not in your PATH
You can amend this by altering your ~/.bashrc file
==> Summary
/usr/local/Cellar/lighttpd/1.4.30: 40 files, 852K, built in 28 seconds
```

That was it. The next time I ran `git instaweb` it started up the web server and a browser and I was viewing my code repository.

![git instaweb screenshot][2]

 [1]: http://mxcl.github.com/homebrew/ "Homebrew Package Manager"
 [2]: bin/git-instaweb-screenshot.png "Screenshot of my local repository"