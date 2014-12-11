---
layout: post
title: "Paper Profiler"
---

I wrote a small profiling helper for NodeJS apps today.

It's called paper-profiler and you can read more about it here: <https://github.com/tybenz/paper-profiler>

### Example

If you're app looks like this:

```javascript
var express = require( 'express' );
var profile = require( 'paper-profile' ).create();
var fs = require( 'fs' );
var path = require( 'path' );
var service = require( './service' );

var app = express();

app.get( '/', function( req, res, next ) {
    var reqId = req.get( 'X-request-id' );
    profile( reqId, 'getInfo' );

    server.getInfo( function( err, data ) {
        if ( err ) {
            res.send( 500 );
            return;
        }
        profile( reqId, 'getInfo' );

        profile( reqId, 'writeToFile' );
        fs.writeFileSync( path.join( __dirname, 'path', 'to', 'file' ), JSON.stringify( data ), 'utf8' );
        profile( reqId, 'writeToFile' );

        console.log( profile.getTimes( reqId ) );
        profile.done( reqId );
        res.send( data );
    });

});

app.listen( 8080, function() {
    console.log( 'Server started up' );
});
```

Profiling might look like:

```
[
  {
    "name": "getInfo",
    "elapsedTime": "0.51s"
  },
  {
    "name": "writeToFile",
    "elapsedTime": "1.32s"
  }
]
```
