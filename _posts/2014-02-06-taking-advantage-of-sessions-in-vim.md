---
layout: post
title: "Taking advantage of sessions in Vim"
nav: blog
---

A few months ago, on my daily hunt for new, useful Vim plugins, I discovered
[vim-startify](http://github.com/mhinz/vim-startify). Startify is a beautiful
and organized startup screen for Vim. It provides shortcuts into files from
your working directory, a list of bookmarks to files you edit all the time, and
your most-recently edited files. As I was going over its docs, I also
discovered that it can list out Vim sessions. I'd never read about sessions
before. So I began googling.

Vim provides some commands that let you store sessions based on the current
state of Vim. You can load in previously created sessions with `source` (just
like any other Vim script). The `mksession` command, out-of-the-box, stores every
little detail about your current session &mdash; plugins, scripts, which buffers are
loaded, what your tabs and windows are doing.  Because of this, a `mksession`
file can be really large and really ugly. I actually found that most of the
things `mksession` was saving to be a bit redundant. For the most part, the
scripts I want executed when I launch Vim are all present in my `.vimrc`. So I
took the bits that I needed and starting creating my own session files by hand.

The only commands I really need in a project-specific session have to do with
changing Vim's working directory, opening files into buffers, and choosing
which of those buffers to edit (I usually start with two files in vertical
split panes).

Here's one of my session files (`vimdeck.vim`):

<div class="highlight">
<pre><code><span class="k">cd</span> ~/src/vimdeck
<span class="k">args</span> lib/* lib/templates/* bin/vimdeck Rakefile Gemfile README.md slides.md
<span class="k">edit</span> slides.md
<span class="k">vertical sb</span> lib/vimdeck.rb</code></pre>
</div>

I change the working directory to the project folder, open files into buffers,
then I choose which two files I want to start with (in the above case slides.md
and lib/vimdeck.rb). And I can execute all of this by typing:

{% highlight text %}
:source ~/.vim/sessions/vimdeck.vim
{% endhighlight %}

Now sessions get even cooler when you pair sessions with Startify. Startify
will list all session files from a directory and provide keyboard shortcuts to
execute them. I put my session files into `~/.vim/sessions` and tell startify
where they're located with:

{% highlight vim %}
let g:startify_session_dir = "~/.vim/sessions"
{% endhighlight %}

Now everytime I start Vim, I'm greeted with this:

![](http://awes0.me/startify.png)

Note that the first section consists of a list of numbered sessions. Startify
assigns shortcut keys to them so if I want to jump into my Vimdeck session, all
I have to do is open Vim and hit `h` and it will drop me into the right
directory with all of the files I want to be able to edit for that project.

Startify also provides directory-level session file detection. So it will check Vim's
working directory for a `Session.vim` file and write out a shortcut for that
session as well. Useful if you'd like to include your session file in your
projects git repository.

Make sure you grab [startify](http://github.com/mhinz/vim-startify) and start
playing around with its session support. Also, hit me up on
[twitter](http://twitter.com/tybenz) if you have any questions/suggestions.
