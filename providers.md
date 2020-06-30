# Service Providers

- [Introduction](#introduction)
- [Writing Service Providers](#writing-service-providers)
    - [The Register Method](#the-register-method)
    - [The Boot Method](#the-boot-method)
- [Registering Providers](#registering-providers)
- [Deferred Providers](#deferred-providers)

<a name="introduction"></a>
## Introduction

Service providers are the central place of all Laravel application bootstrapping. Your own application, as well as all of Laravel's core services are bootstrapped via service providers.

But, what do we mean by "bootstrapped"? In general, we mean **registering** things, including registering service container bindings, event listeners, middleware, and even routes. Service providers are the central place to configure your application.

If you open the [hunt/framework/provider](https://github.com/huntlabs/hunt-framework/tree/master/source/hunt/framework/provider) directory included with Hunt-Framework, you will see all the avaliable `providers`. These are all of the service provider classes that will be loaded for your application. Note that many of these are "deferred" providers, meaning they will not be loaded on every request, but only when the services they provide are actually needed.

In this overview you will learn how to write your own service providers and register them with your Hunt application.

<a name="writing-service-providers"></a>
## Writing Service Providers

All service providers extend the `hunt.framework.provider.ServiceProvider` class. Most service providers contain a `register` and a `boot` method. Within the `register` method, you should **only bind things into the [service container](container.md)**. You should never attempt to register any event listeners, routes, or any other piece of functionality within the `register` method.

<a name="the-register-method"></a>
### The Register Method

As mentioned previously, within the `register` method, you should only bind things into the [service container](container.md). You should never attempt to register any event listeners, routes, or any other piece of functionality within the `register` method. Otherwise, you may accidentally use a service that is provided by a service provider which has not loaded yet.

Let's take a look at a basic service provider. Within any of your service provider methods, you always have access to the function `serviceContainer()` which provides access to the service container:

```d
import hunt.framework.provider.ServiceProvider;

class TranslationServiceProvider : ServiceProvider {
    /**
     * Register any application services.
     *
     * @return void
     */
    override void register() {
        container.register!(I18n)(() {
            ApplicationConfig config = container.resolve!ApplicationConfig();

            I18n i18n = new I18n();
            i18n.loadLangResources(buildPath(DEFAULT_RESOURCE_PATH, langLocation));
            return i18n;
        }).singleInstance();
    }
}
```

This service provider only defines a `register` method, and uses that method to define an implementation of `I18n` in the service container. If you don't understand how the service container works, check out [its documentation](container.md).

<a name="the-boot-method"></a>
### The Boot Method

So, what if we need to register a BreadcrumbProvider within our service provider? This should be done within the `boot` method. **This method is called after all other service providers have been registered**, meaning you have access to all other services that have been registered by the framework:

```d
import hunt.framework.provider.ServiceProvider;
class BreadcrumbProvider : BreadcrumbServiceProvider {
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    override void boot() {
        breadcrumbs.register("home", (Breadcrumbs trail, Object[] params...) {
            trail.push("Home", "/home");
        });

        breadcrumbs.register("index.about", (Breadcrumbs trail, Object[] params...) {
            trail.parent("home");
            trail.push("About", url("index.about"));
        });
    }
}
```

<a name="registering-providers"></a>
## Registering Providers

By default, all service providers in the [hunt/framework/provider](https://github.com/huntlabs/hunt-framework/tree/master/source/hunt/framework/provider) directory are registered automactically, which are a set of Hunt-Framework core service providers. These providers bootstrap the core Hunt-Framework components, such as the HttpServer, queue, cache, and others.

To register your provider, add it to the main entrace of your project:

```d
// Register customized Service Providers
app.register!BreadcrumbProvider;
```

<a name="deferred-providers"></a>
## Deferred Providers

If your provider is **only** registering bindings in the [service container](container.md), you may choose to defer its registration until one of the registered bindings is actually needed. Deferring the loading of such a provider will improve the performance of your application, since it is not loaded from the filesystem on every request.

Hunt-Framework compiles and stores a list of all of the services supplied by deferred service providers, along with the name of its service provider class. Then, only when you attempt to resolve one of these services does Hunt-Framework load the service provider.

To defer the loading of a provider, create a initializer and pass it to the `register` method (see the [example](https://github.com/huntlabs/hunt-examples/blob/master/website-basic/source/app/BasicConfigProvider.d)):

```d
class BasicConfigProvider : ConfigServiceProvider {
    /**
     * Register any application services.
     *
     * @return void
     */
    override void register() {
        container.register!(ApplicationConfig, BasicApplicationConfig)(() {
            ConfigManager configManager = container.resolve!(ConfigManager)();
            return configManager.load!(BasicApplicationConfig);
        }).singleInstance();
    }
}
```
