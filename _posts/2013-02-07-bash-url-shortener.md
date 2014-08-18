---
layout: post
nav: blog
title: "Bash URL Shortener"
---
A couple months ago I got curious about URL shorteners and how they were
implemented. As I was researching the subject, I realized it might be
fun to write my own. So, of course, the next logical step was to steal
someone else's script and modify it for my needs. (I've lost the link to
that actual script I used for my inspiration but it was similar to
[this](https://gist.github.com/zumbojo/1073996) one.)

I wanted my URL Shortener to meet a few criteria:

1. Bash: I wanted to call the script from the command line, have it
   register the redirect on the server and put the shortened URL in my
   clipboard

2. GitHub Pages: I wanted the server with the redirects to be hosted for
   free. That naturally brought me to GitHub.

3. Awesomeness: I wanted my shortened URLs to live on an awesome domain.
   What's more awesome than the word itself right?

So I registered the domain awes0.me, ported the script over to BASH.
And deployed to Github.

The resulting script keeps a running count of how many URLs have been
generated, generates a hash on that value, creates a directory with that
hash as its name and injects a simple JavaScript redirect into an
index.html document inside that directory, and pushes the GitHub Repo.
Deploying a URL that looks something like: <http://awes0.me/K6g>.

And that's it. I can now shorten URLs to my heart's content. No PHP, no
paid hosting, no headaches. You can view the source [here](https://gist.github.com/4735033).
