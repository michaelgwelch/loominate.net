---
title: Doing Code Reviews with GitHub

layout: post
permalink: /2012/04/28/doing-code-reviews-with-github/
categories:
  - software development
tags:
  - git github
---
For many reasons, I really enjoy using git and GitHub for my current project. Today I'll highlight one reason: code reviews.

<!--more-->

Like many others I like that I can implement a code review process and GitHub ensures that it is always followed.

Only two team members have write access to *upstream*. Everyone else works with their own fork and must submit a pull request when their work on some item is complete.

GitHub provides a pretty nice diff view (I hope they add side-by-side like Bitbucket soon) of the pull request and also allows me to comment directly on any line in the diff. However, that isn't a good enough code review (most of the time) for me.

My process requires that I do the following (in addition to checking that, you know, the code looks good):

  * Check that the changes compile
  * Check that there are no warnings (compiler, FxCop, or ReSharper)
  * Manually inspect for "green" code files (ReSharper again)
  * Check the unit tests pass
  * Run the code and do a cursory test to make sure everything still works.

(We are currently using TeamCity, but this is a new tool for me and therefore I'm not used to letting it do most of this work for me. It's currently doing continuous builds and running the units test. I'm just not used to checking it for results. Eventually, I may just check the logs to see that most of this is in place.)

Of course git and GitHub allow me to do this easily. I've added a remote for each member of my team. Assuming I need a remote for a developer named 'William' I just make sure that I've create a remote at some point

    $ git remote add bill https://github.com/bill/project.git


Then when he creates a pull request I view it on GitHub. I can see the description of the pull request. I can view the commits. I can view the diff for each individual commit (if there is more than one, hopefully not), or a diff of all the commits together. But in addition I can grab the changes:

    $ git fetch bill


We use remote branches (rather than master) for new work. This allows team members to have more than one outstanding pull request (if necessary&#8212;perhaps they have a new task and one or more bugs that they've fixed). So when I inspect the branches for bill I may see something like the following:

    $ git branch -r | grep bill
    bill/fixBug17
    bill/fixBug19
    bill/impelementFeature3


I know which branch to look at by looking at the pull request information on GitHub. Assume it is fixBug17.

I have several options for seeing this code. Here's what I normally do now.

Make sure that *master* is clean and up to date. Then I do

    $ git checkout -b mergeFixBug17 // creates a new branch based off of the latest to merge the changes into
    $ git merge bill/fixBug17       // merges bill's changes into my branch


Then I open up Visual Studio and do my inspection. How do I log issues? Well there are many ways to log issues and I'm still playing around to figure out the *best* way.

  1. The first option is to simply log every issue on the pull request form on GitHub. This has the benefit of exposing your comment to everyone on the team to see and discuss. I find that this process can bog down and take too long. I've had times where the review turned into a "comment, fix, submit, comment again, fix again, submit again" cycle. It also slows down how long it takes to get the code into *upstream*. Especially for simple fixes and simple pull requests this can be a bit "process heavy". The key to making this work effectively is to stress the importance of small pull requests. I've informed my team (I made these numbers up recently and I'll refine along the way) that they should have 5 or fewer files with no more than 100 additions/deletions. Any exceptions should have a good reason.

  2. For quick fixes, I just make them in my branch and when I'm done I commit my changes and push the whole thing to upstream which closes the pull request. The simple/quick fixes I'm talking about are spelling mistakes in comments, whitespace issues, some types of "safe-to-fix" Resharper warnings (e.g. unused usings), etc. I'm talking about things that don't require "yet another code review". Of course, if I do this, then I'm responsible if the build breaks. So I still need to check that it compiles, runs, tests pass, etc.

  3. For simple fixes that require a little more time I add TODO items in the code, commit my changes and push to upstream. (We have a policy of having no TODO items in the code and ReSharper will find them all for you). So part of our process is to everyday, clean up any TODO items. When I do this I email the author, "Fix TODO items related to pull #108. See my commit fe4353789.".

  4. Sometimes I think a refactoring is necessary but hard to explain in a comment. In that case I'll talk to the developer or I'll just do the refactoring the way I think it should be done. Then I commit my work and push it to my fork and create a pull request for the original author. If they like my changes/suggestions they can merge my pull request into their fork, finish up any other items and then ask me to merge in their pull request. Then I fetch their latest (do the review process again) and push to upstream.

In almost all cases, I do the merge locally instead of letting GitHub do the merge. I don't like the fact that GitHub always creates a new commit for every merge, even if it was a fast-forward merge. I personally think this clutters up the commit history as you end up having "commit, merge, commit, merge" type of history. Each commit requires a separate merge commit. I avoid this by merging on my machine and pushing to *upstream*.

Oh, and did I mention this process is really fast. Most of the teams at my employer still use [Rational Synergy][1] which is about the crappiest tool you can use for this sort of thing.

I can easily swap between the branches used for multiple code reviews in 2 seconds. Let's say I've got 5 code reviews to do. I pull down all the changes and create the branches as described above. I start the first code review and add my comments. Then I simply do a

    $ git checkout branchForNextPullRequest


and now I'm looking at the code for the next code review. While the fist author responds to my comments I can be working on the next review. When the first author is finished fixing the code I can just do the following

    $ git checkout firstPullRequest // go back to the first code review
    $ git fetch bill                // fetch bill's latest code
    $ git merge bill/fixBug17       // assuming bill is still working on branch fixBug17


Now 5 seconds after starting this process I'm looking at the first code review again, with the latest changes.

Rational Synergy requires about 5 - 20 minutes depending on the network speed, the size of the repository, the size of the code-base for this project, etc. 5 minutes versus 5 seconds!!!!!!!

 [1]: http://www-01.ibm.com/software/awdtools/synergy/
