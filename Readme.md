# express-httpchk

Middleware to provide status endpoint for haproxy.

## Example

Add status endpoint in express.

On shutdown start sending 404, then wait for haproxy to report DOWN
before closing.

```javascript

var express = require('express');
var httpchk = require('express-httpchk');

var app = express();


function getStatus(haproxyState) {
  return { status: 'OK' };
}

var statusMiddleware = httpchk(getStatus);

app.get('/status', statusMiddleware)

var server = app.listen('3000');

process.on('SIGTERM', function() {
  // wait for haproxy to tell us we're out of the lb pool.
  statusMiddleware.shutdown()
  .then(function () {
    server.close();
  });
});

```

An appropriate haproxy config. Notice the backend options.

```
global
    daemon

defaults
    mode http

frontend fe
    bind *:80
    default_backend be

backend be
    option httpchk GET /status
    http-check disable-on-404
    http-check send-state

    server server1 127.0.0.1:3000 check
```
