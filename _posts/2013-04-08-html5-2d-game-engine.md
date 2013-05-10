---
layout: post
hidden: true
title: "HTML5 2D Game Engine"
---
I am not a game developer... yet. But I&#8217;ve had an idea for a 2D
platforming game floating around in my head for a few years now and I
finally decided it was time to get serious about it.

I had noodled around with it in the past. But never made a definitive
decision on the platform/framework I&#8217;d work with. After programming
almost exclusively in Javascript for the last few years and considering the portability that the
web offers, I decided HTML5 would be my weapon of choice. Considering
myself to be a hardcore JS developer and after researching the various
frameworks/engines that are available (it&#8217;s slim pickings by the way),
I came to realization that I&#8217;d have much more fun/control if I
wrote everything from scratch. I invited my friend
[Robby Nickles](http://github.com/robbynickles) to join me in tackling the task.
Here&#8217;s what we did.

**Disclaimer**: This is an overview of the basics of a 2D platform game engine I had to
develop. Most of the techniques are probably not considered to be game
dev "best practices". This is just what we managed to cobble together
with a little reading and limited (or none) experience.

Let me give a brief overview on what the mechanics/story of the game will
be . I&#8217;m calling it [Morph](http://github.com/blacktunnel/morph) and it&#8217;s a simple grid-based
2D platform game (you can read more
[here](http://blacktunnel.github.io/morph/design.html)). The main character
is a shape shifter and his many forms offer different attacks/abilities.
The fun of the game will be choosing the right character for the right
task. On top of that I&#8217;m really interested in making the game an
open-world RPG.

##### Rendering: Sprites
Each sprite of the game is drawn within a 9x9 grid. We actually wrote a
little canvas-based [drawing app](http://blacktunnel.github.io/morph/draw.html)
to quickly create the sprites. The output of the drawing app is not a
PNG. It spits out a 2D array of colors for each sprite that is
used to initialize the graphics in the game.

Why not use PNGs from the start you ask? We wanted to be able to change
the size of game&#8217;s sprites down the road. That way, when the time comes,
we could write different versions for different screen sizes.

Here's an example of the game with **very** large sprites:

<iframe src="http://blacktunnel.github.io/morph/preview/embed/?level=mini" height="420"></iframe>

So the bulk of each entity's constructor is devoted to drawing each
sprite and caching them into image objects for quick rendering later.

{% highlight javascript %}
var i, j, k,
    tempCanvas, tempContext,
    dataURL, currentSprite,
    rectSize = Game.unit / 9;

for ( i in this.bitmaps ) {
    currentSprite = this.bitmaps[ i ];
    tempCanvas = document.createElement( 'canvas' );
    tempCanvas.width = this.width;
    tempCanvas.height = this.height;
    tempContext = tempCanvas.getContext( '2d' );
    for ( j in currentSprite ) {
        for ( k in currentSprite[ j ] ) {
            tempContext.fillStyle = currentSprite[ j ][ k ];
            tempContext.fillRect( k * rectSize, j * rectSize, rectSize, rectSize );
        }
    }
    dataURL = tempCanvas.toDataURL( 'image/png' );
    this.sprites.push( Game.Sprite( dataURL, this.type ) );
}
{% endhighlight %}

As you can see, I can easily change the size of my sprites by alter a
single variable: `Game.unit`.

##### Rendering: invalidateRect

The next step was putting the pixels on the screen. We started by simply
redrawing every entity on every iteration of the game&#8217;s loop. That had
some performance implications. Mountain Lion&#8217;s Activity Monitor was
reporting 98% CPU usage while the game was running (more than half on
the rendering side). So it was time to profile and refactor. We
implemented a good-ol' invalidateRect algorithm and now we only redraw
entities that have changed or moved.

Play the video below to see how it works:

<video src="/video/morph-invalidate-rect.mov" preload="auto" controls></video>


##### Rendering: Animation

Finally, we needed a way to animate a sprite. Each entity has a list of
sprites and a variable called activeSprite which is used to put the
right sprite on the screen on each render. The problem was we didn&#8217;t have a way
to change what activeSprite was. At least not in a sane way. So we came
up with this:

{% highlight javascript %}
Game.Entity.Enemy.Bird = Game.Entity.Enemy.extend({
    init: function( x , y ) {
        this._super( x, y );
        this.animationStates = {
            flying: {
                delta: 200,
                sequence: [ WINGS_UP, WINGS_DOWN ],
                times: 'infinite'
            }
        };
    }
    // ...
});
{% endhighlight %}

Animation states are defined with in the entity&#8217;s constructor and are
used by the base entity to iterate/cycle through the array of sprites.
`WINGS_UP` and `WINGS_DOWN` are merely constants for the index into the
sprites array. The base entity&#8217;s animation logic cycles through the
sequence according to the state the entity is in.

If you'd like to see how the animationStates dictionaries are put to
use, take a look at the `animate` and `nextStep` methods in the
[source](https://github.com/blacktunnel/morph/blob/master/game/entity.js#L50).

##### Collision Detection

Our implementation is rather run-of-the-mill at the moment, and I&#8217;m sure
its complexity will increase as we continue on. For now we&#8217;re using a sort
& sweep algorithm to trim down the list of collision checks. Each collision
check looks for various types of collisions: overlapping, edge
collisions, and exact. If a collision is found we call the entities'
collision handlers. Here&#8217;s the base entity&#8217;s default collision handler
(what happens when an entity hits a wall or the ground):

{% highlight javascript %}
collideWith: function( entity, collisions ) {
    switch ( entity.type ) {
        case 'Terrain.Land':
            if ( this.velocity.y > 0 && 'bottomEdge' in collisions ) {
                this.velocity.y = 0;
                this.pos.y = entity.pos.y - entity.height;
            }
            if ( this.velocity.y < 0 && 'topEdge' in collisions ) {
                this.velocity.y = 0;
                this.pos.y = entity.pos.y + entity.height;
            }
            if ( this.velocity.x < 0 && 'leftEdge' in collisions ) {
                this.velocity.x = 0;
                this.pos.x = entity.pos.x + entity.width;
            }
            if ( this.velocity.x > 0 && 'rightEdge' in collisions ) {
                this.velocity.x = 0;
                this.pos.x = entity.pos.x - entity.width;
            }
            break;
        default: break;
    }
}
{% endhighlight %}

##### Fun stuff

After that stuff was in place it was time to start having fun. We bound
keys to the hero&#8217;s movement, defined physics for jumping (gravity),
gave the enemy entities some simple AI, and lots of
other little tweaks until we could actually start to see the basis of a
platform game forming in front of our eyes.

Oh, and one more thing. We implemented our first transformation. The
premise of the game is that you can morph into different
shapes/characters. The main character starts as a block with an impressively high jump
but using the machine allows him to turn into a human-like
character who can pickup and throw rocks. Trust me, that alone is
entertaining. Try it for yourself:

<iframe src="http://blacktunnel.github.io/morph/preview/embed" height="366"></iframe>

Thanks for reading. If you have experience with game development and you
noticed that I'm doing things absolutely wrong, please
<a target="_blank" href="https://twitter.com/share?text=Hey %40tybenz! You're doing it wrong. ">let me know</a>.
I'd love help in making sure I'm doing things as cleanly as possible.
