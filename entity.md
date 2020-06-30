# Entity: Getting Started

- [Entity: Getting Started](#entity-getting-started)
  - [Introduction](#introduction)
  - [Defining Models](#defining-models)
    - [Entity Model Conventions](#entity-model-conventions)
      - [Table Names](#table-names)
      - [Primary Keys](#primary-keys)
      - [Timestamps](#timestamps)
    - [Default Attribute Values](#default-attribute-values)
  - [Retrieving Models](#retrieving-models)
      - [Adding Additional Constraints](#adding-additional-constraints)
      - [Refreshing Models](#refreshing-models)
  - [Inserting & Updating Models](#inserting--updating-models)
    - [Inserts](#inserts)
    - [Updates](#updates)
      - [Mass Updates](#mass-updates)

<a name="introduction"></a>
## Introduction

The Entity ORM included with Dlang provides a beautiful, simple ActiveRecord implementation for working with your database. Each database table has a corresponding "Model" which is used to interact with that table. Models allow you to query for data in your tables, as well as insert new records into the table.

Before getting started, be sure to configure a database connection in `config/application.conf`. For more information on configuring your database,.

<a name="defining-models"></a>
## Defining Models

To get started, let's create an Entity model. Models typically live in the `app` directory, but you are free to place them anywhere that can be auto-loaded according to your  file. All Entity models extend `hunt.entity.Model` class.


<a name="Entity-model-conventions"></a>
### Entity Model Conventions

Now, let's look at an example `Role` model, which we will use to retrieve and store information from our `role` database table:

    
```
    module app.component.system.model.Role;
    import hunt.entity;
    @Table("role")
    class Role : Model 
    {
        mixin MakeModel;

        @AutoIncrement
        @PrimaryKey
        int id;
        string name;
        int created;
        int updated;
        short status;

    }

```

or use `@Column("field_name")` change output column name

```
    module app.component.rocket.model.User;
    import hunt.entity;

    @Table("rocket_user")
    class User : Model
    {
        mixin MakeModel;
        @AutoIncrement
        @PrimaryKey
        int   id;
        string uid; 
        string cids;

        @Column("user_status")
        short userStatus;

        @Column("total_bonus")
        int totalBonus;

        int created;
        int updated;


    }
```


#### Table Names

Note that we did not tell Entity which table to use for our `Role` model. By convention, the "snake case", plural name of the class will be used as the table name unless another name is explicitly specified. So, in this case, Entity will assume the `Role` model stores records in the `Roles` table. You may specify a custom table by defining a `@Table` property on your model:

```
    module app.component.system.model.Role;
    import hunt.entity;
    @Table("role")
    class Role : Model 
    {


    }
```



#### Primary Keys

Entity will also assume that each table has a primary key column named `id`. You may define a protected `@PrimaryKey 、 @AutoIncrement` property to override this convention:

```
    module app.component.system.model.Role;
    import hunt.entity;
    @Table("role")
    class Role : Model 
    {
        mixin MakeModel;

        @AutoIncrement
        @PrimaryKey
        int id;

    }
```

In addition, Entity assumes that the primary key is an incrementing integer value, which means that by default the primary key will automatically be cast to an `int`. If you wish to use a non-incrementing or a non-numeric primary key you must set the public `@AutoIncrement` property on your model :

    



If your primary key is not an integer, you should set the  `id` property on your model to `string`:

    
    module app.component.system.model.Role;
    import hunt.entity;
    @Table("role")
    class Role : Model 
    {
        mixin MakeModel;

        
        @PrimaryKey
        string id;

    }

#### Timestamps

By default, Entity expects `created_at` and `updated_at` columns to exist on your tables.  If you do not wish to have these columns automatically managed by Entity, :

    
    module app.component.system.model.Role;
    import hunt.entity;
    @Table("role")
    class Role : Model 
    {
        mixin MakeModel;

        
        
        int created_at;
        int updated_at;

    }




<a name="default-attribute-values"></a>
### Default Attribute Values

If you would like to define the default values for some of your model's attributes, you may define  on your model:

    

    module app.component.system.model.Role;
    import hunt.entity;
    @Table("role")
    class Role : Model 
    {
        mixin MakeModel;

        string role_name="test-role";

    }

<a name="retrieving-models"></a>
## Retrieving Models

Once you have created a model and Repository, you are ready to start retrieving data from your database. Think of each Entity model as a powerful allowing you to fluently query the database table associated with the model. For example:

    

    Role[] roles = (new RoleRepository()).findAll();

    foreach (role ;roles) {
        writeln(role.role_name);
    }

#### Adding Additional Constraints

The Entity `all` method will return all of the results in the model's table. Since each Entity model serves as a [query builder], you may also add constraints to queries, and then use the `get` method to retrieve the result:

    User findByEmail(string email) { 
        return _manager.createQuery!(User)("SELECT u FROM User u WHERE u.email = :email ")
            .setParameter("email", email)
            .getSingleResult();
    }

    User[] findBySatus(string status) { 
        return _manager.createQuery!(User)("SELECT u FROM User u WHERE u.status = :status ")
            .setParameter("status", status)
            .getResultList();
    }


#### Refreshing Models



The `save` method will re-hydrate the existing model using fresh data from the database. In addition, all of its loaded relationships will be refreshed as well:
    
    UserRepository userRepository = new UserRepository();
    User user =userRepository.findByEmail("example@example.com");

    user.number = 'FR 456';

    user = userRepository.save(user);

    user.number; // "FR 900"



## Inserting & Updating Models

<a name="inserts"></a>
### Inserts

To create a new record in the database, create a new model instance, set attributes on the model, then call the `save` method:

    User addUser(){
        UserRepository uerRepository = new UserRepository();
        int now = time().to!int;
        User user = new User();

        user.name = "jhon";
        user.phone = "13800000000";
        user.wechatUserId = 1;
        user.userStatus = 2;
        user.created = now;
        user.updated = now;
        user.channel = channel;

        user = uerRepository.insert(user);
        return user
    }




<a name="updates"></a>
### Updates

The `save` method may also be used to update models that already exist in the database. To update a model, you should retrieve it, set any attributes you wish to update, and then call the `save` method. Again, the `updated_at` timestamp will automatically be updated, so there is no need to manually set its value:

    UserRepository uerRepository = new UserRepository();
    User user = uerRepository.find(1);
    user.name = 'New Role Name';
    uerRepository.save(user);

#### Mass Updates

Updates can also be performed against any number of models that match a given query. In this example, all Roles that are `active` and have a `destination` of `San Diego` will be marked as delayed:

    UserRepository uerRepository = new UserRepository();
    User[] users = uerRepository.findAll();
    // ...
    uerRepository.updateAll(users);

The `updateAll` method expects an array of column and value pairs representing the columns that should be updated.


