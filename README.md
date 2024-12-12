full duplex communication: each peer can send and recieve at the same time

syn, syn-ack, ack - tcp uses this three way handshake. synchronize and acknowledge


fin flag to finish communication. so fin then ack.
each one can stop independantly so the other side also does the same handshake to fully close the connection.

# socket primitives
## applications refer to sockets by opaque os handles
tcp connection is handled by your os system. you use the socket handle to refer to the connection in the socket api. what is the socket handle? In Linux, a socket handle is simply a file descriptor (fd). whats a file descriptor? like .py?
in node js sock hands are wrapped into a js obj with methods on them
any os handle needs to be closed to recycle the handle.

tcp server listens on a particular address (ip + port) and accepts client connections from
that addy. listening addy is also represented by a socket handle. when accepting a new client connection you get the socket handle of the tcp connection

2 types of socket handles: listening and connection sockets.

end of transmission
closing a socket terminates a connection and sends tcp fin. closing a handle of any type also recycles the handle itself.

you can also shut down your side of the transmission also sending fin. this makes a half open connection because you can still recieve info from them.

end of transmission is also called the eof

## list of socket primitive

listening sockets:
- bind and listen
- accept
- close
connection socket:
- read
- write
- close

# socket api in node.js

## step 1 make a listening socket

its all in the net module

``` import * as net from 'net' ```

net.createServer() function makes a listening socket whose type is net.Server.net.Server, has a listen() method to bind and listen to an address.

```JavaScript
let server = net.createServer();
server.listen({host: '127.0.0.1', port: 1234});
```

side note: node.js is a javascript run time environment that lets you
execute js code without a browser

The No. 1 beginner trap in socket programming is “concatenating & splitting TCP packets” because there is no such thing as “TCP packets”. Protocols are required to interpret TCP data by imposing boundaries within the byte stream.

## step 2 accept a new connection

accept primitive - this needs some background knowledge.
2 styles of handling IO in js: callbacks - you request something to be done and register a callback with the
run time and when finished the callback is invoked. E.G.

```typescript
function newConn(socket: net.Socket): void {
    console.log('new connection', socket.remoteAddress, socket.remotePort);
    // ...
}

let server = net.createServer();
server.on('connection', newConn); // this registers the callback function newConn.
server.listen({host: '127.0.0.1', port: 1234});
```

the run time will automatically perform the accept operation and invoke the callback with the new connection as an argument of type net.Socket. its registered once but will be used for each new connection.

## step 3 error handling

connections are events and we register things using socket.on to a specific event. ex ->
```js
server.on('error', (err)=> {throw err});
```

## step 4 read and write

data received via connections is also delivered via callbacks. events are 'data' and 'end'. 'data' is when data arrives and 'end' is when the peer has ended the transmission.

```ts
    socket.on('end', () => {
        // FIN received. The connection will be closed automatically.
        console.log('EOF.');
    });
    socket.on('data', (data: Buffer) => {
        console.log('data:', data);
        socket.write(data); // echo back the data.
    });
```

what is the buffer type?

the socket.write() method sends data back to the peer

## step 5 close the connection

socket.end ends the transmission and closes the socket. ex shows ending the socket comms when q is sent

```ts
socket.on('data', (data: Buffer) => {
        console.log('data:', data);
        socket.write(data); // echo back the data.

        // actively closed the connection if the data contains 'q'
        if (data.includes('q')) {
            console.log('closing.');
            socket.end();   // this will send FIN and close the connection.
        }
    });
```

when the transmission is ended from either side the socket is automatically closed by the runtime. 'error' event also closes the socket

## step 6 test it
complete code example


```ts
import * as net from "net";

function newConn(socket: net.Socket): void {
    console.log('new connection', socket.remoteAddress, socket.remotePort);

    socket.on('end', () => {
        // FIN received. The connection will be closed automatically.
        console.log('EOF.');
    });
    socket.on('data', (data: Buffer) => {
        console.log('data:', data);
        socket.write(data); // echo back the data.

        // actively closed the connection if the data contains 'q'
        if (data.includes('q')) {
            console.log('closing.');
            socket.end();   // this will send FIN and close the connection.
        }
    });
}

let server = net.createServer();
server.on('error', (err: Error) => { throw err; });
server.on('connection', newConn);
server.listen({host: '127.0.0.1', port: 1234});
```

Start the echo server by running ```node --enable-source-maps echo_server.js```. And test it with the ```nc``` or ```socat``` command.


# 3.4 Discussion: Half-Open Connections
Each direction of a TCP connection is ended independently, and it is possible to make use of the state where one direction is closed and the other is still open; this unidirectional use of TCP is called TCP half-open. For example, if peer A half-closes the connection to peer B:

- A cannot send any more data, but can still receive from B.
- B gets EOF, but can still send to A.

Not many applications make use of this. Most applications treat EOF the same way as being fully closed by the peer, and will also close the socket immediately.

The socket primitive for this is called ```shudown```. Sockets in Node.js do not support half-open by default, and are automatically closed when either side sends or receives EOF. To support TCP half-open, an additional flag is required.

```
let server = net.createServer({allowHalfOpen: true});
```

When the ```allowHalfOpen``` flag is enabled, you are responsible for closing the connection, because ```socket.end()``` will no longer close the connection, but will only send EOF. Use ```socket.destroy()``` to close the socket manually.

so a server is just a script that is listening for requests?
look into load balancer and types of db blob vs relational and system design stuff

what determines the speed of the internet? i know theres a modem and router and a coaxial cable that are all talking to one another
but what is the limiting factor and when is it the limiting factor? what determines how many devices can be connected to the router
at once and what determines how much internet data can be sent out from the router? is it a wifi card?

this is a test?

this weekend our goal is to finish taking these notes and write it out. from what i understand a web server is super simple to actually code this is more practicing the understanding

reading a hacking book using c and going to attempt to flash a burner phone

continuing to read this book today
