# Hunt Cache
Universal cache library for D programming language.

- [Hunt Cache](#hunt-cache)
  - [Configuration](#configuration)
  - [Support backend](#support-backend)
  - [Usage](#usage)

<a name="configuration"></a>
## Configuration

Hunt framework provides an expressive, unified API for various caching backends. The cache configuration is located at `config/application.conf`. In this file you may specify which cache driver you would like to be used by default throughout your application.

The cache configuration file also contains various other options, which are documented within the file, so make sure to read over these options. By default, Hunt framework is configured to use the `memory` cache driver, which stores the serialized, cached objects in the momory. For larger applications, it is recommended that you use a more robust driver such as Redis. You may even configure multiple cache configurations for the same driver.

```conf
# redis, memory
cache.adapter = memory
cache.prefix = huntcache_
cache.expire = 3600
cache.useSecondLevelCache = false

# For Redis backend
redis.host = 127.0.0.1
redis.port = 6379
redis.enabled = true
```

<a name="support-backend"></a>
## Support backend

- memory
- redis

<a name="Usage"></a>
## Usage

```d
import hunt.framework;

struct User {
    string name;
    int age;
}

void test()
{
    auto cache = Application.instance().cache();

    // code for cache string
    cache.set("name", "jhons", 60);
    string name = cache.get("name");

    // define key
    string key = "userinfo";

    User user;
    user.name = "putao";
    user.age = 23;

    // set value
    cache.set(key, user, 10);

    // get value
    User userinfo = cache.get!User(key);
}
```
