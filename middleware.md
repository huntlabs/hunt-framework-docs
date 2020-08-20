# Middleware

- [Introduction](#introduction)
- [Defining Middleware](#defining-middleware)
- [How to use](#using-middleware)
- [List all the registered middlewares](#listing-middleware)
- [Predefined middlewares](#predefined-middleware)


<a name="introduction"></a>
## Introduction

Middleware provide a convenient mechanism for filtering HTTP requests entering your application. For example, Laravel includes a middleware that verifies the user of your application is authenticated. If the user is not authenticated, the middleware will redirect the user to the login screen. However, if the user is authenticated, the middleware will allow the request to proceed further into the application.

Additional middleware can be written to perform a variety of tasks besides authentication. A CORS middleware might be responsible for adding the proper headers to all responses leaving your application. A logging middleware might log all incoming requests to your application.

There are several middleware included in the Hunt Framework, including middleware for authentication and CSRF protection. All of these middleware are located in the `hunt/framework/middleware` directory.

<a name="defining-middleware"></a>
## Defining a new middleware

To create a new middleware, you can inherit from the interface `MiddlewareInterface` or the abstract class `AbstractMiddleware`:

In this middleware, we will only allow access to the route if the supplied `age` is greater than 200. Otherwise, we will redirect the users back to the `home` URI:

```d
import hunt.framework;

class CheckAgeMiddleware : AbstractMiddleware {
    shared static this() {
        MiddlewareInterface.register!(typeof(this));
    }

    /**
     * Handle an incoming request.
     *
     * @return Response or null
     */
    Response onProcess(Request req, Response res) {
        if (req.get!(int)("age") <= 200) {
            return new RedirectResponse(req, "home");
        }
        return null;
    }
}
```

As you can see, if the given `age` is less than or equal to `200`, the middleware will return an HTTP redirect to the client; otherwise, the request will be passed further into the application. To pass the request deeper into the application (allowing the middleware to "pass"), return `null`.

It's best to envision middleware as a series of "layers" HTTP requests must pass through before they hit your application. Each layer can examine the request and even reject it entirely.


<a name="using-middleware"></a>
## How to use

- Using code register a middleware in the constructor of a controller

The `addMiddleware` method in `Controller` can be used to register a middleware.

```d
class AdminController : Controller {
    this() {
    	this.addMiddleware(new CheckAgeMiddleware());
    }
}
```

- Using code in the `main` function

In this way, you can bind the middlware with a goup of routes or a specified route. The `withMiddleware` method is used to register a middlware, and the `withoutMiddleware` method can be used to ignore some middlewares on one specified route.

```d
import app.providers;
import app.middleware;
import hunt.framework;

void main(string[] args)
    Application app = Application.instance();
    app.onBooted(() {
        // The default group
        app.route().get("index.security").withMiddleware!(AuthMiddleware)();

        // The admin group
        app.route().group("admin").withMiddleware(AuthMiddleware.stringof);
        app.route().group("admin").get("index.test").withoutMiddleware(AuthMiddleware.stringof);
    });
}
```

- Using annotation in a controller

The annotation `Middleware` and `WithoutMiddleware` can be added on an `action` of a `Controller`:

```d
class IndexController : Controller {
    @Middleware(AuthMiddleware.stringof)
    @Action string secret() {
        return "It's a secret page.";
    }

    @WithoutMiddleware(AuthMiddleware.stringof)
    @Action Response login(LoginUser user) {
        // do something
    }
}
```

The `Middleware` is used to regist a middleware, and the `WithoutMiddleware` is used to ignore a middleware (which means the specified middleware will not apply to this `action`).

<a name="listing-middleware"></a>
## List all the registered middlewares
All the middlewares defined by your App will be registered automatically, and they can be accessed via a static method `all()` in `MiddlewareInterface`:

```d
    TypeInfo_Class[string] all = MiddlewareInterface.all();
    foreach(string key, TypeInfo_Class typeInfo; all) {
        writefln("Registed middleware: %s => %s", key, typeInfo.toString());
    }
```

<a name="predefined-middleware"></a>
## Predefined middlewares
Here are some middlewares defined by Hunt Framework.

| name | description |
|--------|--------|
| AuthMiddleware | For authorization |
