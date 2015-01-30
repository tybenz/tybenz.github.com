---
layout: post
title: "Visualizr: Fun with Web Audio and Canvas"
---

I made a thing with Web Audio and Canvas. It's a music visualizr. You can play
with the full thing [here](http://tybenz.com/visualizr), but here's a quick
demo of it (music by [mrsjxn](http://mrsjxn.com)):

<div class="video-container skinny">
  <iframe frameborder="0" src="http://tybenz.com/visualizr/#width=15&height=1&gap=12&delay=40&hue=0&animate=out&auto_delay=5000&song=let_go&hide_controls=1&small=1"></iframe>
</div>

<p>&nbsp;</p>

## Crash-course

The rest of this blog post will be a sort of crash-course in how to get a basic
visualization set up with Web Audio and Canvas.

The Web Audio API is well-supported across modern browsers, except for IE (big
surprise). It allows you to not only play sounds but also manipulate them at a
very low level.

First, an example of just getting a song to play with Web Audio:

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

Web Audio also contains a set of interfaces that allow you to analyze and
manipulate frequency data. `AnalyserNode` is meant for reading frequency data
only and it's perfect for the purposes of visualization.

You can create an `AnalyserNode` with `context.createAnalyser()`, and "hook it"
into the audio source with `source.connect( analyser )`.

Once you have the analyser set up, you can read in frequency data synchronously
by using `anaylser.getByteFrequencyData()`. To demonstrate, here's an example that
uses `setInterval` to periodically log the average volume:


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

Now, it's time to have fun with canvas. We've got some volume data. Let's do something with it.

First, a couple quick notes about canvas:

1. We can call `createContext( '2d' )` on the canvas DOM element to get the rendering
   context. This is the object that gives you access to all the drawing methods
2. Use `requestAnimationFrame`, not `setInterval` for constant re-rendering.
   `requestAnimationFrame` is the browsers way of binding a callback to the next
   tick of the browser's internal render loop. It's much more performant, and
   simple to use.
3. On each tick of the clock, wipe the canvas clean. Each frame will be drawn from scratch.
   We'll do this with `clearReact( 0, 0, canvasWidth, canvasHeight )`.

Here's a quick example using `requestAnimationFrame`, `clearRect` and
`fillRect` to draw a rectangle whose height matches the average volume:

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

And that's it. The visualizr I created wasn't much different from this example.
After setting this up, I just worked on improving the animation. I created a collection of
bars whose heights decrease as you go away from the center. I then added a
delay to when the change to the smaller bars' heights gets applied.

If you visit the full visualizr here: <http://tybenz.com/visualizr>, you can
play with various parameters and see how it effects the animation. If you'd
like to see more details, the source is up on [GitHub](
http://github.com/tybenz/visualizr) and
[CodePen](http://codepen.io/tybenz/pen/dPRWJa).

As always, if you have questions/comments, you can find me on
[Twitter](http://twitter.com/tybenz).
