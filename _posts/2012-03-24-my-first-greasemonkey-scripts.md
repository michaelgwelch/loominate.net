---
title: My First Greasemonkey Script

layout: post
permalink: /2012/03/24/my-first-greasemonkey-scripts/
categories:
  - software development
tags:
  - commit
  - github
  - Greasemonkey
  - javascript
  - pivotal tracker
  - scripting
  - story
---
# Link Github Commit to Pivotal Tracker Story

I'm using Github and Pivotal Tracker. You can easily [configure][1] Github to notify Pivotal Tracker of commits. Once you have done that, then whenever you want to commit something that is related to a story you simply embed the tracker story id in your commit message. Something like:

 [1]: https://www.pivotaltracker.com/help/api?version=v3#github_hooks "Configure post-commit hooks on Github"

<!--more-->

<pre>> git commit -m "[#87654321] Finish the new story"</pre>

It's necessary that you put it in square brackets and use the pound sign. If you do this, then a comment will be added to your Pivotal Tracker story with a url that links back to the commit. This is cool.

However, this doesn't generate any link from Github to Pivotal Tracker. Sure you could either a) copy the story id and switch over to Pivotal Tracker and do a search or b) embed the URL every time you do a commit.

But there is something easier you can do. You can install my new Greasemonkey script. (This works in Firefox if you have Greasemonkey installed, and it works in Chrome natively.)

For easy install you can go to http://userscripts.org to install my script.

[Link a commit to its tracker story][2]

The code is stored in a gist at Github (Note, this gist may have been updated since this post was written. Click thru to see the latest):


<noscript>
  <pre><code class="language-javascript javascript">// --------------------------------------------------------------------
//
// ==UserScript==
// @name           Link to Story 2
// @namespace      loominate.net
// @version        1.4
// @description    Used on github.com to convert a story id to a link to the pivotal tracker story.
// @include        https://github.com/*/commits/*
// @include        https://github.com/*/commits
// @include        https://github.com/*/commit/*
// @include        https://github.com/*/pull/*
// ==/UserScript==
"use strict";
var regex = /\[[^\]]*#([0-9]{8}).*\]/;

function alwaysTrue(x) { return true; }

function createLinksToStories(className, elementFilter)
{
    if (!elementFilter) {
        elementFilter = alwaysTrue;
    }

    var elements = document.getElementsByClassName(className);
    for (var index in elements) {
        var element = elements[index];
        if (elementFilter(element)) {
            var matches = regex.exec(element.innerHTML);
            if (matches) {
                element.innerHTML = element.innerHTML + "<a href='https://www.pivotaltracker.com/story/show/" + matches[1] + "'><img src='https://www.pivotaltracker.com/favicon.ico'/></a>"
            }
        }
    }
}


var pullRequestRegex = new RegExp("https://github.com/.*/.*/pull/[0-9]*");
function isPullRequestPage() {
    return pullRequestRegex.test(document.location);
}

function isAnchor(element) {
    return (element.localName === "a");
}

createLinksToStories('commit-title');
createLinksToStories('content-title');

if (isPullRequestPage()) {
    createLinksToStories('message', isAnchor);
}


</code></pre>
</noscript>

Here's a screenshot of my commits in my mbasic99 project. The most recent commit was related to a story on Pivotal Tracker.

<div id="attachment_324" style="width: 1025px" class="wp-caption aligncenter">
  <a href="http://www.loominate.net/wp-content/uploads/2012/03/LinkToStory.png"><img class="size-full wp-image-324" title="LinkToStory" src="http://www.loominate.net/wp-content/uploads/2012/03/LinkToStory.png" alt="Shows Pivotal Tracker icon and URL to Pivotal Tracker" width="1015" height="633" /></a>

  <p class="wp-caption-text">
    The Pivotal Tracker icon links back to the story identified in the commit.
  </p>
</div>


 [2]: http://userscripts.org/scripts/show/129133
