# Routing

- [Basic Routing](#basic-routing)
- [Route Parameters](#route-parameters)
    - [Required Parameters](#required-parameters)
- [Route Groups](#route-groups)

<a name="basic-routing"></a>
## Basic Routing

#### The Default Route Files

All hunt-framework routes are defined in your route files, which are located in the `config` directory. These files are automatically loaded by the framework. The `config/routes` file defines routes that are for your web interface.

For most applications, you will begin by defining routes in your `config/routes` file. The routes defined in `config/routes` may be accessed by entering the defined route's URL in your browser. For example, you may access the following route by navigating to `http://your-app.test/user` in your browser:

    # method    path            controller.action
    GET         /user           user.user

Routes defined in the `config/api.routes` file are nested within a route group by the `api`. Within this group, the `/api` URI prefix is automatically applied so you do not need to manually apply it to every route in the file. 

#### Available Router Methods

The router allows you to register routes that respond to any HTTP request:

    GET             path            module.controller.action
    POST            path            module.controller.action
    PUT             path            module.controller.action
    DELETE          path            module.controller.action
    HEAD            path            module.controller.action
    OPTIONS         path            module.controller.action
    PATCH           path            module.controller.action

Sometimes you may need to register a route that responds to multiple HTTP verbs. You may even register a route that responds to all HTTP request using the `*` method:

    *               path            module.controller.action

<a name="route-parameters"></a>
## Route Parameters

<a name="required-parameters"></a>
### Required Parameters

Sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:
    
    GET         /block/{id<[0-9]+>}          block.block.detail

<a name="route-groups"></a>
## Route Groups

If you want use advanced method, allow bind domain & directory go to config/application.conf find this option:

    route.groups = admin:path:admincp, api:domain:api.example.com

You have create groups config files:

    config/admin.routes
    config/api.routes
