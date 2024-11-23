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

its all in the net module

``` import * as net from 'net' ```

net.createServer() function makes a listening socket whose type is net.Server.net.Server, has a listen() method to bind and listen to an address.

```JavaScript
let server = net.createServer();
server.listen({host: '127.0.0.1', port: 1234});
```

this 
