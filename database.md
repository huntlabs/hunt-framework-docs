# Database: Getting Started

- [Introduction](#introduction)
- [Configuration](#configuration)
- [Using EntityManager](#using-entitymanager)
- [Raw SQL Queries](#running-raw-sql-queries)
    - [Select](#running-a-select-query)
    - [Insert](#running-an-insert-statement)
    - [Update](#running-an-update-statement)
    - [Delete](#running-a-delete-statement)
    - [Named Bindings](#using-named-bindings)
    - [General Query](#running-a-general-statement)
- [Database Transactions](#database-transactions)
    - [Manually Using Transactions](#manually-using-transactions)

<a name="introduction"></a>
## Introduction

The Hunt framework makes interacting with databases extremely simple across a variety of database backends using either raw SQL. Currently, the Hunt framework supports two databases:

- MySQL 5.6+ ([Version Policy](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 9.4+ ([Version Policy](https://www.postgresql.org/support/versioning/))

<a name="configuration"></a>
### Configuration

The database configuration for your application is located at `config/application.conf`. In this file you may define all of your database connections, as well as specify which connection should be used by default. Examples for most of the supported database systems are provided in this file.

By default, hunt-framework's sample is ready to use  which is a convenient virtual machine for doing hunt-framework development on your local machine. You are free to modify this configuration as needed for your local database.

`config/application.conf`

```ini
# Database
# postgresql, mysql
database.driver=postgresql
database.host=127.0.0.1
database.port=2345
database.database=test
database.username=root
database.password=
database.charset=utf8
database.prefix=
database.enabled=false

database.pool.name=
database.pool.minIdle=5
database.pool.idleTimeout=30000
database.pool.maxConnection=20
database.pool.minConnection=5
database.pool.maxPoolSize=20
database.pool.minPoolSize=20
database.pool.maxLifetime=2000000
database.pool.connectionTimeout=30000

```

<a name="using-multiple-database-connections"></a>
### Using EntityManager

```d
    EntityManager defaultEntityManager()
    {
        
        return  defaultEntityManagerFactory().currentEntityManager();
        
    }
    _manager = defaultEntityManager();
```
or 
```d
    public EntityManager createEntityManager()
    {
        return defaultEntityManagerFactory().currentEntityManager();
    }

    _manager = createEntityManager();
    
```

use `_manager` to query database result

```d
    User findByUid(int uid){
        return _manager.createQuery!(User)(" SELECT a FROM User a WHERE a.uid= :uid")
        .setParameter("uid", uid)
        .getSingleResult();
    }

```

<a name="running-queries"></a>
## Raw SQL Queries

Once you have configured your database connection, you may run queries using the `DB` facade. The `DB` facade provides methods for each type of query: `select`, `update`, `insert`, `delete`, and `statement`.

#### Select
To run a basic query, you may use the `select` statement:

```d
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

```d
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
```d
    UserRepository UserRepository = new UserRepository();
    User userModel = new User();
    userModel.name = "John";
    userModel.age = 20;
    UserRepository.insert(userModel);
```

#### Running An Update Statement

The `save` method should be used to update existing records in the database. The database model affected by the statement will be returned:
```d
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

```d
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

```d
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
