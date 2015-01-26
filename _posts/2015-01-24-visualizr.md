---
layout: post
hidden: true
title: "Visualizr: Made with Web Audio and Canvas"
---

I made a thing with Web Audio and Canvas. It's a music visualizr. You can play
with the full thing [here](http://tybenz.com/visualizr), but here's a quick
demo of it:

<div class="video-container skinny">
  <iframe frameborder="0" src="http://tybenz.com/visualizr/#width=15&height=1&gap=12&delay=40&hue=0&animate=out&auto_delay=5000&song=let_go&hide_controls=1&small=1"></iframe>
</div>

<p>&nbsp;</p>

The Web Audio API is well-supported across modern browsers, except for IE (big
surprise). It allows you to not only play sounds but also manipulate them at a
very low level. It's incredibly powerful, if you know what you're doing.

For my first Web Audio project, I wanted to just mess around with getting a
song to play and having some animation to match it.

Here's how you can get a basic song to play with Web Audio:

```javascript
var context = new AudioContext();

var request = new XMLHttpRequest();
request.open( 'GET', songUrl, true );
request.responseType = 'arraybuffer';
request.onload = function() {
    context.decodeAudioData( request.response, function( buffer ) {
        var source = context.createBufferSource();
        source.connect( context.destination );
        source.buffer = buffer;
        source.start( 0 );
    });
};

request.send();
```

Here's how you can get analyser data on the context's destination:

```javascript
/*
 * This is inside decodeAudioData callback
 */

// Set up analyser
var analyser = context.createAnalyser();
analyser.smoothingTimeConstant = 0.3;
analyser.fftSize = 1024;
var bufferLength = analyser.fftSize;
var dataArray = new Uint8Array( bufferLength );

// This will periodically compute average volume and log it out
setInterval( function() {
    analyser.getByteFrequencyData( dataArray );

    // Sum frequency amplitudes
    var values = 0;
    for ( var i = 0; i < bufferLength; i++ ) {
         values += dataArray[ i ];
    }

    // Log average volume
    console.log( values / bufferLength );
}, 100 );

var source = context.createBufferSource();
// Hook up the source to the analyser and the destination
source.connect( analyser );
source.connect( context.destination );
source.buffer = buffer;
source.start( 0 );
```

From there, it really just became about having fun with canvas. I had the raw
data, just needed to do something with it. Here's a quick example of how to
draw a rectangle whose height matches the average volume:

```javascript
var context = new AudioContext();
var analyser;
var bufferLength = 1024;
var dataArray = new Uint8Array( bufferLength );

var request = new XMLHttpRequest();
request.open( 'GET', 'https://s3.amazonaws.com/tybenz.assets/visualizr/really_wanna.mp3', true );
request.responseType = 'arraybuffer';
request.onload = function() {
    // Audio stuff goes in here
    // analyser is initialized and its fftSize
    // is set to bufferLength (1024)
    // Also, draw will be kicked off after it's all set up
};

var canvas = document.getElementById( 'canvas' );
var canvasWidth = canvas.width;
var canvasHeight = canvas.height;
var rectWidth = 20;
var ctx = canvas.getContext( '2d' );

var draw = function() {
    analyser.getByteFrequencyData( dataArray );

    // Sum frequency amplitudes
    var values = 0;
    for ( var i = 0; i < bufferLength; i++ ) {
         values += dataArray[ i ];
    }

    var volume = values / bufferLength;

    ctx.clearRect( 0, 0, canvasWidth, canvasHeight );
    ctx.fillStyle = 'black';
    ctx.fillRect(
        canvasWidth / 2 - rectWidth / 2,
        canvasHeight / 2 - volume / 2,
        rectWidth,
        volume
    );
    requestAnimationFrame( draw );
};
```

After that, it was all about improving the animation. I created a collection of
bars whose heights decrease as you go away from the center. I then added a
delay to when the change to the smaller bars' heights gets applied. If you
visit the full visualizr here: <http://tybenz.com/visualizr>, you can play with
various parameters and see how it effects the animation.

If you'd like to see more details, the source is up on [GitHub](
http://github.com/tybenz/visualizr) and
[CodePen](http://codepen.io/tybenz/pen/dPRWJa).
