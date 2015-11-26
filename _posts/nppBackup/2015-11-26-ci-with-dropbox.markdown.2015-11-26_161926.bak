---
title: Socket.io first look
layout: post
---

# Socket.io first look

Hi friends,

I saw a project on freelancer and that wanted to see phonegap and socket.io in action. I put on my thinking cap and started exploring. Socket.io is a beautifully brewed concept where sockets are used for transport. 

# Namespaces

There are namespaces, which you can create and the client will connect to that. 

{% highlight javascript %}
// the following two will emit to all the sockets connected to `/`
io.sockets.emit('hi', 'everyone');
io.emit('hi', 'everyone'); // short form
{% endhighlight %}

Each namespace emits a connection event that receives each Socket instance as a parameter.

{% highlight javascript %}
io.on('connection', function(socket){
  socket.on('disconnect', function(){ });
});
{% endhighlight %}

# Rooms

Also, sockets can be put in channels. When an socket is initialized, it is part of a room which is equal to its socket id.

{% highlight javascript %}
io.on('connection', function(socket){
  socket.join('some room');
});
{% endhighlight %}

And then simply use *to* or *in* (they are the same) when broadcasting or emitting:

{% highlight javascript %}
io.to('some room').emit('some event'):
{% endhighlight %}

To leave a channel you call *leave* in the same fashion as *join*.

# Default room

Each *Socket* in Socket.IO is identified by a random, unguessable, unique identifier *Socket#id*. For your convenience, each socket automatically joins a room identified by this id.

This makes it easy to broadcast messages to other sockets:

{% highlight javascript %}
io.on('connection', function(socket){
  socket.on('say to someone', function(id, msg){
    socket.broadcast.to(id).emit('my message', msg);
  });
});
{% endhighlight %}

# Disconnection

Upon disconnection, sockets *leave* all the channels they were part of automatically, and no specially teardown is needed on your part.

Knowing this, I leaped into code.

The client side code is  as below:

{% highlight javascript %}
<!doctype html>
<html>
  <head>
    <title>Pingu chat</title>
    <style>
      * { margin: 0; padding: 0; box-sizing: border-box; }
      body { font: 13px Helvetica, Arial; }
      form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
      form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
      form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
      #messages { list-style-type: none; margin: 0; padding: 0; }
      #messages li { padding: 5px 10px; }
      #messages li:nth-child(odd) { background: #eee; }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form action="">
      <input id="m" autocomplete="off" /><button>Send</button>
    </form>
    <script src="https://cdn.socket.io/socket.io-1.2.0.js"></script>
    <script src="http://code.jquery.com/jquery-1.11.1.js"></script>
    <script>
      var socket = io.connect('http://52.18.12.164:8080/');
      $('form').submit(function(){
        socket.emit('chat message', $('#m').val());
        $('#m').val('');
        return false;
      });
      socket.on('chat message', function(msg){
        $('#messages').append($('<li>').text(msg));
      });
    </script>
  </body>
</html>
{% endhighlight %}

And the server side code is:

{% highlight javascript %}
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.get('/', function(req, res){
  res.sendFile(__dirname + '/index.html');
});

io.on('connection', function(socket){
  socket.on('chat message', function(msg){
    io.emit('chat message', msg);
  });
});

http.listen(8080, function(){
  console.log('listening on *:8080');
});
{% endhighlight %}

To start serving through node, run the commend (preferably in screen, so that it keeps running):

{% highlight javascript %}
node index.js
{% endhighlight %}

So, the app Pingu chat is running on:

http://52.18.12.164/machinenode.tk/socket/index.html

Cheers!