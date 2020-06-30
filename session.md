# HTTP Session

- [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
- [Using The Session](#using-the-session)
    - [Retrieving Data](#retrieving-data)
    - [Storing Data](#storing-data)
    - [Flash Data](#flash-data)
    - [Deleting Data](#deleting-data)
    
<a name="introduction"></a>
## Introduction

Since HTTP driven applications are stateless, sessions provide a way to store information about the user across multiple requests. hunt-framework ships with a variety of session backends that are accessed through an expressive, unified API. Support for popular backends such as [Memcached](https://memcached.org), [Redis](https://redis.io), and databases is included out of the box.

<a name="configuration"></a>
### Configuration

The session configuration file is stored at `config/application.conf`. Be sure to review the options available to you in this file. 

The session `driver` configuration option defines where session data will be stored for each request. 

<a name="driver-prerequisites"></a>
### Driver Prerequisites

#### Cache

For example, you may wish to use a cache driver in your project.

    hunt.cache.adapter = redis
    hunt.cache.prefix = hunt_cache:
    hunt.cache.expire = 7200
    hunt.cache.args = 127.0.0.1:6379
    hunt.cache.useSecondLevelCache = false

> {tip} More item value reference file [ApplicationConfig.d](https://github.com/huntlabs/hunt-framework/blob/master/source/hunt/framework/config/ApplicationConfig.d#L41)

> {tip} More reference file [reference](https://github.com/huntlabs/hunt-framework-docs/blob/master/cache.md)

<a name="using-the-session"></a>
## Using The Session

<a name="retrieving-data"></a>
### Retrieving Data

There are two primary ways of working with session data in hunt-framework: the global `session` helper and via a `Request` instance. First, let's look at accessing the session via a `Request` instance, which can be type-hinted on a controller method. Remember, controller method dependencies are automatically injected via the hunt-framework :

    request.session.get("KEY");

#### Retrieving All Session Data

If you would like to retrieve all the data in the session, you may use the `all` method:

    string[string] data = request.session.all();

#### Determining If An Item Exists In The Session

To determine if an item is present in the session, you may use the `exists` method:

    if (request.session.exists("KEY");) {
        //
    }

<a name="storing-data"></a>
### Storing Data

To store data in the session, you will typically use the `set` method :

    // Via a request instance...
    request.session.set("key", "value");

#### Pushing To Array Session Values

The `push` method may be used to push a new value onto a session value that is an array. For example, if the `teams` key contains an array of team names, you may push a new value onto the array like so:

    request.session.push("teams", "developers");

<a name="flash-data"></a>
### Flash Data

Sometimes you may wish to store items in the session only for the next request. You may do so using the `flash` method. Data stored in the session using this method will be available immediately and during the subsequent HTTP request. After the subsequent HTTP request, the flashed data will be deleted. Flash data is primarily useful for short-lived status messages:

    request.session.flash("status", "Task was successful!");

If you need to keep your flash data around for several requests, you may use the `reflash` method, which will keep all of the flash data for an additional request. If you only need to keep specific flash data, you may use the `keep` method:

    request.session.reflash();

    request.session.keep(['key1', 'key2']);

<a name="deleting-data"></a>
### Deleting Data

The `remove` method will remove a piece of data from the session:

    // Forget a single key...
    request.session.remove('key');
