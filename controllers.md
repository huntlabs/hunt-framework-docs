# Controllers

- [Introduction](#introduction)
- [Basic Controllers](#basic-controllers)
    - [Defining Controllers](#defining-controllers)
- [Controller Middleware](#controller-middleware)
- [Request and Response](#request-and-response)
- [Resource Controllers](#resource-controllers)
- [Params Validation and From Validation](#params-validation-and-from-validation)

<a name="introduction"></a>
## Introduction

Instead of defining all of your request handling logic as Closures in route files, you may wish to organize this behavior using Controller classes. Controllers can group related request handling logic into a single class. Controllers are stored in the `source/app/controller/` directory.

<a name="basic-controllers"></a>
## Basic Controllers

<a name="defining-controllers"></a>
### Defining Controllers

Below is an example of a basic controller class. Note that the controller extends the base controller class included with hunt-framework. The base class provides a few convenience methods such as the `middleware` method, which may be used to attach middleware to controller actions:

```d
    module app.controller.HelloController;

    import hunt.framework;

    class HelloController : Controller
    {
        // must be here
        mixin MakeController;

        @Action
        string world()
        {
            return "Hello world!";
        }
    }
```    

You can define a route to this controller action like so:

    GET    /user/world  user.world

Now, when a request matches the specified route URI, the `world` method on the `HelloController` class will be executed. The route parameters will also be passed to the method.

> {tip} Controllers are not **required** to extend a base class. However, you will not have access to convenience features such as the `middleware`, `validate`, and `dispatch` methods.

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](https://github.com/huntlabs/hunt-framework-docs/blob/master/middleware.md) may be assigned to the controller files:

```d
    import app.component.middleware.AccountMiddleware;
```

However, it is more convenient to specify middleware within your controller's constructor. Using the `middleware` method from your controller's constructor, you may easily assign middleware to the controller's action. 

```d
    module app.controller.HelloController;

    import hunt.framework;
    import app.component.middleware.AccountMiddleware;

    class HelloController : Controller
    {
        // must be here
        mixin MakeController;

        this() {
            addMiddleware(new AccountMiddleware(["user.world"]);
        }

        @Action
        string world()
        {
            return "Hello world!";
        }
    }
```

<a name="request-and-response"></a>
## Request and Response

Controller have two property request and response，look: [Request](https://github.com/huntlabs/hunt/wiki/request) and [Response](https://github.com/huntlabs/hunt/wiki/response)。


#### Action return types

1. return `string`

```d
    @Action string showString()
    {
        return "Hello world.";
    }
```

2. return int、float

```d
    @Action int showInt()
    {
        return 2018;
    }
```

3. don't return

```d
    @Action void showString()
    {
        // do nothing;
    }
```

4. return `bool`

```d
    @Action bool showBool()
    {
        return true;
    }
```

5. return`Response`

```d
    @Action Response showResponse()
    {
        return new Response("Hello world.");
    }
```

6. return `JsonResponse`

```d
    @Action JsonResponse testJson2()
    {
        JSONValue company;
        company["name"] = "Putao";

        JsonResponse res = new JsonResponse(company);
        return res;
    }
```

7. return custom Response object `RedirectResponse`

```d
    @Action RedirectResponse testRedirect()
    {
        RedirectResponse r = new RedirectResponse("https://github.com/huntlabs/hunt-framework/");
        return r;
    }
```

<a name="resource-controllers"></a>
## Resource Controllers

The hunt-framework resource allocates a typical "crud" route to the controller through a routing file configuration, such as you may wish to create a controller and settings routing that handles HTTP requests "welcome" for print "welcome to hunt-framework" by your application:

1. Create a controller `IndexController.d` and create an action `testWelcome`:
```d
module app.controller.IndexController;

import hunt.framework;

class IndexController : Controller {

    mixin MakeController;

    @Action 
    string testWelcome() {
        return "welcome to hunt-framework";
    }
}
```
2. You may register a resourceful route to the controller: 

```ini
*     /welcome      index.testWelcome
```

More routing settings, please check [this](https://github.com/huntlabs/hunt-framework-docs/blob/master/routing.md).

<a name="params-validation-and-from-validation"></a>
## Params Validation and From Validation

Controller can verify parameters and forms, please check [this](https://github.com/huntlabs/hunt-framework-docs/blob/master/validation.md).
