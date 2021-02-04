
**ACKNOWLEDGE:**
-------
**What is gRPC**
-------
A general gRPC (Remote Procedure Call) framework over HTTP/2, initially developed at Google. gRPC is a language-neutral and platform-neutral framework that allows users to write applications where independent services can work with each other as if they were native. It uses Protocol Buffers as the interface description language.

**Why we should use GRPC instead of Restfull API as a microservices communication?**
-------
Early microservices implementations leveraged Representational State Transfer (REST) architecture as the de-facto communication technology. However, RESTful services are often useful for external-facing services, which are directly exposed to consumers. As they are based on conventional text-based messaging (JSON, XML, CVS over HTTP, etc.), which are optimized for humans, these are not ideal choices for internal service-to-service communication.

**Is gRPC better than REST?**
-------
First, gRPC uses HTTP/2 which is, as you know, much faster than HTTP/1.1 used in REST by default. Note that today we can enable HTTP/2 in REST as well, but normally it often goes with HTTP/1.1. 

Second, gRPC uses Protocol buffer to serialize payload data, which is binary and smaller, while REST uses JSON, which is text and larger.

**Protobuf vs JSON**
-------

**Protobuf** Protocol Buffers was internally a better way, compared to XML, for data serialization -deserialization. So they focused on making it simpler, smaller, faster and more maintainable then XML. But, this protocol even surpassed JSON with better performance, better maintainability, and smaller size.  

**JSON** (JavaScript Object Notation) is a lightweight data-interchange format and is based on a subset of the JavaScript Programming Language.

**gRPC Communication Type**
-------
When working with gRPC we have 4 different types of communication between client and server. 
## Unary RPCs: 
Where the client sends a single request to the server and gets a single response back, just like a normal function call.

## Server streaming RPCs
Where the client sends a request to the server and gets a stream to read a sequence of messages back.
## Client streaming RPCs
Where the client writes a sequence of messages and sends them to the server, again using a provided stream.
## Bidirectional streaming RPCs
Where both sides send a sequence of messages using a read-write stream.


**How to apply GRPC to Microservices Structure**
-------
Server
-------
```

var PROTO_PATH = __dirname + '/../protos/helloworld.proto';

var grpc = require('grpc');
var protoLoader = require('@grpc/proto-loader');
var packageDefinition = protoLoader.loadSync(
    PROTO_PATH,
    {keepCase: true,
     longs: String,
     enums: String,
     defaults: true,
     oneofs: true
    });
var hello_proto = grpc.loadPackageDefinition(packageDefinition).helloworld;

/**
 * Implements the SayHello RPC method.
 */
function sayHello(call, callback) {
  callback(null, {message: 'Hello ' + call.request.name});
}

/**
 * Starts an RPC server that receives requests for the Greeter service at the
 * sample server port
 */
function main() {
  var server = new grpc.Server();
  server.addService(hello_proto.Greeter.service, {sayHello: sayHello});
  server.bind('0.0.0.0:50051', grpc.ServerCredentials.createInsecure());
  server.start();
}

main();

```

Client
-------

```
var PROTO_PATH = __dirname + '/../protos/helloworld.proto';

var grpc = require('grpc');
var protoLoader = require('@grpc/proto-loader');
var packageDefinition = protoLoader.loadSync(
    PROTO_PATH,
    {keepCase: true,
     longs: String,
     enums: String,
     defaults: true,
     oneofs: true
    });
var hello_proto = grpc.loadPackageDefinition(packageDefinition).helloworld;

function main() {
  var client = new hello_proto.Greeter('localhost:50051',
                                       grpc.credentials.createInsecure());
  var user;
  if (process.argv.length >= 3) {
    user = process.argv[2];
  } else {
    user = 'world';
  }
  client.sayHello({name: user}, function(err, response) {
    console.log('Greeting:', response.message);
  });
}

main();

```


TRY IT!
-------

There are two ways to generate the code needed to work with protocol buffers in Node.js - one approach uses [Protobuf.js](https://github.com/dcodeIO/ProtoBuf.js/) to dynamically generate the code at runtime, the other uses code statically generated using the protocol buffer compiler `protoc`. The examples behave identically, and either server can be used with either client.

 - Run the server

   ```sh
   $ # from this directory
   $ node ./dynamic_codegen/greeter_server.js &
   $ # OR
   $ node ./static_codegen/greeter_server.js &
   ```

 - Run the client

   ```sh
   $ # from this directory
   $ node ./dynamic_codegen/greeter_client.js
   $ # OR
   $ node ./static_codegen/greeter_client.js
   ```

TUTORIAL
--------
You can find a more detailed tutorial in [gRPC Basics: Node.js][]

[Install gRPC Node]:../../src/node
[gRPC Basics: Node.js]:https://grpc.io/docs/tutorials/basic/node.html
