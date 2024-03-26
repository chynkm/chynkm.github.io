---
author: "Karthik M"
title: "Securely transmit real-time broadcasts over HTTPS"
date: "2020-05-22"
canonicalUrl: https://litebreeze.com/software-development/transmit-real-time-broadcasts/
tags:
- node.js
- socket.io
- laravel
- security
---

PHP frameworks like [Laravel](https://laravel.com/) aren’t built to handle real-time applications.
Hence, we integrate & transfer this responsibility to another application which excels in it.
[Socket.io](https://socket.io/) is my de facto choice for developing real-time, bidirectional and event-based communication.

Many posts on the internet don’t provide a fitting solution to run the WebSockets using HTTPS over TLS/SSL.
The solution mentioned in [Laracasts](https://laracasts.com/index.php/discuss/channels/forge/setting-on-nodejs-socketio-on-laravel-forge)
and [StackOverflow](https://stackoverflow.com/a/31165649) relies on providing read access to SSL certificates.
This means that either superuser access is required for running the Node server or to defy security by providing
read access to the website’s SSL certificates.

In this post, we will discuss an alternate method. The solution assumes that NGINX is used as the webserver.
Redis is used as the broadcast driver for Socket.io. Here’s a sample script which listens to socket connection requests
and broadcasts messages over established connections:

```
var app = require('express')();
var server = require('http').Server(app);
var io = require('socket.io')(server);
require('dotenv').config();

var redisPort = process.env.REDIS_PORT;
var redisHost = process.env.REDIS_HOST;
var ioRedis = require('ioredis');
var redis = new ioRedis(redisPort, redisHost);

redis.subscribe('action-channel-one', 'action-channel-two');
redis.on('message', function (channel, message) {
    message  = JSON.parse(message);
    io.emit(channel + ':' + message.event, message.data);
});

var broadcastPort = process.env.BROADCAST_PORT;
server.listen(broadcastPort, function () {
    console.log('Socket server is running.’);
});
```

Let’s assume that Socket.io server is running on the port 3444(available through the variable process.env.BROADCAST_PORT
configured in Laravel's .env file). Run the server script to bind the listener to the port 3444.

Next, we proceed to include the Socket.io JS library and the JavaScript snippet to handle the broadcast event.

```
<script src=”path/to/socket.io.js”></script>
<script>
io().on("action-channel-one:App\\Events\\ActionEvent", function(data){
    // fetch and process the data, add needed functionality here
});
</script>
```

As we load the webpage, the socket is registered.
Closer inspection of the polled WebSocket will reveal the requested URL in the following format:

```
wss://example.com/socket.io/?EIO=3&transport=websocket&sid=4aGAhWliFdIaGmfUAAGl
```

From the URL, we can understand that the requests are routed to the default path `/socket.io`.
We modify the NGINX config file to capture the requests to `/socket.io` and to forward(proxy) it to the port 3444 where
the Socket.io server is listening. This is achieved by adding the following to the NGINX configuration file of the web
application.

```
location /socket.io/ {
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_pass http://localhost:3444;
}
```

Once the above modifications are in place, we would need to restart the NGINX web server for the change to take effect.
In this manner, we utilise the existing SSL certificate of the webserver to run a secured broadcasting solution.
