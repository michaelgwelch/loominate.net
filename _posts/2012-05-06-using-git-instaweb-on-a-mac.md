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
Today I discovered `git instaweb`. This is a very cool feature that allows you to instantly view your local repository (the one in your current working directory) in a web browser. It was incredibly easy to setup on my Mac. I did some googling and found out not everyone thinks it is so easy to setup. This is because they either don't have a web server installed or the web server that is installed (apache2) isn't installed on a Mac the way `git instaweb` expects.  
<!--more-->

As you will see shortly, I was in good shape to use this feature because I was already using the [Homebrew Package Manager][1].

Here is what happened the first time I tried using it:  
`<br />
$ git instaweb<br />
lighttpd not found. Install lighttpd or use --httpd to specify another httpd daemon.<br />
`

Like I said I use homebrew, so whenever I see an error like this the first thing I try doing is installing the necessary software using brew:  
``<br />
$ brew install lighttpd<br />
Error: You must `brew link pkg-config' before lighttpd can be installed<br />
``  
Ok, I thought, this might be a problem, but let's just do what it is asking me to do:  
`<br />
$ brew link pkg-config<br />
Linking /usr/local/Cellar/pkg-config/0.25... 2 symlinks created<br />
`  
and now let's try again:  
`<br />
$ brew install lighttpd<br />
==> Installing lighttpd dependency: pcre<br />
==> Downloading ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.30.<br />
######################################################################## 100.0%<br />
######################################################################## 100.0%==> ./configure --prefix=/usr/local/Cellar/pcre/8.30 --enable-utf8 --enable-unic<br />
==> make test<br />
==> make install<br />
/usr/local/Cellar/pcre/8.30: 130 files, 3.2M, built in 34 seconds<br />
==> Installing lighttpd<br />
==> Downloading http://download.lighttpd.net/lighttpd/releases-1.4.x/lighttpd-1.<br />
######################################################################## 100.0%<br />
==> ./configure --prefix=/usr/local/Cellar/lighttpd/1.4.30 --with-openssl --with<br />
==> make install<br />
Warning: /usr/local/sbin is not in your PATH<br />
You can amend this by altering your ~/.bashrc file<br />
==> Summary<br />
/usr/local/Cellar/lighttpd/1.4.30: 40 files, 852K, built in 28 seconds<br />
`

That was it. The next time I ran `git instaweb` it started up the web server and a browser and I was viewing my code repository.

<div id="attachment_452" style="width: 310px" class="wp-caption aligncenter">
  <a href="http://loominate.net/wp-content/uploads/2012/05/git-instaweb-screenshot1.png"><img src="http://loominate.net/wp-content/uploads/2012/05/git-instaweb-screenshot1-300x194.png" alt="" title="Screenshot of my local repository" width="300" height="194" class="size-medium wp-image-452" /></a>
  
  <p class="wp-caption-text">
    Screenshot of my local respository
  </p>
</div>

 [1]: http://mxcl.github.com/homebrew/ "Homebrew Package Manager"
