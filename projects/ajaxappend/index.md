---
layout: default
project: true
nav: projects
title: Ajax Append
hidden: true
fork:
zip:
tar:
summary: Javascript plugin to tie AJAX responses to template injection
---


This is a jQuery Plugin designed to work with Rail's UJS remote forms.
It makes it easier to automatically inject populated templates into your page based on a JSON AJAX response.

## Example

This is an example of a form that is meant to grab a block of "posts" and inject them into a feed div

{% highlight html %}
<!-- Initializing the ajaxAppend plugin -->
<script type="text/javascript">
    $( '#rf1' ).ajaxAppend({
        dataName: 'stories',
        templates: [{
            template: $( "#story-template" ).text(),
            selector: '#whats-new-feed',
            order: 'prepend'
        }],
        formReset: false
    });
</script>

<!-- Template for our news story -->
<script type="underscore-template" id="story-template">
    <% _.each(stories, function( post ) { %>
        <div class="post">
            <h4>{{post.headline}}<span class="date">{{post.date}}</span></h4>
            <p>
                {{post.story}}
            </p>
        </div>
    <% }); %>
</script>

<div id="whats-new">
    <h3>News Example</h3>
    <form id="rf1" data-remote="true" action="get.php" method="post" style="">
        <input type="hidden" name="num" value="0" />
    
        <input type="submit" value="Get More News" />
    </form>
    <!-- This is where the templates will be injected -->
    <div id="whats-new-feed">
    </div>
    <br class="clear" />
</div>
{% endhighlight %}
