# Database: Getting Started

- [Database: Getting Started](#database-getting-started)
  - [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Using EntityManager](#using-entitymanager)
  - [Running Raw SQL Queries](#running-raw-sql-queries)
      - [Running A Select Query](#running-a-select-query)
      - [Using Named Bindings](#using-named-bindings)
      - [Running An Insert Statement](#running-an-insert-statement)
      - [Running An Update Statement](#running-an-update-statement)
      - [Running A Delete Statement](#running-a-delete-statement)
      - [Running A General Statement](#running-a-general-statement)
  - [Database Transactions](#database-transactions)
      - [Manually Using Transactions](#manually-using-transactions)

<a name="introduction"></a>
## Introduction

hunt-framework makes interacting with databases extremely simple across a variety of database backends using either raw SQL, Currently, hunt-framework supports four databases:

- MySQL 5.6+ ([Version Policy](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 9.4+ ([Version Policy](https://www.postgresql.org/support/versioning/))
- SQLite 3.8.8+
- SQL Server 2017+ ([Version Policy](https://support.microsoft.com/en-us/lifecycle/search))


<a name="configuration"></a>
### Configuration

The database configuration for your application is located at `config/application.conf`. In this file you may define all of your database connections, as well as specify which connection should be used by default. Examples for most of the supported database systems are provided in this file.

By default, hunt-framework's sample is ready to use  which is a convenient virtual machine for doing hunt-framework development on your local machine. You are free to modify this configuration as needed for your local database.

`config/application.conf or config/application.production.conf`

```
hunt.database.default.driver=mysql
hunt.database.default.host=127.0.0.1
hunt.database.default.port=3306
hunt.database.default.database=hunt-framework
hunt.database.default.username=hunt-user
hunt.database.default.password=123456789
hunt.database.default.charset=utf8mb4
hunt.database.default.prefix=hc_
hunt.database.default.enabled=true

hunt.database.pool.name=
hunt.database.pool.minIdle=5
hunt.database.pool.idleTimeout=30000
hunt.database.pool.maxConnection=20
hunt.database.pool.minConnection=5
hunt.database.pool.maxPoolSize=20
hunt.database.pool.minPoolSize=20
#hunt.database.pool.maxLifetime=2000000
hunt.database.pool.connectionTimeout=30000

```




<a name="using-multiple-database-connections"></a>
### Using EntityManager

```
    EntityManager defaultEntityManager()
    {
        
        return  defaultEntityManagerFactory().currentEntityManager();
        
    }
    _manager = defaultEntityManager();
```
or 
```
    public EntityManager createEntityManager()
    {
        return defaultEntityManagerFactory().currentEntityManager();
    }

    _manager = createEntityManager();
    
```

use `_manager` to query database result

```
    User findByUid(int uid){
        return _manager.createQuery!(User)(" SELECT a FROM User a WHERE a.uid= :uid")
        .setParameter("uid", uid)
        .getSingleResult();
    }

```

<a name="running-queries"></a>
## Running Raw SQL Queries

Once you have configured your database connection, you may run queries using the `DB` facade. The `DB` facade provides methods for each type of query: `select`, `update`, `insert`, `delete`, and `statement`.

#### Running A Select Query

To run a basic query, you may use the `select` method on the `DB` facade:
```

    module app.component.user.repository.UserRepository;
    import hunt.entity;
    import hunt.framework.Simplify;

    import app.component.user.model.User;
    class UserRepository : EntityRepository!(User, int)
    {
        this() {
            super(defaultEntityManager());
        }

        // Query one result
        User findByUid(int uid){
            return _manager.createQuery!(User)(" SELECT a FROM User a WHERE a.uid= :uid")
            .setParameter("uid", uid)
            .getSingleResult();
        }

        // Query multiple result
        User[] finUsers(){
            return _manager.createQuery!(User)(" SELECT a FROM User a")
            .getResultList();
        }
    }
```


The first argument passed to the `select` method is the raw SQL query, while the second argument is any parameter bindings that need to be bound to the query. Typically, these are the values of the `where` clause constraints. Parameter binding provides protection against SQL injection.

The `select` method will always return an `array` of results. Each result within the array will be a Dlang  object, allowing you to access the values of the results:

```
    User[] users = (new UserRepository()).finUsers();
    foreach (user;users) {
        writeln(user.name);
    }
```
#### Using Named Bindings

Instead of using `setParameter()` to represent your parameter bindings, you may execute a query using named bindings:

    User users = _manager.createQuery!(User)(" SELECT a FROM User a WHERE a.uid= :uid")
            .setParameter("uid", uid)
            .getSingleResult();

#### Running An Insert Statement

To execute an `insert` statement, you may use the `insert` method on the `DB` facade.:
```
    UserRepository UserRepository = new UserRepository();
    User userModel = new User();
    userModel.name = "John";
    userModel.age = 20;
    UserRepository.insert(userModel);
```

#### Running An Update Statement

The `save` method should be used to update existing records in the database. The database model affected by the statement will be returned:
```
    UserRepository UserRepository = new UserRepository();
    User user = UserRepository.findById(1);
    user.name = "Johns";
    user = UserRepository.save(user);
```

#### Running A Delete Statement

The `delete` method should be used to delete records from the database. Like `update`, the number of rows affected will be returned:

    
    UserRepository UserRepository = new UserRepository();
    UserRepository.removeById(1);
    
    or 
    
    UserRepository.remove(user);// user is User model 

#### Running A General Statement

Some database statements do not return any value. For these types of operations, you may use the `getValue` method on the `RowSet` result:

```
    RowSet findUser(int id){
        string idstr = id.to!string;
        RowSet results = _manager.getSession().prepare(
            "SELECT * FROM user WHERE "~idstr
            ).query();
        return results;
    }

    // or uses below Statement : 
    // RowSet res = _manager.getSession().query(sql);
    
    // get rowset result value
    RowSet results = (new UserRepository()).findUser(1);
    foreach(item; results){
        // get string value
        string name = item.getValue("nickname").toString();
        // get int value
        int age = isNumeric(item.getValue("age").toString())?item.getValue("age").toString().to!int:0;
    }
```


<a name="database-transactions"></a>
## Database Transactions

You may use the `transaction` method on the `DB` facade to run a set of operations within a database transaction. If an exception is thrown within the transaction `Closure`, the transaction will automatically be rolled back. If the `Closure` executes successfully, the transaction will automatically be committed. You don't need to worry about manually rolling back or committing while using the `transaction` method:

```
    defaultEntityManager().getTransaction().begin();
    try
    {
        // ....
        // some database Statements
        // ....

        defaultEntityManager().getTransaction().commit();
    }
    catch(Exception e)
    {
        // some code deal Exception 
        defaultEntityManager().getTransaction().rollback();
    }
    
````


#### Manually Using Transactions

If you would like to begin a transaction manually and have complete control over rollbacks and commits, you may use the `beginTransaction` method on the `DB` facade:

    defaultEntityManager().getTransaction().begin();

You can rollback the transaction via the `rollBack` method:

    defaultEntityManager().getTransaction().rollback();

Lastly, you can commit a transaction via the `commit` method:

    defaultEntityManager().getTransaction().commit();

