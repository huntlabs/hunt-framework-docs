# Queues

- [Introduction](#introduction)
    - [Connections Vs. Queues](#connections-vs-queues)
- [Produce Jobs](#produce-jobs)
- [Consume Jobs](#consume-jobs)

<a name="introduction"></a>
## Introduction

>  Support for RabbitMQ and the other MQ based amqp2.0 protocol.

<a name="connections-vs-queues"></a>
### Connections Vs. Queues

Before getting started with hunt-framework queues, it is important to understand the distinction between "connections" and "queues". In your `application.conf` configuration file, there is a `queue.driver` configuration option. This option defines a particular connection to a backend service such as memory, amqp, or Redis. However, any given queue connection may have multiple "queues" which may be thought of as different stacks or piles of queued jobs.

Note that each connection configuration example in the `amqp` configuration file contains a `amqp` attribute. This is the default queue that jobs will be dispatched to when they are sent to a given connection. In other words, if you dispatch a job without explicitly defining which queue it should be dispatched to, the job will be placed on the queue that is defined in the `amqp` attribute of the connection configuration:

 >  queue.driver = amqp


<a name="produce-jobs"></a>
## Produce Jobs

        string message = "hello world";
        AbstractQueue queue  = messageQueue();
        queue.push("my-queue", cast(ubyte[])message);

<a name="consume-jobs"></a>

## Consume Jobs

        enum string ChannelName = "my-queue";
        FuturePromise!string promise = new FuturePromise!string();
        string registTime = hunt.util.DateTime.DateTime.getTimeAsGMT();
        
        AbstractQueue queue  = messageQueue();
        scope(exit) {
            queue.remove(ChannelName);
        }

        queue.addListener(ChannelName, (ubyte[] message) {
            string msg = cast(string)message;
            warningf("Received message: %s", msg);
            promise.succeeded(msg);
        });

        string resultContent;
        Response response = new Response();
        resultContent = format("Listener for channel %s registed at %s", ChannelName, registTime);

        try {
            string message = promise.get(25.seconds);

            resultContent ~= "<br>\nThe received message: " ~ message;
        } catch(Exception ex) {
            resultContent ~= "<br>\nThe error message: " ~ ex.msg;
        }

