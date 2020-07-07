# Hunt Cache
Universal cache library for D programming language.

- [Install](#install)
- [Configuration](#configuration)
- [Support backend](#support-backend)
- [Versions](#versions)
- [Tips](#tips)
- [Using Memory adapter](#memory-adapter)
- [Using Redis adapter](#redis-adapter)
- [For a struct or a class](#struct-class)
- [Full example](#full-example)
- [Helper in the framework](#helper)


<a name="install"></a>
## Install
To use this package, run the following command in your project's root directory:

```sh
dub add hunt-cache
```

<a name="configuration"></a>
## Configuration

hunt-framework provides an expressive, unified API for various caching backends. The cache configuration is located at `config/application.conf`. In this file you may specify which cache driver you would like to be used by default throughout your application. hunt-framework supports popular caching backends like [Memcached](https://memcached.org) and [Redis](https://redis.io) out of the box.

The cache configuration file also contains various other options, which are documented within the file, so make sure to read over these options. By default, hunt-framework is configured to use the `file` cache driver, which stores the serialized, cached objects in the filesystem. For larger applications, it is recommended that you use a more robust driver such as Memcached or Redis. You may even configure multiple cache configurations for the same driver.

```ini
hunt.cache.adapter = redis
hunt.cache.prefix = hunt_cache_
hunt.cache.expire = 3600
hunt.cache.args = 127.0.0.1:6379
hunt.cache.useSecondLevelCache = false

hunt.cache.redis.host = 127.0.0.1
hunt.cache.redis.port = 6379
hunt.cache.redis.database = 0
hunt.cache.redis.password =
hunt.cache.redis.timeout = 0
hunt.cache.redis.cluster.enabled = false
hunt.cache.redis.cluster.nodes =
```

<a name="support-backend"></a>
## Support backend

- memory
- redis
- libmemcached

<a name="versions"></a>
## Versions

 - WITH_HUNT_CACHE
 - WITH_HUNT_REDIS
 - WITH_HUNT_MEMCACHE
 - WITH_HUNT_ROCKSDB
 
<a name="tips"></a>
## Tips

Default support memory and redis drivers.

<a name="memory-adapter"></a>
## Using Memory adapter

```d
import hunt.cache;
import std.stdio;

void main() {
    auto cache = CacheFactory.create();

    // define key
    string key = "my_cache_key";

    // set cache
    cache.set(key, "My cache value.");

    // get cache
    string value = cache.get(key);

    writeln(value);
}
```

<a name="redis-adapter"></a>
## Using Redis adapter

```d
import hunt.cache;

void main() {
    CacheOption option;
    option.adapter = "redis";
    option.redis.host = "127.0.0.1";
    option.redis.port = 6379;

    auto cache = CacheFactory.create(option);

    // code for set / get ..
}
```

<a name="struct-class"></a>
## For a struct or a class

```d
import hunt.cache;
import std.stdio;

struct User {
    string name;
    int age;
}

void main() {
    auto cache = CacheFactory.create();

    // define key
    string key = "user_info";

    User user;
    user.name = "zoujiaqing";
    user.age = 99;

    // set cache
    cache.set(key, user);

    // get cache
    User userinfo = cache.get!User(key);

    writeln(userinfo.name);
}
```

<a name="full-example"></a>
## Full example

```d 
module example;

import hunt.cache;
import hunt.net.NetUtil;

struct User {
    string name;
    int age;
}

void main() {
    CacheOption option;
    option.adapter = AdapterType.MEMORY;
    option.redis.host = "127.0.0.1";
    option.redis.password = "foobared";

    auto cache = CacheFactory.create(option);

    // define key
    string key = "userinfo";

    User user;
    user.name = "putao";
    user.age = 23;

    try {
        // set value
        cache.set(key, user, 10);

        // get value
        User userinfo = cache.get!User(key);
    } catch (Exception ex) {
        warning(ex);
    }
}
```

<a name="helper"></a>
## Helper in the framework

```d
import hunt.framework;

Cache cache = Application.getInstance().cache();
cache.set("name", "jhons", 60);
string name = cache.get("name");
```
