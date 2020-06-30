# Authentication

- [Introduction](#introduction)
- [Quickstart](#quickstart)
	- [Prepare For The User's Data](#user-data)
    - [Authentication scheme and Routes protection](#scheme-route)
    - [Authenticating](#authenticating)
    - [Retrieving The Authenticated User](#retrieving-the-user)
    - [Redirecting Unauthenticated Users](#redirecting)
	- [Logging Out](#logging-out)
- [HTTP Basic Authentication](#http-basic-authentication)
- [JWT Authentication](#jwt-authentication)

<a name="introduction"></a>
## Introduction
Hunt framework makes implementing authentication very simple. In fact, almost everything is configured for you out of the box.

At its core, Hunt framework's authentication facilities are based on `Shiro` and `JWT`. See https://github.com/dicoth/dicoth for more examples. 

<a name="quickstart"></a>
## Quickstart


<a name="user-data"></a>
## Prepare For The User's Data
There are two ways to provider all the user's data. One is through config files, the other is to implement the `UserService` interface.

### Config file
By default, the Hunt framework will try to load the users from `config/users`:
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

<a name="scheme-route"></a>
### Authentication scheme and Routes protection
Two middlewares authentication schemes are implemented via `Middleware` in Hunt framework, which are `BasicAuthMiddleware` and `JwtAuthMiddleware`. Each one stands for the HTTP Basic Authentication and the JWT Authentication respectively.

<a name="checkRoute"></a>
```d
class UserController : Controller {
    mixin MakeController;

    this() {
        this.addMiddleware(new BasicAuthMiddleware(&checkRoute));
        // this.addMiddleware(new JwtAuthMiddleware(&checkRoute));
    }

    private bool checkRoute(string path, string method) {
        if (path[$ - 1] != '/')
            path ~= "/";

        string[] anonymousPaths = [url("index.login")]; // All the paths can be visited by everyone
        foreach (string p; anonymousPaths) {
            if (path.startsWith(p))
                return false;
        }

        return true;
    }
}
```


<a name="authenticating"></a>
### Authenticating
The `signIn` method in `this.request.auth()` can be used to authorize a user. It will return a `Identity`, which identifies a user. You can use it to check the status of authentication and the role-based permissions.

```d
    @Action Response login(LoginUserForm user) {
        string username = user.name;
        string password = user.password;
        string rememeber = user.rememeber;

        Identity authUser = this.request.auth().signIn(username, password, rememeber, AuthenticationScheme.Basic);
        // Identity authUser = this.request.signIn(username, password, rememeber, AuthenticationScheme.Bearer);

        string msg;
        if(authUser.isAuthenticated()) {
            msg = "User [" ~ authUser.name ~ "] logged in successfully.";

            if (authUser.hasRole("admin")) {
                msg ~= "<br>Welcome Administrator!";
            }
        } else {
            msg = "Login failed!";
        }

        return new Response(msg);
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
    @Action string logout() {
        Identity currentUser = this.request.auth().user();
        if(currentUser.isAuthenticated()) {
            this.request().auth().signOut();
            return "The user has logged out.";
        } else {
            return "No user logged in.";
        }
    }
```

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication

[HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) provides a quick way to authenticate users of your application without setting up a dedicated "login" page. To get started, add the `BasicAuthMiddleware` to your controller. The middleware is included in the Hunt framework, so you do not need to define it:

```d
this() {
    this.addMiddleware(new BasicAuthMiddleware(&checkRoute));
}
```
**Note**: The [checkRoute](#checkRoute) is used to filter these routes that you don't want to protect.

Once the middleware has been attached to the controller, you will automatically be prompted for credentials when accessing the route in your browser.


<a name="jwt-authentication"></a>
## JWT Authentication
[JWT Authentication](https://es.wikipedia.org/wiki/JSON_Web_Token) provides a more safe way to authenticate users of your application. You are recommended to use it in your projects. To get started, add the `JwtAuthMiddleware` to your controller. The middleware is included in the Hunt framework, so you do not need to define it:

```d
this() {
    this.addMiddleware(new JwtAuthMiddleware(&checkRoute));
}
```
**Note**: The [checkRoute](#checkRoute) is used to filter these routes that you don't want to protect.
