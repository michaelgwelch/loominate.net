---
title: Importing Subversion Into Git
author: Michael
layout: post
permalink: /2012/05/13/importing-subversion-into-git/
categories:
  - software development
tags:
  - git
  - svn
excerpt: Learn how to import your subversion repositories (with history) into git.
---
Over the years I've accumulated some Subversion repositories. I used Subversion to control the source code I wrote for my masters courses. These repositories are sprinkled around a couple of computers I own. Now that I have accounts on GitHub and Bitbucket I decided it was time to import the old code into git and push it to the cloud. This allows me to clean up all the old repositories and easily access it from any computer.



The process was really easy. First I reminded myself what was in each repository. This was actually the &#8220;hardest&#8221; part as I haven't used Subversion in at least 4 or 5 years and couldn't remember how to list the contents of a repository. Turns out you need to use a URL even if it's a local file system repository.

The following will return me a list of directories like

```shell
$ svn list file:///Users/welch/mysvn
cs732/  
cs838/
```

These are two of the courses I took that I wanted to import into my coursework repository on GitHub.

The first thing I did is google a bit and found a couple of useful articles like this one on [stackoverflow][1]. This gave me the basics and then I just played around with it.

I created an author's file as discussed and named it users.txt with the following line:

mgwelch = Michael Welch <myemail@email.com>

Then I cloned the cs732 directory:

$ git svn clone -A users.txt &#8211;no-metadata file:///Users/welch/mysvn/cs732

Then I changed directory to my git repository named coursework.

I added the new cs732 git repository as a remote

$ git remote add cs732remote /path/to/cs732

And then did a fetch

$ git fetch cs732remote

And then rebased it onto master and merged

$ git checkout -b cs732 cs732remote/master  
$ git rebase master  
$ git checkout master  
$ git merge cs732  
$ git push 

At which point all of the commits from the old repositories were added to master. (In hindsight it would have made more sense to rebase in the other direction so that all of the old commits from cs732 came before all of the new commits that were already in my coursework repository.)

It's pretty neat that all of the original check-ins from Subversion were preserved including the dates and times (but not the timezone apparently).

 [1]: http://stackoverflow.com/questions/79165/how-to-migrate-svn-with-history-to-a-new-git-repository "Import svn into git"
