---
layout: post
title: "Making Spam Cool Again"
nav: blog
---

I'm an information junky. I spend hours a day on [Twitter](http://twitter.com),
[Hacker News](http://news.ycombinator.com), [UseVim](http://usevim.com), [Giant
Robots](http://robots.thoughtbot.com), [DailyJS](http://dailyjs.com) &mdash; if
it's about developer-anything, I'm there. So I decided to share the wealth of
links that I visit on a daily basis with the world.

Introducing **spamthe.net** &mdash; my very own link spammer. Follow on twitter
([@spamthenet](http://twitter.com/spamthenet)), bookmark in your browser
([spamthe.net](http://spamthe.net)) \*, subscribe to the weekly newsletter
([here](http://spamthe.net)), or join the IRC channel on Freenode
([#spamthenet](http://webchat.freenode.net)).

The links themselves can be found easily by anyone else who is as obsessed
with the above-mentioned resources as I am. But the fun part was building it.

I use [GitHub Pages](http://pages.github.com) to host the site and a node.js
app to update twitter and the blog simultaneously. Oh, and I wrote a
bookmarklet so that I can just click a button from within a site and send it to
the spammer.

The node app listens for a request (CORS of course) from the
bookmarklet. The title and url are passed in the request, and the app
posts to the twitter account and writes a new file into the website's repo,
commits, and pushes to GitHub. Now I can share links with the click of a
button. Copy + paste is so 2013.

The IRC channel is running a version of [hubot](http://hubot.github.io) that
listens to spamthenet's Twitter stream and auto-posts the links.
Join the channel if you feel like chatting with cool people while you wait for
the links to pour in.

[Click here](http://github.com/tybenz/spamthe.net/tree/master) to check out the
awful source. If you're interested in helping me collect links, ping me on IRC
(tybenz) or Twitter ([@tybenz](http://twitter.com/tybenz)) and be sure to check the
[contributing](https://github.com/tybenz/spamthe.net#contributing) section of the
README.

Also don't forget to bookmark [spamthe.net](http://spamthe.net), sign up for
our newsletter, follow [@spamthenet](http://spamthe.net), and join us on IRC.
Or maybe just one of those, if you feel doing all of them would be redundant.

\* I know the site has spam in the title. So here's a screenshot for all you out
there who are too scared to actually click on the link:

[![](http://awes0.me/spamsite.png)](http://spamthe.net)
