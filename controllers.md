# Controllers

- [Introduction](#introduction)
- [Basic Controllers](#basic-controllers)
    - [Defining Controllers](#defining-controllers)
- [Controller Middleware](#controller-middleware)
- [Dependency Injection & Controllers](#dependency-injection-and-controllers)
- [Request and Response](#request-and-response)

<a name="introduction"></a>
## Introduction

Instead of defining all of your request handling logic as Closures in route files, you may wish to organize this behavior using Controller classes. Controllers can group related request handling logic into a single class. Controllers are stored in the `source/app/controller/` directory.

<a name="basic-controllers"></a>
## Basic Controllers

<a name="defining-controllers"></a>
### Defining Controllers

Below is an example of a basic controller class. Note that the controller extends the base controller class included with hunt-framework. The base class provides a few convenience methods such as the `middleware` method, which may be used to attach middleware to controller actions:

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

You can define a route to this controller action like so:

    GET    /user/world  user.world

Now, when a request matches the specified route URI, the `world` method on the `HelloController` class will be executed. The route parameters will also be passed to the method.

> {tip} Controllers are not **required** to extend a base class. However, you will not have access to convenience features such as the `middleware`, `validate`, and `dispatch` methods.

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](https://github.com/huntlabs/hunt-framework-docs/blob/master/middleware.md) may be assigned to the controller files:

    import app.component.middleware.AccountMiddleware;

However, it is more convenient to specify middleware within your controller's constructor. Using the `middleware` method from your controller's constructor, you may easily assign middleware to the controller's action. 

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

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection & Controllers

#### Constructor Injection

 You are able to type-hint any dependencies your controller may need in its constructor. The declared dependencies will automatically be resolved and injected into the controller instance:

    module app.controller.HelloController;

    import hunt.framework;

    class HelloController : Controller
    {
        // must be here
        mixin MakeController;

        protected string name;

        this(sting name) {
            this.name = name;
        }
    }

#### Method Injection

In addition to constructor injection, you may also type-hint dependencies on your controller's methods. A common use-case for method injection is injecting the `hunt.framework.http.Request` instance into your controller methods:

    module app.controller.HelloController;

    import hunt.framework;
    import hunt.framework.http.Request;

    class HelloController : Controller
    {
        // must be here
        mixin MakeController;

        @Action
        string hello(Request request)
        {
            return "Hello " ~ request.input("name");
        }
    }

<a name="request-and-response"></a>
## Request and Response

Controller have two property request and response，look: [Request](https://github.com/huntlabs/hunt/wiki/request) and [Response](https://github.com/huntlabs/hunt/wiki/response)。


#### Action return types

1. return `string`

    @Action string showString()
    {
        return "Hello world.";
    }

2. return int、float

    @Action int showInt()
    {
        return 2018;
    }

3. don't return

    @Action void showString()
    {
        // do nothing;
    }

4. return `bool`

    @Action bool showBool()
    {
        return true;
    }

5. return`Response`

    @Action Response showResponse()
    {
        return new Response("Hello world.");
    }

6. rerturn `JsonResponse`

    @Action JsonResponse testJson2()
    {
        JSONValue company;
        company["name"] = "Putao";

        JsonResponse res = new JsonResponse(company);
        return res;
    }

7. return custom Response object `RedirectResponse`

    @Action RedirectResponse testRedirect()
    {
        RedirectResponse r = new RedirectResponse("https://github.com/huntlabs/hunt-framework/");
        return r;
    }