# Queues

- [Introduction](#introduction)
    - [Connections vs. Queues](#connections-vs-queues)
- [Produce Jobs](#produce-jobs)
- [Consume Jobs](#consume-jobs)

<a name="introduction"></a>
## Introduction

>  Support for RabbitMQ and the other MQ based AMQP 1.0 protocol.

<a name="connections-vs-queues"></a>
### Connections vs. Queues

Before getting started with hunt-framework queues, it is important to understand the distinction between "connections" and "queues". In your `application.conf` configuration file, there is a `queue.driver` configuration option. This option defines a particular connection to a backend service such as memory, amqp, or Redis. However, any given queue connection may have multiple "queues" which may be thought of as different stacks or piles of queued jobs.

Note that each connection configuration example in the `amqp` configuration file contains a `amqp` attribute. This is the default queue that jobs will be dispatched to when they are sent to a given connection. In other words, if you dispatch a job without explicitly defining which queue it should be dispatched to, the job will be placed on the queue that is defined in the `amqp` attribute of the connection configuration:

```ini
queue.driver = amqp

amqp.enabled = true
amqp.host = 127.0.0.1
amqp.port = 5672
amqp.username = test
amqp.password = 123 
```

<a name="produce-jobs"></a>
## Produce Jobs
```d
    string message = "hello world";
    AbstractQueue queue  = app().queue();
    queue.push("my-queue", cast(ubyte[])message);
```

<a name="consume-jobs"></a>

## Consume Jobs
```d
    enum string ChannelName = "my-queue";
    FuturePromise!string promise = new FuturePromise!string();
    
    AbstractQueue queue  = app().queue();
    scope(exit) {
        queue.remove(ChannelName);
    }

    queue.addListener(ChannelName, (ubyte[] message) {
        string msg = cast(string)message;
        promise.succeeded(msg);
    });

    try {
        string message = promise.get(25.seconds);
        // do somthing
    } catch(Exception ex) {
        // Log the error
    }
```
