# gRPC

- [Introduction](#introduction)
- [Setup](#setup)
    - [Defining the service](#define-service)
    - [Generating client and server code](#generating-code)
    - [Implementing the service](#implementing)
- [Creating the server](#creating-server)
    - [Configuration](#configuration-server)
    - [Registering the service](#registering-service)
- [Creating the client as a channel](#creating-client)
    - [Configuration](#configuration-channel)
    - [Creating a stub and call service methods](#calling-service)
- [Try it out!](#try)

<a name="introduction"></a>
## Introduction

From Hunt Framework 3.4, a new component named GRPC is added, which provides a convenient way for programming with RPC.

<a name="setup"></a>
## Setup

You should have already installed the tools needed to generate client and server interface code – if you haven’t, see the [here](https://github.com/huntlabs/grpc-dlang).

<a name="define-service"></a>
### Defining the service

Our first step (as you’ll know from the [Introduction to gRPC](https://grpc.io/docs/what-is-grpc/introduction/)) is to define the gRPC service and the method `request` and `response` types using [protocol buffers](https://developers.google.com/protocol-buffers/docs/overview). For the complete `.proto` file, see [helloworld.proto](https://github.com/huntlabs/hunt-examples/blob/master/website-basic/protos/helloworld.proto).

```proto
syntax = "proto3";

option java_multiple_files = true;
option java_package = "io.grpc.examples.helloworld";
option java_outer_classname = "HelloWorldProto";
option objc_class_prefix = "HLW";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}

  // Sends another greeting
  rpc SayGoodBye (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

<a name="generating-code"></a>
### Generating client and server code

Next we need to generate the gRPC client and server interfaces from our `.proto` service definition. We do this using the protocol buffer compiler `protoc` with a special gRPC D plugin. This is similar to what we did in the [grpc-dlang](https://github.com/huntlabs/grpc-dlang).

From the `protos` directory, run the following command:

```sh
protoc --plugin=/usr/local/bin/protoc-gen-d --d_out=../source -I ./ helloworld.proto
protoc --plugin=protoc-gen-grpc=/usr/local/bin/grpc_dlang_plugin -I ./ --grpc_out=../source/helloworld helloworld.proto
```

Running this command generates the following files in the `helloworld` directory:
- `helloworld.d`, which contains all the protocol buffer code to populate, serialize, and retrieve request and response message types.
- `helloworldrpc.d`, which contains the following:
    An interface type (or stub) for clients to call with the methods defined in the `Greeter` service.
    An interface type for servers to implement, also with the methods defined in the `Greeter` service.

<a name="implementing"></a>
### Implementing `Greeter`
In the `source` directory, create a file named `GreeterImpl.d`:

```d
module GreeterImpl;

import hunt.logging;
import helloworld.helloworld;
import helloworld.helloworldrpc;
import grpc;

class GreeterImpl : GreeterBase {
    override Status SayHello(HelloRequest request, ref HelloReply reply) {
        reply.message = "Hello " ~ request.name;
        tracef("request: %s, reply: %s", request.name, reply.message);
        return Status.OK;
    }

    override Status SayGoodBye(HelloRequest request, ref HelloReply reply) {
        reply.message = "Bye " ~ request.name;
        tracef("request: %s, reply: %s", request.name, reply.message);
        return Status.OK;
    }
}
```

<a name="creating-server"></a>
## Creating the server

<a name="configuration-server"></a>
### Configuration

Open the configuration file `application.conf` for your Hunt Framework application, and add the settings.

```ini
# gRPC
grpc.server.enabled = true
grpc.server.host = 127.0.0.1
grpc.server.port = 50051
grpc.server.workerThreads = 4
```

<a name="registering-service"></a>
### Registering the service

Open the `main` file of the Hunt application:

```d
import hunt.framework;
import GreeterImpl;

void main(string[] args) {
    app.booting(() {
        app.grpc.server.register( new GreeterImpl());
    });
}
```

<a name="creating-client"></a>
## Creating the client as a channel

In this section, we’ll look at creating a D client for our `Greeter` service. 

<a name="configuration-channel"></a>
### Configuration

In the configuration file `application.conf`, you can add some settings for one or more gRPC clients:

```ini
grpc.clientChannels[0].name = ch1
grpc.clientChannels[0].host = 127.0.0.1
grpc.clientChannels[0].port = 50051
grpc.clientChannels[0].timeout = 15000

grpc.clientChannels[1].name = ch2
grpc.clientChannels[1].host = 127.0.0.1
grpc.clientChannels[1].port = 50052
grpc.clientChannels[1].timeout = 15000
```

Then you can use the channel name to get one gRPC client. 

<a name="calling-service"></a>
### Creating a stub and call service methods

To call service methods, we first need to create a gRPC channel to communicate with the server. Once the gRPC channel is setup, we need a client stub to perform RPCs. We get it by creating a `GreeterClient` instance which is provided by the `helloworld` package generated from the example `.proto` file. [Here](https://github.com/huntlabs/hunt-examples/blob/master/website-basic/source/app/controller/IndexController.d#L748) is the complete example client code.

```d
import hunt.logging;
import grpc;
import helloworld.helloworld;
import helloworld.helloworldrpc;

class IndexController : Controller {
    mixin MakeController;

    @Action
    string testGrpc() {
        string[string] queryParameters = request.queries();
        string channelName = request.queries()["channel"];

        try {
            Channel channel = app.grpc.getChannel(channelName);
            scope(exit) {
                channel.destroy();
            }

            GreeterClient client = new GreeterClient(channel);
            HelloRequest request = new HelloRequest();
            request.name = "Hunt";   

            HelloReply replyHello = client.SayHello(request);
            tracef("++++++++++++++++%s", replyHello.message);

            HelloReply replyBye = client.SayGoodBye(request);
            tracef("++++++++++++++++%s", replyBye.message);

            return channelName ~ " passed";
        } catch(Exception ex) {
            return ex.msg;
        }
    }
}
```

<a name="try"></a>
## Try it out!

1. Build the Hunt Framework application, and run the server.

    ```sh
    2021-Feb-07 14:57:26.399074 | 15835 | info | register.__lambda1 | gRPC server started at 0.0.0.0:50051. | ../../hunt-framework/source/hunt/framework/provider/GrpcServiceProvider.d:41
    ```

2. Visit the server in the browser or `curl`.

    ```sh
    $ curl 'http://127.0.0.1:8080/grpc?channel=ch1'
    ```

You’ll see output in the server's terminal like this:

```sh
2021-Feb-07 14:57:41.59891 | 15854 | trace | GreeterImpl.SayHello | request: Hunt, reply: Hello Hunt | source/GreeterImpl.d:15
2021-02-07 14:57:41 (15892) [debug] testGrpc - ++++++++++++++++Hello Hunt - source/app/controller/IndexController.d:788
2021-Feb-07 14:57:41.6429258 | 15854 | trace | GreeterImpl.SayGoodBye | request: Hunt, reply: Bye Hunt | source/GreeterImpl.d:21
2021-02-07 14:57:41 (15892) [debug] testGrpc - ++++++++++++++++Bye Hunt - source/app/controller/IndexController.d:791
```
