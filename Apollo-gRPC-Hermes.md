## Apollo, gRPC, Hermes
- [Apollo](#apollo)
- [gRPC](#grpc)

### Apollo
- Apollo: manages modules lifecycle. A module can be a server or anything
```java
final Service service = 
  Services.usingName(SERVICE_NAME)
    .withModule(...)
    .withEnvVarPrefix("")
    ...
    build();
StandaloneService.boot(service, args);
```
- Apollo API - Environment
  - `Environment.resolve()`: resolves an instance of a class out of the underlying module system
  - `.config()`: get all the config values
  - `.closer()`: shutting down the application gracefully
  - `.routingEngine()`: register Hermes endpoints
- Config file:
  - `<service-name>.conf`
  - run locally, may not want to use production secret/database, use `<service-name>-user.conf` will override the production config
  ```protobuf
  grpc.server {
    port: 5990
    healthService.enabled: true
  }
  hermes.server {
    port: 5700
  }
  ```
- Service Auth
  - Authentication: proven identity of the calling service
  - Authorization: control what different calling services have access to
  ```protobuf
  serviceauth {
    enabled: true
    enforcing: true
    defaultAction: "deny"
    accessPolicies: [
      { matchType: "prefix", path: "/_meta/0/", method: "GET", principals: [] },
      ...
    ]
  ```
- Apollo as an Application Server: need to add one or more of the following modules
  - GrpcServerModule
  - HttpServerModule
  - HermesServerModule
- Apollo as a clinet: enable Apollo making outgoing calls, need to add one or more of the following modules
  - GrpcChannelFactoryModule
  - HermesClientModule
  - Currently has five different Google data centers, set with `apollo.domain`. It's set automatically in production.

### gRPC
- google Remote Procedure Call
  - running a function on a remote system and get back the result
- Open source
- HTTP/2 transport
- Protobuf schema
  - Alternative to JSON. More high-performance
  - Predefine the entire schema
  - Serializing structured data
  ```protobuf
  message PersonName {
    string first_name = 1;
    repeated string middle_name 2;
    string family_name = 3;
  }
  ```
  - All fields are optional
  - Need to pick a number that hasn't been used before. Don't need to be in sequential
  ```protobuf
   syntax = "proto3"
   
   service Greeter {   // Interface for a Greeter service
    rpc SayHello (HelloRequest) returns (HelloReply){}    // One rpc, with parameters and results
   }
   message HelloRequest {            // Message/Type definition
    string name = 1;                 // a string field 'name'
   }
   message HelloReply {
    string message = 1;
   }
   ```
- protoc: Protobuf compiler
   - compiles into many languages
   - Maven plugin: `mvn generate-sources` will invoke protoc and compile all the schemas to java codes
   - Example
     ```protobuf
     service Greeter {
      rpc SayHello (HelloRequest) returns (HelloReply) {}
     }
     ```
     will get 
      - HelloRequest.java, HelloReply.java
      - GreeterGrpc.java
- gRPC and futures
  - Java - CompletableFuture
  - gRPC - ListenableFuture (Guaba)
- gRPC Service Implementation
  ```java
  class GreeterImpl extends GreeterGrpc.GreeterImplBase
  ```
  - Implement a method for each RPC
  
    **.proto**
    ```protobuf
    rpc SayHello (HelloRequest) returns (HelloReply) {}
    ```
    **.java**
    ```java
    public void sayHello(
      HelloRequest request,
      StreamObserver<HelloReply> responseObserver) {...}
    ```
  - StreamObserver<T>: a way to send back a reply
- gRPC Service Concat in proto
  ```protobuf
  syntax = "proto3";
  
  message ConcatRequest {
    string string_one = 1;
    string string_two = 2;
  }
  message ConcatResponse {
    string response = 1;
  }
  service ConcatService {
    rpc Concat(ConcatRequest) returns (ConcatResponse) {}
  }
  ```
- gRPC Service Concat in Java
```java
  