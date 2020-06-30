# Logging

- [Introduction](#introduction)
- [Configuration](#configuration)
    - [Building Log Stacks](#building-log-stacks)
- [Writing Log Messages](#writing-log-messages)

<a name="introduction"></a>
## Introduction

To help you learn more about what's happening within your application, hunt-framework provides robust logging services that allow you to log messages to files, the system error log, and even to Slack to notify your entire team.

<a name="configuration"></a>
## Configuration

All of the configuration for your application's logging system is housed in the `config/application.conf` configuration file. This file allows you to configure your application's log channels, so be sure to review each of the available channels and their options. We'll review a few common options below.

    hunt.log.level= DEBUG
    hunt.log.path= /root
    hunt.log.file= log.txt
    hunt.log.maxSize = 8M
    hunt.log.maxNum= 10
    
<a name="building-log-stacks"></a>
### Building Log Stacks 

    void main()
    {
      writeln("Edit source/app.d to start your project.");
      LogConf conf;
      logLoadConf(conf);

      auto user = new User();
      user.id = 10;
      user.name = "2123";
      user.email = "18601699402@163.com34";
      user.addr = "tianlin road";
      user.age = 18;
      auto context = user.valid();
      logDebug("Valid Context : ",context);
    }
    
<a name="writing-log-messages"></a>
## Writing Log Messages

You may write information to the logs using the `hunt.logging` . As previously mentioned, the logger provides the eight logging levels defined in the `hunt.logging.ConsoleLogger`

    alias logDebug = trace;
    alias logDebugf = tracef;
    alias logInfo = info;
    alias logInfof = infof;
    alias logWarning = warning;
    alias logWarningf = warningf;
    alias logError = error;
    alias logErrorf = errorf;
    
So, you may call any of these methods to log a message for the corresponding level. By default, the message will be written to the default log channel as configured by your `config/application.conf` configuration file:    
   
    class IpFilterMiddleware : MiddlewareInterface {
        string name() {
            return IpFilterMiddleware.stringof;
        }

        override Response onProcess(Request req, Response res) {
            string path = req.path();
            infof("path: %s, post name: %s", path, req.post("name"));
            return null;
        }
    }   
    
 
