# Cache

- [Cache](#cache)
  - [Hunt Cache](#hunt-cache)
  - [install](#install)
  - [Configuration](#configuration)
  - [Support backend](#support-backend)
  - [Versions](#versions)
  - [Tips](#tips)
  - [Sample code for Memory adapter](#sample-code-for-memory-adapter)
  - [Sample code for struct & class](#sample-code-for-struct--class)
  - [How to use Redis adapter?](#how-to-use-redis-adapter)
  - [Cache example](#cache-example)
  - [other method use framework function](#other-method-use-framework-function)
  - [cache object example](#cache-object-example)

## Hunt Cache

Universal cache library for D programming language.




## install
To use this package, run the following command in your project's root directory:

```
dub add hunt-cache
```

<a name="configuration"></a>

## Configuration

hunt-framework provides an expressive, unified API for various caching backends. The cache configuration is located at `config/application.conf`. In this file you may specify which cache driver you would like to be used by default throughout your application. hunt-framework supports popular caching backends like [Memcached](https://memcached.org) and [Redis](https://redis.io) out of the box.

The cache configuration file also contains various other options, which are documented within the file, so make sure to read over these options. By default, hunt-framework is configured to use the `file` cache driver, which stores the serialized, cached objects in the filesystem. For larger applications, it is recommended that you use a more robust driver such as Memcached or Redis. You may even configure multiple cache configurations for the same driver.

```
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

## Support backend

- memory
- redis
- libmemcached

## Versions

- WITHHUNTCACHE
- WITHHUNTREDIS
- WITHHUNTMEMCACHE
- WITHHUNTROCKSDB

## Tips

Default support memory and redis drivers.

## Sample code for Memory adapter

```
import hunt.cache;

import std.stdio;

void main()
{
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

## Sample code for struct & class

```
import hunt.cache;

import std.stdio;

struct User
{
    string name;
    int age;
}

void main()
{
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
## How to use Redis adapter?
```
import hunt.cache;

import std.stdio;

void main()
{
    CacheOption option;
    option.adapter = "redis";
    option.redis.host = "127.0.0.1";
    option.redis.port = 6379;

    auto cache = CacheFactory.create(option);

    // code for set / get ..
}
```

## Cache example

```
 
module example;

import hunt.cache;
import hunt.logging;
import hunt.net.NetUtil;

struct User {
    string name;
    int age;
}

void main() {
    CacheOption option;
    option.adapter = AdapterType.MEMORY;
    option.redis.host = "10.1.222.120";
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

        logDebug(userinfo);
    } catch (Exception ex) {
        warning(ex);
    }
}
```


## other method use framework function 


```
import hunt.framework;
Cache cache = Application.getInstance().cache(); 
cache.set("name","jhons",60);
string name = cache.get("name");
```


## cache object example


```
Product productInfo(){
    import hunt.framework;
    Cache cache = Application.getInstance().cache(); 
    string key = "product_info";
    Product product;
    if(cache.hasKey(key)){
        product = cache.get!(Product)(key);
    }else{
        product.name = "car";
        product.id = 1;
        product.price = 100;

        cache.set(key, product, cast(uint) 10);
    }
   return product;
}

 
```



