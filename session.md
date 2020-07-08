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

Since HTTP driven applications are stateless, sessions provide a way to store information about the user across multiple requests. hunt-framework ships with a variety of session backends that are accessed through an expressive, unified API. Support for popular backends such as [Redis](https://redis.io), and memory is included out of the box. That's because the session storage depends on the [Hunt-Cache](cache.md).

<a name="configuration"></a>
### Configuration

The configuration for session is stored in `config/application.conf`. Be sure to review the options available to you in this file. If the backend's settings will use the ones for [Cache](cache.md).

```ini
# Session
session.prefix = huntsession_
session.expire = 3600

# redis, memory
cache.adapter = redis
cache.prefix = huntcache_
cache.expire = 3600
```

> {tip} More item value reference file [ApplicationConfig.d](https://github.com/huntlabs/hunt-framework/blob/master/source/hunt/framework/config/ApplicationConfig.d#L41)

> {tip} More reference file [reference](https://github.com/huntlabs/hunt-framework-docs/blob/master/cache.md)

<a name="using-the-session"></a>
## Using The Session

<a name="retrieving-data"></a>
### Retrieving Data

There are two primary ways of working with session data in hunt-framework: the global `session` helper and via a `Request` instance. First, let's look at accessing the session via a `Request` instance, which can be type-hinted on a controller method. Remember, controller method dependencies are automatically injected via the hunt-framework :
```d
    request.session.get("KEY");
```

#### Retrieving All Session Data

If you would like to retrieve all the data in the session, you may use the `all` method:

    string[string] data = request.session.all();

#### Determining If An Item Exists In The Session

To determine if an item is present in the session, you may use the `exists` method:
```d
    if (request.session.exists("KEY");) {
        //
    }
```

<a name="storing-data"></a>
### Storing Data

To store data in the session, you will typically use the `set` method :

    // Via a request instance...
    request.session.set("key", "value");

#### Pushing To Array Session Values

The `push` method may be used to push a new value onto a session value that is an array. For example, if the `teams` key contains an array of team names, you may push a new value onto the array like so:

```d
    request.session.push("teams", "developers");
```

<a name="deleting-data"></a>
### Deleting Data

The `remove` method will remove a piece of data from the session:

```d
    // Forget a single key...
    request.session.remove('key');
```
