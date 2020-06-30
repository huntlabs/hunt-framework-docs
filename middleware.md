# Middleware

- [Introduction](#introduction)
- [Defining Middleware](#defining-middleware)
- [Registering Middleware](#registering-middleware)


<a name="introduction"></a>
## Introduction

Middleware provide a convenient mechanism for filtering HTTP requests entering your application. For example, Laravel includes a middleware that verifies the user of your application is authenticated. If the user is not authenticated, the middleware will redirect the user to the login screen. However, if the user is authenticated, the middleware will allow the request to proceed further into the application.

Additional middleware can be written to perform a variety of tasks besides authentication. A CORS middleware might be responsible for adding the proper headers to all responses leaving your application. A logging middleware might log all incoming requests to your application.

There are several middleware included in the Hunt Framework, including middleware for authentication and CSRF protection. All of these middleware are located in the `hunt.framework.middleware` directory.

<a name="defining-middleware"></a>
## Defining Middleware

To create a new middleware, you must make it inherit from the interface `MiddlewareInterface`:

In this middleware, we will only allow access to the route if the supplied `age` is greater than 200. Otherwise, we will redirect the users back to the `home` URI:

```d
import hunt.framework;

class CheckAgeMiddleware : MiddlewareInterface {
    string name() {
        return CheckAgeMiddleware.stringof;
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


<a name="registering-middleware"></a>
## Registering Middleware

The `addMiddleware` method in Controller can be used to register a middleware.
```d
this() {
    this.addMiddleware(new CheckAgeMiddleware());
}
```
