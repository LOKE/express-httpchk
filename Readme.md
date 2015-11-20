# express-httpchk


## example

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

    server server1 127.0.0.1:8000 check
```
