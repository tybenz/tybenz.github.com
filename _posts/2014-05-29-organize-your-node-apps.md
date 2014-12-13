---
layout: post
title: "Paper Router"
---

I wrote a node module called [paper-router](http://tybenz.com/paper-router).
It's sole purpose is to help me organize my node apps.

## It turns this

```javascript
var restify = require('restify');
var BananaModel = require( './models/banana.js' );
var PeelModel = require( './models/peel.js' );

var server = restify.createServer({
    name: 'luca',
});

server.use( function crossOrigin( req, res, next ) {
    res.header( "Access-Control-Allow-Origin", "*" );
    res.header( "Access-Control-Allow-Headers", "X-Requested-With" );
    return next();
});

server.use( restify.acceptParser( server.acceptable ) );
server.use( restify.queryParser() );
server.use( restify.bodyParser() );
server.pre( restify.pre.sanitizePath() );

server.get( '/', function( req, res, next ) {
    res.send( { message: 'You are home' } );
})

server.get( '/bananas', function( req, res, next ) {
    res.send( BananaModel.getAll() );
});

server.post( '/bananas', function( req, res, next ) {
    var banana = BananaModel.new({
        weight: req.params.weight,
        length: req.params.length,
    })
    .save();

    res.send( banana );
});

server.get( '/bananas/:banana_id/peels', function( req, res, next ) {
    var peels = PeelModel.where( { banana_id: req.params.banana_id } );

    res.send( peels );
});

server.listen( 8000, function() {
    console.log( "%s listening at %s", server.name, server.url );
});
```

## Into this

```javascript
var restify = require('restify');
var PaperRouter = require( 'paper-router' );
var routes = require( './routes' );

var server = restify.createServer({
    name: 'luca',
});

server.use( function crossOrigin( req, res, next ) {
    res.header( "Access-Control-Allow-Origin", "*" );
    res.header( "Access-Control-Allow-Headers", "X-Requested-With" );
    return next();
});

server.use( restify.acceptParser( server.acceptable ) );
server.use( restify.queryParser() );
server.use( restify.bodyParser() );
server.pre( restify.pre.sanitizePath() );

//
// Using paper router to separate out controllers from routes
// Routes are read in from a separate file (see routes.js below)
//
var router = new PaperRouter( server, __dirname + '/controllers', routes );

server.listen( 8000, function() {
    console.log( "%s listening at %s", server.name, server.url );
});
```

routes.js:

```javascript
module.exports = function( router ) {
    // GET '/' => controller: 'home', action: 'index'
    router.get( '/', 'home#index' );

    // Creates the following routes but only if the actions exist
    // GET '/bananas' => controller: 'bananas', action: 'index'
    // GET '/bananas/:id' => controller: 'bananas', action: 'show'
    // GET '/bananas/new' => controller: 'bananas', action: 'new'
    // GET '/bananas/:id/edit' => controller: 'bananas', action: 'edit'
    // POST '/bananas' => controller: 'bananas', action: 'create'
    // PUT '/bananas/:id' => controller: 'bananas', action: 'update'
    // DELETE '/bananas/:id' => controller: 'bananas', action: 'destroy'
    router.resources( 'bananas' );

    // Creates the following routes but only if the actions exist
    // GET '/bananas/:banana_id/peels' => controller: 'peels', action: 'index'
    // GET '/bananas/:banana_id/peels/:id' => controller: 'peels', action: 'show'
    // GET '/bananas/:banana_id/peels/new' => controller: 'peels', action: 'new'
    // GET '/bananas/:banana_id/peels/:id/edit' => controller: 'peels', action: 'edit'
    // POST '/bananas/:banana_id/peels' => controller: 'peels', action: 'create'
    // PUT '/bananas/:banana_id/peels/:id' => controller: 'peels', action: 'update'
    // DELETE '/bananas/:banana_id/peels/:id' => controller: 'peels', action: 'destroy'
    router.resources( 'peels', '/bananas/:banana_id' );
};
```

bananasController.js:

```javascript
// bananasController.js
var BananaModel = require( '../models/banana.js' );

// Controllers are just plain objects with actions as properties
// Use typical CRUD action names to line up with paper-router's
// resourceful routing (see routes.js)
var BananaController = {
    index: function( req, res, next ) {
        res.send( BananaModel.getAll() );
    }

    create: function( req, res, next ) {
        var banana = BananaModel.new({
            weight: req.params.weight,
            length: req.params.length,
        })
        .save();

        res.send( banana );
    }
};

module.exports = BananaController;
```

peelsController.js

```javascript
// peelsController.js
var PeelModel = require( '../models/peel.js' );

var PeelsController = {
    index: function( req, res, next ) {
        var peels = PeelModel.where( { banana_id: req.params.banana_id } );

        res.send( peels );
    }
};

module.exports = PeelsController;
```

homeController.js

```javascript
// homeController.js
var HomeController = {
    index: function( req, res, next ) {
        res.send( { message: 'You are home' } );
    }
};

module.exports = HomeController;
```
