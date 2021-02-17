
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



**gRPC Error Handling**
-------
Ref: https://grpc.github.io/grpc/core/md_doc_statuscodes.html  

| Code                | Number | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |   |   |   |   |   |   |   |   |   |
|---------------------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---|---|---|---|---|---|---|---|---|
| OK                  | 0      | Not an error; returned on success.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |   |   |   |   |   |   |   |   |   |
| CANCELLED           | 1      | "The operation was cancelled, typically by the caller."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |   |   |   |   |   |   |   |   |   |
| UNKNOWN             | 2      | "Unknown error. For example, this error may be returned when a Status value received from another address space belongs to an error space that is not known in this address space. Also errors raised by APIs that do not return enough error information may be converted to this error."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |   |   |   |   |   |   |   |   |   |
| INVALID_ARGUMENT    | 3      | "The client specified an invalid argument. Note that this differs from FAILED_PRECONDITION. INVALID_ARGUMENT indicates arguments that are problematic regardless of the state of the system (e.g., a malformed file name)."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |   |   |   |   |   |   |   |   |   |
| DEADLINE_EXCEEDED   | 4      | "The deadline expired before the operation could complete. For operations that change the state of the system, this error may be returned even if the operation has completed successfully. For example, a successful response from a server could have been delayed long"                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |   |   |   |   |   |   |   |   |   |
| NOT_FOUND           | 5      | "Some requested entity (e.g., file or directory) was not found. Note to server developers: if a request is denied for an entire class of users, such as gradual feature rollout or undocumented allowlist, NOT_FOUND may be used. If a request is denied for some users within a class of users, such as user-based access control, PERMISSION_DENIED must be used."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |   |   |   |   |   |   |   |   |   |
| ALREADY_EXISTS      | 6      | "The entity that a client attempted to create (e.g., file or directory) already exists."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |   |   |   |   |   |   |   |   |   |
| PERMISSION_DENIED   | 7      | The caller does not have permission to execute the specified operation. PERMISSION_DENIED must not be used for rejections caused by exhausting some resource (use RESOURCE_EXHAUSTED instead for those errors). PERMISSION_DENIED must not be used if the caller can not be identified (use UNAUTHENTICATED instead for those errors). This error code does not imply the request is valid or the requested entity exists or satisfies other pre-conditions.                                                                                                                                                                                                                                                                                                                                                                                                                                             |   |   |   |   |   |   |   |   |   |
| RESOURCE_EXHAUSTED  | 8      | "Some resource has been exhausted, perhaps a per-user quota, or perhaps the entire file system is out of space."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |   |   |   |   |   |   |   |   |   |
| FAILED_PRECONDITION | 9      | The operation was rejected because the system is not in a state required for the operation's execution. For example, the directory to be deleted is non-empty, an rmdir operation is applied to a non-directory, etc. Service implementors can use the following guidelines to decide between FAILED_PRECONDITION, ABORTED, and UNAVAILABLE: (a) Use UNAVAILABLE if the client can retry just the failing call. (b) Use ABORTED if the client should retry at a higher level (e.g., when a client-specified test-and-set fails, indicating the client should restart a read-modify-write sequence). (c) Use FAILED_PRECONDITION if the client should not retry until the system state has been explicitly fixed. E.g., if an "rmdir" fails because the directory is non-empty, FAILED_PRECONDITION should be returned since the client should not retry unless the files are deleted from the directory. |   |   |   |   |   |   |   |   |   |
| ABORTED             | 10     | "The operation was aborted, typically due to a concurrency issue such as a sequencer check failure or transaction abort. See the guidelines above for deciding between FAILED_PRECONDITION, ABORTED, and UNAVAILABLE."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |   |   |   |   |   |   |   |   |   |
| OUT_OF_RANGE        | 11     | "The operation was attempted past the valid range. E.g., seeking or reading past end-of-file. Unlike INVALID_ARGUMENT, this error indicates a problem that may be fixed if the system state changes. For example, a 32-bit file system will generate INVALID_ARGUMENT if asked to read at an offset that is not in the range [0,2^32-1], but it will generate OUT_OF_RANGE if asked to read from an offset past the current file size. There is a fair bit of overlap between FAILED_PRECONDITION and OUT_OF_RANGE. We recommend using OUT_OF_RANGE (the more specific error) when it applies so that callers who are iterating through a space can easily look for an OUT_OF_RANGE error to detect when they are done."                                                                                                                                                                                 |   |   |   |   |   |   |   |   |   |
| UNIMPLEMENTED       | 12     | The operation is not implemented or is not supported/enabled in this service.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |   |   |   |   |   |   |   |   |   |
| INTERNAL            | 13     | Internal errors. This means that some invariants expected by the underlying system have been broken. This error code is reserved for serious errors.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |   |   |   |   |   |   |   |   |   |
| UNAVAILABLE         | 14     | "The service is currently unavailable. This is most likely a transient condition, which can be corrected by retrying with a backoff. Note that it is not always safe to retry non-idempotent operations."                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |   |   |   |   |   |   |   |   |   |
| DATA_LOSS           | 15     | Unrecoverable data loss or corruption.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |   |   |   |   |   |   |   |   |   |
| UNAUTHENTICATED     | 16     | The request does not have valid authentication credentials for the operation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |   |   |   |   |   |   |   |   |   |




**Load-Ballancing gRPC Services**
-------
## 1. Proxy load balancing  

 The LB keeps track of load on each backend and implements algorithms for distributing load fairly. The clients themselves do not know about the backend servers.

![alt text](./readme/image_0.png)


## 2. Client side load balancing  

In Client side load balancing, the client is aware of multiple backend servers and chooses one to use for each RPC. The client gets load reports from backend servers and the client implements the load balancing algorithms. 

![alt text](./readme/image_1.png)

**gRPC Middlewares**
-------


**Testing gRPC Services**
-------


**Drawbacks of using gRPC**
-------

## 1. Limited browser support

It's impossible to directly call a gRPC service from a browser today. gRPC heavily uses HTTP/2 features and no browser provides the level of control required over web requests to support a gRPC client. For example, browsers do not allow a caller to require that HTTP/2 be used, or provide access to underlying HTTP/2 frames.


## 2. Not human readable

gRPC messages are encoded with Protobuf by default. While Protobuf is efficient to send and receive, its binary format isn't human readable. Protobuf requires the message's interface description specified in the .proto file to properly deserialize.


**gRPC tooling**
-------

## gRPC command line tool


Ref: https://github.com/grpc/grpc/blob/master/doc/command_line_tool.md

Core functionality

- Send unary rpc.
- Attach metadata and display received metadata.
- Handle common authentication to server.
- Infer request/response types from server reflection result.
- Find the request/response types from a given proto file.
- Read proto request in text form.
- Read request in wire form (for protobuf messages, this means serialized binary form).
- Display proto response in text form.
- Write response in wire form to a file.

**Protobuf vs JSON**
-------

**Protobuf** Protocol Buffers was internally a better way, compared to XML, for data serialization -deserialization. So they focused on making it simpler, smaller, faster and more maintainable then XML. But, this protocol even surpassed JSON with better performance, better maintainability, and smaller size.  

**JSON** (JavaScript Object Notation) is a lightweight data-interchange format and is based on a subset of the JavaScript Programming Language.


**Implement GRPC Example**
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
   $ node ./dynamic_codegen/greeter_server.js
   ```

 - Run the client

   ```sh
   $ # from this directory
   $ node ./dynamic_codegen/greeter_client.js
   ```

TUTORIAL
--------
You can find a more detailed tutorial in [gRPC Basics: Node.js][]

[Install gRPC Node]:../../src/node
[gRPC Basics: Node.js]:https://grpc.io/docs/tutorials/basic/node.html
