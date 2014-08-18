---
title: Widget-ize All The Things!
layout: post
nav: blog
---

**Reduce, reuse, refactor.** That&#8217;s what we do as developers _all day_.
The **reduce** and **refactor** are obvious necessities, and admittedly some of the most fun and/or
most painful parts of the job. In my experience, the **reuse** goal is often the easiest to
acheive. Yet, somehow, it is often the one most ignored, in the name of
"iterative development". Trust me, nothing helps you iterate more
quickly than
writing things in a generic and reusable way the _first_ time.

In my experience writing code to be purposefully reusable is fun,
challenging, and incredibly useful. I want to outline what writing
solid, reusable front-end web components should look like.

There are 3 different approaches when it comes to solving a UI problem or implementing a common UI pattern.

1. Blackbox
2. Framework-based
3. The sweet spot

### Blackbox

The blackbox approach is one that _every_ web developer is familiar with.
What I&#8217;m referring to is a UI-based plugin that solves a problem, but
might require specific markup structure,
or an entire 300 lines of code to accomplish something you could have done in 20. It&#8217;s a plugin you grab and throw
in your app/site and it&#8217;s supposed to just "work". And it does for the most part. The convenience is tempting.

The problem comes when you want to customize/extend the thing to do a
little something extra. Most plugins don&#8217;t leave room for
interaction with the plugin itself. Sure, you got a slideshow widget by
downloading a couple of files, but what good is it if it&#8217;s not
fully customizable?

### Enter the example

The example I&#8217;ll be using throughout this post is what I call a "secret"
widget. The pattern we're going for is simple: mousedown -> add a class,
mouseup -> remove a class. That class can do whatever it wants in CSS.
We&#8217;re going to have it hide/show a child element (thus the name
"secret").
Let&#8217;s take a look at the blackbox approach to this widget:

{% highlight javascript %}
$.fn.secret = function() {

    return this.each(function() {
        var $this = $( this );
        $this.on( 'mousedown', function() { $this.addClass( 'show' ); } );
        $this.on( 'mouseup', function() { $this.removeClass( 'show' ); } );
    });

};
{% endhighlight %}

Live demo [here](/demos/widgetize/secret.html)

This is an example of a typical jQuery plugin\*. It has no options,
and very simple widget logic. What if I wanted to use different events
other than mousedown/mouseup? What if I want some other piece of an
application to perform an action when the class is applied? There's
nothing external for integration and not enough options for it to do
what I want.

### Framework-based

Next up is the framework-based approach to writing components. I&#8217;m
personally a big fan of MVC in app development. But, when it comes to
writing UI components, there&#8217;s one thing I get annoyed with
often &mdash; if you do it the "framework" way, it&#8217;s less reusable by
definition.

Here&#8217;s an example:

{% highlight javascript %}
var Secret = Backbone.View.extend({
    events: {
        'mousedown': 'apply',
        'mouseup': 'remove'
    },

    apply: function() { this.$el.addClass( 'show' ) },

    remove: function() { this.$el.removeClass( 'show' ) },
});
{% endhighlight %}

Notice that there are no options at all in this view. This is because
it&#8217;s custom-made for this application and we (ideally) know what our requirements are
while growing the codebase.

There&#8217;s an even bigger problem here, we&#8217;ve just rewritten a common UI
widget in a very specific "Backbone-y" way. All reusability is gone
(outside of another Backbone project that is). Converting
this code into a more flexible widget will take time and effort.

### The Sweet Spot

That&#8217;s where option #3 comes in. The compromise between isolated,
they-just-work plugins and event-driven MVC-friendly components. A solid
reusable widget should have the following criteria:

1. Options (and defaults)
2. Browser/User events
3. Widget Logic
4. External events

Let&#8217;s take a look at some code within the constructor of our
widget:

{% highlight javascript %}
widget.options = $.extend( {}, widget.defaultOptions, options );
widget.$el.on( widget.options.applyEvent, function() {
    // apply/remove are just calling addClass/removeClass respectively
    widget.apply();
    widget.trigger( 'secret-apply' );
});
widget.$el.on( widget.options.removeEvent, function() {
    widget.remove();
    widget.trigger( 'secret-remove' );
});
{% endhighlight %}

Here we have a component that is customizable with options, that does its
job as it&#8217;s supposed to and that provides ways for other pieces of an
application to hook into and modify its behavior.

### Time to extend

Let&#8217;s try to extend this widget's behavior. What if I had two divs
on the page each with their own secret? And, let&#8217;s say I want to
show both secrets when I mousedown the first div.
With our new event-based widget, it&#8217;s simple:

{% highlight javascript %}
var secret1 = new Secret( $( '#secret1' ) ),
    secret2 = new Secret( $( '#secret2' ) );

secret1.on( 'secret-apply', function() {
    secret2.apply();
});

secret1.on( 'secret-remove', function() {
    secret2.remove();
});
{% endhighlight %}

Live demo [here](/demos/widgetize/secret4.html).

Notice that the secret widget does it&#8217;s normal thing. It&#8217;s really good
at applying/removing classes, but it also fires events when those things
happens. This let&#8217;s us write a bit of code to create the master/slave
relationship. If we want, we could even package this new behavior as a
widget.

Slick huh? Now you&#8217;ve got a widget that can handle a simple UI pattern,
and it&#8217;s customizable/extensible enough to do something _really_ useful.

Also, if you haven&#8217;t noticed, our example widget is extremely
simple. But I wanted to show how small components can be used to do cool
things (especially when they leverage CSS).
<a href="/demos/widgetize/secret5.html" target="_blank">Here&#8217;s</a> an example I made to show
that. The example consists of a whole bunch of secret widgets lined up side-by-side.
Each widget contains an image and the widget&#8217;s applyClass toggle's
the visibility of the image. I'm also using some CSS trickery here to
shift the child image a certain amount so that they all have the same
top and left coordinates. We use the events mouseover and mouseout for this one,
giving the effect that hovering over the image from left/right scrubs a
rotating animation.

### Wrap up

In my experience amassing an arsenal of these small pattern-based
reusable bits of code can really make working on any application so much
easier. Give it a try and you&#8217;ll find that building out a library of
these types of components is extremely rewarding and in general, it will
make you a better developer. And if you don&#8217;t have the time to
roll your own, I hope you&#8217;ll at least
research some widget-based frameworks and begin to "think reusable" when
faced with a UI problem.

I&#8217;ve tried to lay out what I think is a template for simple, but
powerful components so that you can go forth and widgetize all of your UI.
As always feel free to submit any comments/questions on
[Twitter](http://twitter.com/tybenz). Also, I&#8217;ll be giving a talk
on this very topic at a meetup with the [Sonora Software Developers
Group](http://www.meetup.com/The-Sonora-Software-Developers-Group/) next Month.
If you happen to be near the Central Valley/Foothills, swing by.

\* To clarify, I have nothing against jQuery plugins. In my experience
most jQuery plugins are written to solve a specific problem. They do
their job well but they don&#8217;t necessarily provide
the options and events to modify/extend behavior.
