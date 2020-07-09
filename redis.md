

# redis


  - [Introduction](#introduction)
    - [So what can I do with Redis?](#so-what-can-i-do-with-redis)
  - [Hunt framework Redis](#hunt-framework-redis)
    - [Configuration](#configuration)
    - [Use example](#use-example)
  - [Custom using Hunt Redis](#custom-using-hunt-redis)
    - [To use it just:](#to-use-it-just)
    - [Redis Cluster](#redis-cluster)
  - [hunt-redis functions](#hunt-redis-functions)

<a name="introduction"></a>
## Introduction

[Redis](https://redis.io) is an open source, advanced key-value store. It is often referred to as a data structure server since keys can contain [strings](https://redis.io/topics/data-types#strings), [hashes](https://redis.io/topics/data-types#hashes), [lists](https://redis.io/topics/data-types#lists), [sets](https://redis.io/topics/data-types#sets), and [sorted sets](https://redis.io/topics/data-types#sorted-sets).

<a name="so-what-can-i-do-with-redis"></a>
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

<a name="hunt-framework-redis"></a>
## Hunt framework Redis

<a name="configuration"></a>
###  Configuration

The Redis configuration for your application is located in the `config/application.conf` configuration file. Within this file, you will see a `redis` array containing the Redis servers utilized by your application:

```ini
# Redis
redis.enabled = true
redis.prefix = app:
redis.host = 127.0.0.1
redis.port = 6379
redis.database = 0
redis.password = 
redis.timeout = 0

# Redis pool
redis.pool.enabled = false
redis.pool.maxWait = 5000
redis.pool.maxIdle = 50
redis.pool.minIdle = 5

# Redis cluster
redis.cluster.enabled = false
redis.cluster.nodes = 127.0.0.1:6379, 127.0.0.1:6380, 127.0.0.1:6381
```

The default server configuration should suffice for development. However, you are free to modify this array based on your environment. Each Redis server defined in your configuration file is required to have a name, host, and port.


<a name="use-example"></a>
### use example
hunt framework redis you can you use  like example below:

```d
import hunt.framework;
void test()
{
    auto redis = app().redis();
    
    redis.set("name","john");
    string name = redis.get("name");
}
```

<a name="custom-using-hunt-redis"></a>
## Custom using Hunt Redis

<a name="to-use-it-just"></a>
### To use it just:

```D
import hunt.redis;

void main()
{
    string password="123456";

    Redis redis = new Redis("localhost", "6379");

    redis.auth(password);  // if has password ,use auth() method

    redis.set("foo", "bar");
    string value = redis.get("foo");

    import std.stdio : writeln;

    writeln(value);
}
```

<a name="redis-cluster"></a>
###  Redis Cluster
Redis cluster specification (still under development) is implemented

```d
Set!(HostAndPort) redisClusterNodes = new HashSet!(HostAndPort)();
//Redis Cluster will attempt to discover cluster nodes automatically
redisClusterNodes.add(new HostAndPort("127.0.0.1", 7379));
RedisCluster rc = new RedisCluster(redisClusterNodes);
rc.set("foo", "bar");
string value = rc.get("foo");
```

<a name="hunt-redis-functions"></a>
## Hunt-redis functions
All redis functions you can see :

[Hunt-redis functions](https://github.com/huntlabs/hunt-redis/blob/master/source/hunt/redis/Redis.d)
