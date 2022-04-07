# Get started

In this application-guide we’ll create a basic
chat application.
It requires almost no basic prior knowledge of Node.JS
or Socket.IO, so it’s ideal for users of all knowledge levels.

## INTRODUCTION

Sockets have traditionally been the solution around
which most real-time chat systems are architected,
providing a bi-directional communication channel
between a client and a server, instead of popular web
application stacks like LAMP (PHP).

With sockets, the server can push messages to clients.
Whenever you write a chat message, the idea is that
the server will get it and push it to all other
connected clients.

## THE WEB FRAMEWORK

The first goal is to set up a simple HTML webpage
that serves out a form and a list of messages.
We’re going to use the Node.JS web framework
<i style="color: red">express</i> to this end. Make sure Node.JS is installed.

1. Let’s create a <i style="color: red">package.json</i> manifest
   file that describes our project.
   I recommend you place it in a dedicated empty directory.

```CS
{
  "name": "socket-chat-example",
  "version": "0.0.1",
  "description": "my first socket.io app",
  "dependencies": {}
}
```

Now, in order to easily populate the dependencies
property with the things we need,
we’ll use npm install:

```node
npm install express@4
```

Once it's installed we can create an <i style="color: red">index.js</i>
file that will set up our application.

```js
const express = require("express");
const app = express();
const http = require("http");
const server = http.createServer(app);

app.get("/", (req, res) => {
  res.send("<h1>Hello world</h1>");
});

server.listen(3000, () => {
  console.log("listening on *:3000");
});
```

This means that:

Express initializes <i style="color: red">app</i> to be a function handler
that you can supply to an HTTP server (as seen in line 4).
We define a route handler <i style="color: red">/ </i>
that gets called when we hit our website home.
We make the http server listen on port 3000.
If you run node <i style="color: red">index.js</i>
you should see the following:

![01-img](/socket-chat-example/screenshot/Screen%20Shot%202022-04-04%20at%204.41.21%20PM.png)

And if you point your browser to http://localhost:3000:

![02-img](/socket-chat-example/screenshot/Screen%20Shot%202022-04-04%20at%204.48.47%20PM.png)

## Serving HTML

So far in <i style="color: red">index.js</i>
we’re calling <i style="color: red">res.send</i>
and passing it a string of HTML. Our code would look
very confusing if we just placed our entire application’s
HTML there, so instead we're going to create a
<i style="color: red">index.html</i> file and serve that instead.

Let’s refactor our route handler to use
<i style="color: red">sendFile</i> instead.

```js
app.get("/", (req, res) => {
  res.sendFile(__dirname + "/index.html");
});
```

Put the following in your
<i style="color: red">index.html</i> file:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Socket.IO chat</title>
    <style>
      body {
        margin: 0;
        padding-bottom: 3rem;
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
          Helvetica, Arial, sans-serif;
      }

      #form {
        background: rgba(0, 0, 0, 0.15);
        padding: 0.25rem;
        position: fixed;
        bottom: 0;
        left: 0;
        right: 0;
        display: flex;
        height: 3rem;
        box-sizing: border-box;
        backdrop-filter: blur(10px);
      }

      #input {
        border: none;
        padding: 0 1rem;
        flex-grow: 1;
        border-radius: 2rem;
        margin: 0.25rem;
      }

      #input:focus {
        outline: none;
      }
      #form > button {
        background: #333;
        border: none;
        padding: 0 1rem;
        margin: 0.25rem;
        border-radius: 3px;
        outline: none;
        color: #fff;
      }

      #messages {
        list-style-type: none;
        margin: 0;
        padding: 0;
      }
      #messages > li {
        padding: 0.5rem 1rem;
      }
      #messages > li:nth-child(odd) {
        background: #efefef;
      }
    </style>
  </head>
  <body>
    <ul id="messages"></ul>
    <form id="form" action="">
      <input id="input" autocomplete="off" /><button>Send</button>
    </form>
  </body>
</html>
```

If you restart the process and refresh the page it should look like this:

![03-img](/socket-chat-example/screenshot/Screen%20Shot%202022-04-04%20at%205.07.21%20PM.png)

## Integrating Socket.IO
Socket.IO is composed of two parts:

1. A server that integrates with (or mounts on) the Node.JS HTTP Server socket.io
2. A client library that loads on the browser side socket.io-client

During development, socket.io serves the client automatically for us, as we’ll see, so for now we only have to install one module:

```
npm install socket.io
```
That will install the module and add the dependency to package.json. 
Now let’s edit index.js to add it:

```js
const express = require('express');
const app = express();
const http = require('http');
const server = http.createServer(app);
const { Server } = require("socket.io");
const io = new Server(server);

app.get('/', (req, res) => {
  res.sendFile(__dirname + '/index.html');
});

io.on('connection', (socket) => {
  console.log('a user connected');
});

server.listen(3000, () => {
  console.log('listening on *:3000');
});
```
Now in index.html add the following snippet before the </body> (end body tag):

```html 
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io();
</script>
```

That’s all it takes to load the <i style="color: red">socket.io-client</i>, 
which exposes an io global (and the endpoint <i style="color: red">GET /socket.io/socket.io.js</i>), and then connect.

Notice that I’m not specifying any URL when 
I call <i style="color: red">io()</i>, since it defaults to trying to 
connect to the host that serves the page.

Try opening several tabs, and you’ll see several messages.

![04-img](/socket-chat-example/screenshot/Screen%20Shot%202022-04-04%20at%205.28.22%20PM.png)

Each socket also fires a special disconnect event:

```CS
io.on('connection', (socket) => {
  console.log('a user connected');
  socket.on('disconnect', () => {
    console.log('user disconnected');
  });
});
```

Then if you refresh a tab several times you can see it in action.

![05-img](/socket-chat-example/screenshot/Screen%20Shot%202022-04-04%20at%205.31.26%20PM.png)

##Emitting events
The main idea behind Socket.IO is that you can send and receive any events you want, with any data you want. Any objects that can be encoded as JSON will do, and binary data is supported too.

Let’s make it so that when the user types in a 
message, the server gets it as a chat message event. 
The script section in index.html should now look as follows:

```HTML
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io();

  const form = document.getElementById('form');
  const input = document.getElementById('input');

  form.addEventListener('submit', function(e) {
    e.preventDefault();
    if (input.value) {
      socket.emit('chat message', input.value);
      input.value = '';
    }
  });
</script>
```
And in index.js we print out the chat message event:

```JS
io.on('connection', (socket) => {
  socket.on('chat message', (msg) => {
    console.log('message: ' + msg);
  });
});
```
The result should be like this:

![06-img](/socket-chat-example/screenshot/Screen%20Shot%202022-04-04%20at%205.52.34%20PM.png)

## Broadcasting
The next goal is for us to emit the event from the server to the rest of the users.

In order to send an event to everyone, Socket.IO gives us the io.emit() method.

```js
io.emit('some event', { someProperty: 'some value', otherProperty: 'other value' }); // This will emit the event to all connected sockets
```
If you want to send a message to everyone except for a certain emitting socket, we have the broadcast flag for emitting from that socket:

```js
io.on('connection', (socket) => {
  socket.broadcast.emit('hi');
});
```
In this case, for the sake of simplicity we’ll send the message to everyone, including the sender.

```js
io.on('connection', (socket) => {
  socket.on('chat message', (msg) => {
    io.emit('chat message', msg);
  });
});
```
And on the client side when we capture a chat message event we’ll include it in the page. The total client-side JavaScript code now amounts to:

```html
<script src="/socket.io/socket.io.js"></script>
<script>
  var socket = io();

  var messages = document.getElementById('messages');
  var form = document.getElementById('form');
  var input = document.getElementById('input');

  form.addEventListener('submit', function(e) {
    e.preventDefault();
    if (input.value) {
      socket.emit('chat message', input.value);
      input.value = '';
    }
  });

  socket.on('chat message', function(msg) {
    var item = document.createElement('li');
    item.textContent = msg;
    messages.appendChild(item);
    window.scrollTo(0, document.body.scrollHeight);
  });
</script>
```
And that completes our chat application, in about 20 lines of code! This is what it looks like:

![07-img](/socket-chat-example/screenshot/Screen%20Shot%202022-04-04%20at%206.20.28%20PM.png)

We can improve the app with different features:

```text
1. Broadcast a message to connected users when someone connects or disconnects.
2. Add support for nicknames.
3. Don’t send the same message to the user that sent it. Instead, append the message directly as soon as he/she presses enter.
4. Add “{user} is typing” functionality.
5. Show who’s online.
6. Add private messaging.
7. Share your improvements!
8. We can by the user camera, show whot is online if he/she agree.
```