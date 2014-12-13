---
layout: post
title: "Paper API Client"
---

Don&rsquo;t put URL string literals in your code. Use my latest &ldquo;paper&rdquo; module instead.

It turns this:

```javascript
request({
    method: 'post',
    url: 'https://host.com/v1/items?api_key=ApiKey',
    json: {
        name: 'foo',
        description: 'bar'
    }
})
.on( 'response', function( res ) {
    console.log( JSON.parse( res.body ).id );
})
.on( 'error', function( err ) {
    console.error( err );
});
```

Into this:

```javascript
apiClient.createItem({
    name: 'foo',
    description: 'bar'
}, itemCreated );

function itemCreated( err, data ) {
    if ( err ) {
        console.error( err );
        return;
    }

    console.log( data.id );
};
```

You can read more about how it works on the GitHub page: <https://github.com/tybenz/paper-api-client>.

As always, any feedback is appreciated. Give me a shoutout on [Twitter](http://twitter.com/tybenz).
