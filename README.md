
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

**Principles & Requirements**
-------
**Coverage & Simplicity** - The stack should be available on every popular development platform and easy for someone to build for their platform of choice. It should be viable on CPU & memory limited devices.

**Payload Agnostic** - Different services need to use different message types and encodings such as protocol buffers, JSON, XML, and Thrift; the protocol and implementations must allow for this. Similarly the need for payload compression varies by use-case and payload type: the protocol should allow for pluggable compression mechanisms.

**Streaming** - Storage systems rely on streaming and flow-control to express large data-sets. Other services, like voice-to-text or stock-tickers, rely on streaming to represent temporally related message sequences.

**Blocking & Non-Blocking** - Support both asynchronous and synchronous processing of the sequence of messages exchanged by a client and server. This is critical for scaling and handling streams on certain platforms.


**what are Protocol Buffers and how are they working with gRPC**
-------
Protocol buffers are a mechanism for serializing structured data.  Protobuf message definitions have values, but no methods; they are data-holders. Protobuf messages are defined in .proto files.  

You can define a protobuf message in a .proto file like so:
```
syntax = "proto3";
package moviecatalog;
service MovieCatalog {
  rpc SaveNewMovie (MovieItem) returns (AddMovieResponse) {}
  rpc FetchExistingMovie (FetchMovieRequest) returns (MovieItem) {}
}
message MovieItem {
    string name = 1;
    double price = 2;
    bool inStock = 3;
}
message AddMovieResponse {
    bool wasSaved = 1;
    int32 itemId = 2;
}
message FetchMovieRequest {
    string name = 1;
}
```
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



**gRPC development flow**
-------
https://grpc.io/docs/languages/node/basics/


## 1. Defining the service 
To define a service, you specify a named service in your .proto file:  
Example: `bidirectional streaming RPC`
```
rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
```
## 2. Loading service descriptors from proto files
To load a `.proto` file, simply require the gRPC proto loader library and use its `loadSync()` method, then pass the output to the gRPC library’s `loadPackageDefinition` method:
```
var PROTO_PATH = __dirname + '/../../../protos/route_guide.proto';
var grpc = require('@grpc/grpc-js');
var protoLoader = require('@grpc/proto-loader');
// Suggested options for similarity to existing grpc.load behavior
var packageDefinition = protoLoader.loadSync(
    PROTO_PATH,
    {keepCase: true,
     longs: String,
     enums: String,
     defaults: true,
     oneofs: true
    });
var protoDescriptor = grpc.loadPackageDefinition(packageDefinition);
// The protoDescriptor object has the full package hierarchy
var routeguide = protoDescriptor.routeguide;
```
## 3. Creating the server 
```
var Server = new grpc.Server();

```
## 4. Implementing service methods
```
function checkFeature(point) {
  var feature;
  // Check if there is already a feature object for the given point
  for (var i = 0; i < feature_list.length; i++) {
    feature = feature_list[i];
    if (feature.location.latitude === point.latitude &&
        feature.location.longitude === point.longitude) {
      return feature;
    }
  }
  var name = '';
  feature = {
    name: name,
    location: point
  };
  return feature;
}
function getFeature(call, callback) {
  callback(null, checkFeature(call.request));
}
```
## 5. Starting the server
```
function getServer() {
  var server = new grpc.Server();
  server.addProtoService(routeguide.RouteGuide.service, {
    getFeature: getFeature,
    listFeatures: listFeatures,
    recordRoute: recordRoute,
    routeChat: routeChat
  });
  return server;
}
var routeServer = getServer();
routeServer.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
  routeServer.start();
});
```
## 6. Creating the client
```
var call = client.recordRoute(function(error, stats) {
  if (error) {
    callback(error);
  }
  console.log('Finished trip with', stats.point_count, 'points');
  console.log('Passed', stats.feature_count, 'features');
  console.log('Travelled', stats.distance, 'meters');
  console.log('It took', stats.elapsed_time, 'seconds');
});
```

**Protobuf vs JSON**
-------

**Protobuf** Protocol Buffers was internally a better way, compared to XML, for data serialization -deserialization. So they focused on making it simpler, smaller, faster and more maintainable then XML. But, this protocol even surpassed JSON with better performance, better maintainability, and smaller size.  

**JSON** (JavaScript Object Notation) is a lightweight data-interchange format and is based on a subset of the JavaScript Programming Language.



**gRPC Error Handling**
-------



**gRPC Middlewares**
-------


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
