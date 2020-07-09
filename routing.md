# Routing

- [Basic Routing](#basic-routing)
    - [The Default Route Files](#default-route-files)
    - [Available Router Methods](#availalbe-route-methods)
- [Route Parameters](#route-parameters)
    - [Required Parameters](#required-parameters)
- [Route Groups](#route-groups)
- [Cross-Origin Resource Sharing (CORS)](#cors)

<a name="basic-routing"></a>
## Basic Routing

<a name="default-route-files"></a>
#### The Default Route Files

All the routes in Hunt framework are defined in your route files, which are located in the `config/` directory. These files are automatically loaded by the framework. The `config/routes` file defines routes that are for your web interface.

For most applications, you will begin by defining routes in your `config/routes` file. The routes defined in `config/routes` may be accessed by entering the defined route's URL in your browser. For example, you may access the following route by navigating to `http://your-app.test/user` in your browser:

```ini
    # method    path            controller.action
    GET         /user           user.user
```

Routes defined in the `config/api.routes` file are nested within a route group by the `api`. Within this group, the `/api` URI prefix is automatically applied so you do not need to manually apply it to every route in the file. 

<a name="availalbe-route-methods"></a>
#### Available Router Methods

The router allows you to register routes that respond to any HTTP request:

```
GET             path            module.controller.action
POST            path            module.controller.action
PUT             path            module.controller.action
DELETE          path            module.controller.action
HEAD            path            module.controller.action
OPTIONS         path            module.controller.action
PATCH           path            module.controller.action
```

Sometimes you may need to register a route that responds to multiple HTTP verbs. You may even register a route that responds to all HTTP request using the `*` method:

```
*               path            module.controller.action
```

<a name="route-parameters"></a>
## Route Parameters

<a name="required-parameters"></a>
### Required Parameters

Sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:

```
GET         /block/{id<[0-9]+>}          block.block.detail
```

<a name="route-groups"></a>
## Route Groups
Route groups allow you to share route attributes, such as middleware or namespaces, across a large number of routes without needing to define those attributes on each individual route. 

All the route groups are defined in `config/application.conf` and are separated by a comma. Each group is consisted of three parts (separated by a colon): the goupe route name, the group type and the group name. There are two kinds of Route. One is `path`, which means all the routes are grouped by the path's name. The other is `domain`, which means all the routes are grouped by the domain name. For example:

```ini
route.groups = admin:path:admincp, api:domain:api.example.com
```

Then, define all the route items in different group config files based on the goupe route name with an extersion `.routes`. For example:

```
config/admin.routes
config/api.routes
```

<a name="cors"></a>
## Cross-Origin Resource Sharing (CORS)

The Hunt framework can automatically respond to CORS OPTIONS requests with values that you configure. All CORS settings may be configured in your `config/application.conf` file:
```ini
http.allowOrigin=*
http.allowMethods=*
http.allowHeaders=DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type
http.enableCors=true
```

> {tip} For more information on CORS and CORS headers, please consult the [MDN web documentation on CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).
