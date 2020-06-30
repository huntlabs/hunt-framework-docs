# Redis

- [Redis](#redis)
  - [Introduction](#introduction)
  - [Hunt Redis](#hunt-redis)
    - [So what can I do with Redis?](#so-what-can-i-do-with-redis)
    - [To use it just:](#to-use-it-just)
    - [Redis Cluster](#redis-cluster)
    - [hunt-framework Configuration](#hunt-framework-configuration)
  - [hunt-redis functions](#hunt-redis-functions)

<a name="introduction"></a>
## Introduction

[Redis](https://redis.io) is an open source, advanced key-value store. It is often referred to as a data structure server since keys can contain [strings](https://redis.io/topics/data-types#strings), [hashes](https://redis.io/topics/data-types#hashes), [lists](https://redis.io/topics/data-types#lists), [sets](https://redis.io/topics/data-types#sets), and [sorted sets](https://redis.io/topics/data-types#sorted-sets).

Before using Redis with hunt-framework, we encourage you to install and use the [Redis] . The extension is more complex to install but may yield better performance for applications that make heavy use of Redis.

Alternatively, you can install the `hunt-redis` package via dub:

    dub add hunt-redis

<a name="Hunt Redis"></a>
## Hunt Redis

### So what can I do with Redis?

All of the following redis features are supported:

- Sorting
- Connection handling
- Commands operating on any kind of values
- Commands operating on string values
- Commands operating on hashes
- Commands operating on lists
- Commands operating on sets
- Commands operating on sorted sets
- Transactions
- Pipelining
- Publish/Subscribe
- Persistence control commands
- Remote server control commands
- Connection pooling
- Sharding (MD5, MurmurHash)
- Key-tags for sharding
- Sharding with pipelining
- Scripting with pipelining
- Redis Cluster

### To use it just:

```
# string password="123456";
Redis redis = new Redis("localhost","6379");
# redis.auth(password);  // if has password ,use auth() method
redis.set("foo", "bar");
string value = redis.get("foo");
```

###  Redis Cluster
Redis cluster specification (still under development) is implemented

```
Set!(HostAndPort) redisClusterNodes = new HashSet!(HostAndPort)();
//Redis Cluster will attempt to discover cluster nodes automatically
redisClusterNodes.add(new HostAndPort("127.0.0.1", 7379));
RedisCluster rc = new RedisCluster(redisClusterNodes);
rc.set("foo", "bar");
string value = rc.get("foo");
```

<a name="hunt-framework Configuration"></a>
### hunt-framework Configuration

The Redis configuration for your application is located in the `config/application.conf` configuration file. Within this file, you will see a `redis` array containing the Redis servers utilized by your application:

```
    hunt.redis.enabled = true
    hunt.redis.host = 127.0.0.1
    hunt.redis.port = 6379
    hunt.redis.database = 0
    hunt.redis.password = 
    hunt.redis.timeout = 0
```

The default server configuration should suffice for development. However, you are free to modify this array based on your environment. Each Redis server defined in your configuration file is required to have a name, host, and port.


    

simlpe use with `hunt.framework.storage.redis`

```
void main(){
    import hunt.redis.Redis;
    import hunt.framework.storage.redis;

    Redis redis = getRedis();
    scope(exit) redis.close();
    redis.set("name","john");
    string name = redis.get("name");
}


```


## hunt-redis functions
all redis functions you can see :

[hunt-redis functions](https://github.com/huntlabs/hunt-redis/blob/master/source/hunt/redis/Redis.d
)



