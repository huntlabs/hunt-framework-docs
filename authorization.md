# Authentication

- [Introduction](#introduction)
- [Quick Start](#quickstart)
	- [Prepare For The User's Data](#user-data)
    - [Guard or Authentication Scheme](#scheme)
    - [Routes Protection](#route-protection)
    - [Authenticating](#authenticating)
    - [Retrieving The Authenticated User](#retrieving-the-user)
    - [Redirecting Unauthenticated Users](#redirecting)
	- [Logging Out](#logging-out)
- [Multi Authentication](#multi-authentication)

<a name="introduction"></a>
## Introduction
Hunt framework makes implementing authentication very simple. In fact, almost everything is configured for you out of the box.

At its core, Hunt framework's authentication facilities are based on `Shiro` and `JWT`. See https://github.com/dicoth/dicoth for more examples. 

<a name="quickstart"></a>
## Quick Start


<a name="user-data"></a>
## Prepare For The User's Data
There are two ways to provider all the user's data. One is through config files, the other is to implement the `UserService` interface.

### Config file
By default, the Hunt framework will try to load the users‘s basic information from `config/users`:
```ini
# name, password, roles
admin  admin  superuser
Bob    test   user|manager
```
and load their roles and permissions from `config/roles`:
```ini
# name, permissions
superuser    user.add|user.del
manager      user.add|user.logout
user         user.profile|user.logout
```
<a name="implementing-userservice"></a>
### Implementing the `UserService` interface
While implementing the `UserService` interface, you can load the user's data from anywhere:
```d
class UserProviderService : UserService {
	// Implements all the APIs defined in UserService
}
```
Then register the user-defined `UserService` to the `ServiceContainer` as a `ServiceProvider` in the main entrace of your project like `app.d` or `boostrap.d`:
```d
app.register!UserProviderService;
```
If a user-defined `UserService` is registered as `ServiceProvider`, the user's data will be loaded only from this `ServiceProvider` which means that the config files for user's data will be ignored automatically.

<a name="scheme"></a>
### Guard or Authentication Scheme
There are two schemes you can choice in Hunt Framework. They are `basic` and `jwt`. You can set this in `config/application.conf`:

```ini
# guardScheme: (none), basic, bear/jwt
auth.guardScheme = jwt
```

1. `base` scheme

When you access a protected route via a browser like `FireFox` or `Chrome`, you will be prompted to input your  `username` and `password`. Or, you can set them in the headers of the request like this:
```html
Authorization: Basic YWRtaW46YWRtaW4=
```

After login, a cookie named `__basic_token__` will be returnd if you set the `rememeber` to true when calling the `signIn`.

2. `jwt` scheme

First, you need to post your `username` and `password` to the `login`. Then, a cookie named `__jwt_token__` will be returnd if you set the `rememeber` to true when calling the `signIn`.


<a name="route-protection"></a>
### Routes protection
To protect a route or a group of routes, you can call the `withMiddleware` method in the main entrace like `app.d`.

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
```

The `withoutMiddleware` method can be used to ignore some middlewares on one specified route. There is another way to do this. That's annotation `Middleware` and `WithoutMiddleware`, see belown:

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

<a name="authenticating"></a>
### Authenticating
The `signIn` method in `request().auth()` can be used to authorize a user. It will return a `Identity`, which identifies a user. You can use it to check the status of authentication and the role-based permissions.

```d
class IndexController : Controller {
    @Middleware(AuthMiddleware.stringof)
    @Action string secret() {
        return "It's a secret page.";
    }

    @WithoutMiddleware(AuthMiddleware.stringof)
    @Action Response login(LoginUserForm user) {
        string username = user.name;
        string password = user.password;
        bool rememeber = user.rememeber;

        Identity authUser = this.request.auth().signIn(username, password, rememeber);
        string msg;
        if(authUser.isAuthenticated()) {
            msg = "User [" ~ authUser.name ~ "] logged in successfully.";
        } else {
            msg = "Login failed!";
        }
        return new Response(msg);
    }
}
```

<a name="retrieving-the-user"></a>
### Retrieving The Authenticated User
You may access the authenticated user via the `auth` method in `Request`.
```d
	Identity currentUser = this.request.auth().user();
```

#### Retrieving The User's Name and ID
To retrieve the user's name and id, you may use the `name` and `id` method in `Identity`:
```d
	string name = currentUser.name();
    ulong id = currentUser.id();
```

#### Determining If The Current User Is Authenticated
To determine if the user is already logged into your application, you may use the `isAuthenticated` method in `Identity`, which will return `true` if the user is authenticated:
```d
    if (currentUser.isAuthenticated()) {
        // The user is logged in...
    }
```

#### Determining If The User Has The Roles
To determine if the user has some roles, you may use the `hasRole` or `hasAllRoles` method in `Identity`, which will return `true` if the user has the provided role(s):
```d
    if (currentUser.hasRole("admin")) {
        // The user is an Administrator...
    }

    if (authUser.hasAllRoles("admin", "manager")) {
        // The user has all the roles
    }
```

#### Determining If The User's Permissions Is Permitted
To determine if the user is permitted, you may use the `isPermitted` method in `Identity`, which will return `true` if the user's has permissions passed:
```d
    if (currentUser.isPermitted(["user.add", "user.del"])) {
        // The user is isPermitted
    }
```

<a name="redirecting"></a>
### Redirecting Unauthenticated Users
When the `auth` middleware detects an unauthorized user, it will redirect the user to the route specified by the `unauthorizedUrl`, which can be set in `application.conf`:
```ini
# Auth
auth.loginUrl = /login
auth.successUrl = /
auth.unauthorizedUrl = /403.html
auth.basicRealm = Secure Area
```

<a name="logging-out"></a>
### Logging Out
To manually log users out of your application, you may use the `signOut` method in `Auth`. This will clear the authentication information in the user's session:

```d
class IndexController : Controller {
    @Action string logout() {
        Identity currentUser = this.request.auth().user();
        if(currentUser.isAuthenticated()) {
            this.request().auth().signOut();
            return "The user has logged out.";
        } else {
            return "No user logged in.";
        }
    }
}
```


<a name="multi-authentication"></a>
## Multi Authentication
In many cases, you need different authentications for different endpoints like `admin`, `web` and `api` etc. How to do this? Here are the steps:

1. Define constants

```d
module app.auth.Constants;

enum string WEB_GUARD_NAME = "web";
enum string WEB_JWT_TOKEN_NAME = "__web_jwt_token__";

enum string ADMIN_GUARD_NAME = "admin";
enum string ADMIN_JWT_TOKEN_NAME = "__admin_jwt_token__";
```
The `XXX_JWT_TOKEN_NAME` is the token's name returnd to the visitor which is used to store the token value after authenticate. The `XXX_GUARD_NAME` is the custom guard's name which is used to identified the guard by Hunt Framework.

2. Implementing the `UserService`

Implementing the interface `UserService` for `admin` and `web` as [above](#implementing-userservice). As an example, we call them `AdminUserService` and `WebUserService`. The will be passed to the custom guards. 

It's strongly recomended to put all the `UserService` in a named `data` folder.

3. Custom guards

You need to defined your own guard implement by inheriting from `JwtGuard` or `BasicGuard`. 

It's strongly recomended to put all the guards in a named `auth` folder.

- For admin

```d
module app.auth.AdminGuard;

import app.auth.Constants;
import app.data.AdminUserService;
import hunt.framework;

class AdminGuard : JwtGuard {
    this() {
        super(new AdminUserService(), ADMIN_GUARD_NAME);
        this.tokenCookieName = ADMIN_JWT_TOKEN_NAME;
    }
}
```

- For web

```d
module app.auth.WebGuard;

import app.auth.Constants;
import app.data.WebUserService;
import hunt.framework;

class WebGuard : JwtGuard {
    this() {
        super(new WebUserService(), WEB_GUARD_NAME);
        this.tokenCookieName = WEB_JWT_TOKEN_NAME;
    }
}
```

4. Define an `AuthServiceProvider`

This `AuthServiceProvider` is used to register all the cusome guards.  

```d
module app.providers.DemoAuthServiceProvider;

import app.auth;
import hunt.framework;
import poodinis;

class DemoAuthServiceProvider : AuthServiceProvider {
    
    override void boot() {
        AuthService authService = container().resolve!AuthService();
        authService.addGuard(new AdminGuard());
        authService.addGuard(new UserGuard());

        authService.boot();
    }
}
```

It is used to replace the default guards impelemnted in Hunt Framework. So, it should be registered 
in `app.d` like this:

```d
    app.register!DemoAuthServiceProvider; 
```

5. Extend the `AuthMiddleware`

The default auth middleware defined in Hunt Framework is not suitable for multi authentication. Especially, 
after the authentication failed, you would redirect to different `login` route for `admin` and `web`.

```d
module app.middleware.AdminAuthMiddleware;

import hunt.framework;
import std.range;

class AdminAuthMiddleware : AuthMiddleware {
    shared static this() {
        MiddlewareInterface.register!(typeof(this));
    }

    override protected bool onAccessable(Request request) {
        return true;
    }

    override protected Response onRejected(Request request) {
        return new RedirectResponse(request, url("system.user.login", null, "admin"));
    }
}
```

It's strongly recomended to put all the `AuthMiddleware` in a named `middleware` folder.

6. Binding the guards with the routes

Because all the routes use the `default` guard by default, we need bind our guards to the specified route group.

```d
  app.onBooted(() {
        RouteGroup adminGroup = app.route().group("admin");
        adminGroup.guardName = ADMIN_GUARD_NAME;
        adminGroup.withMiddleware(AdminAuthMiddleware.stringof);
        adminGroup.get("system.user.login").withoutMiddleware(AdminAuthMiddleware.stringof); 

        RouteGroup userGroup = app.route().group();
        userGroup.guardName = USER_GUARD_NAME;       
    });  
```
