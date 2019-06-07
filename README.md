# NodeJS-Notes

___

## Node.js Basics

Much of the Node.js core API is built around an idiomatic asynchronous event-driven architecture in which certain kinds of objects (called "emitters") emit named events that cause Function objects ("listeners") to be called.

The Node Basics:
1. Instantiate an HTTP server with a request handler function, and have it listen on a port.
2. Get headers, URL, method and body data from request objects. Handle stream errors on the request stream.
3. Make routing decisions based on URL and/or other data in request objects. Or use `request.pipe(response)` to pipe data from request objects and to response objects in response to `POST` requests.
4. Send headers, HTTP status codes and body data via response objects. Handle stream errors on the response stream.




**Example:** A simple Node server
```js
const http = require('http');

const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at port: ${port}/`);
});
```
This will print a "Hello World" message on the screen when a user visits `http://localhost:3000`.

___

## Events

* All objects that emit events are instances of the `EventEmitter` class. 

  These objects expose an `myEmitter.on()` function that allows one or more functions to be attached to named events emitted by the object.

  When the `EventEmitter` object emits an event, all of the functions attached to that specific event are called synchronously.

  Any values returned by the called listeners are ignored and will be discarded.

* The `myEmitter.on()` method is used to register listeners, while the `myEmitter.emit()` method is used to trigger the event.

#### Event Methods

```js
myEmitter.emit(eventName, [ ...args])
```
Synchronously calls each of the listeners registered for the event named `eventName`, in the order they were registered, passing the supplied arguments to each.

```js
myEmitter.on(eventName, listenerFunc)
```
Adds the `listenerFunc` to the end of the listeners array for the event named `eventName`.
  
```js
const myEmitter = new MyEmitter();

myEmitter.on('error', (err) => {
  console.error('whoops! there was an error');
});

myEmitter.emit('error', new Error('whoops!'));    // Prints: whoops! there was an error
```
If an `EventEmitter` does _not_ have at least one listener registered for the `error` event, and an `error` event is emitted, the error is thrown, a stack trace is printed, and the Node.js process exits.

___

## Streams

A `stream` is an abstract interface for working with streaming data in Node.js.

Streams can be readable, writable, or both. All streams are instances of `EventEmitter`.

**Example:** Node.js application that's using streams and implements an HTTP server:
```js
const http = require('http');

const server = http.createServer((req, res) => {
  // `req` is an http.IncomingMessage, which is a Readable Stream
  // `res` is an http.ServerResponse, which is a Writable Stream

  let body = '';

  // Readable streams emit 'data' events once a listener is added
  req.on('data', (chunk) => {
    body += chunk;
  });

  // The 'end' event indicates that the entire body has been received
  req.on('end', () => {
    const data = JSON.parse(body);
    res.write(typeof data);       // Write back something interesting to the user
    res.end();
   }
  });
});

server.listen(1337);
```

`myWritable.end(chunk, encoding, callbackFunc)` all arguments are optional
Signals that no more data will be written to the Writable Stream named `myWritable`.

`writable.write(chunk, encoding, callbackFunc)` all arguments are optional
Writes some data to the stream, and calls the supplied callback once the data has been fully handled.

## HTTP Transaction Anatomy

* HTTP Methods, URL and Headers In A Request
```js
  const { method, url } = request;
  
  const { headers } = request;
  const userAgent = headers['user-agent'];
```
When handling a request, Node places these properties in the `request` object.
  `method` : GET/POST/PUT/DELETE
  `url` : The full URL without the server, protocol or port. 
  `headers` : Is an object in `request` containing the header information for the specific request.


* Request Body
When receiving a `POST` or `PUT` request, the request body might be important to your application.

The `request` object that's passed in to a handler implements the `ReadableStream` interface.

We can grab the data right out of the stream by listening to the stream's `data` and `end` events.

The chunk emitted in each `data` event is a `Buffer`. If you know it's going to be string data, the best thing to do is collect the data in an array, then at the `end`, concatenate and stringify it.

```js
let body = [];
request.on('data', (chunk) => {
  body.push(chunk);
}).on('end', () => {
  body = Buffer.concat(body).toString();
  // at this point, `body` has the entire request body stored in it as a string
});
```

* Sending The Header Data For A Response
`writeHead()` 
Explicitly writes the headers and status code to the response stream.

```js
response.writeHead(200, {
  'Content-Type': 'application/json',
  'X-Powered-By': 'bacon'
});
```

* Sending Response Body
Since the `response` object is a `WritableStream`, writing a response body out to the client is just a matter of using the usual stream methods.

```js
response.write('<html>');
response.write('<body>');
response.write('<h1>Hello, World!</h1>');
response.write('</body>');
response.write('</html>');
response.end();
```
