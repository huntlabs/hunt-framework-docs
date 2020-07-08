# Configuration

- [Introduction](#introduction)
- [Environment Configuration](#environment-configuration)
    - [Environment Variable](#environment-variable)
- [Custom configuration](#custom-configuration)

<a name="introduction"></a>
## Introduction

All of the configuration files for the Hunt framework are stored in the `config` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

<a name="environment-configuration"></a>
## Environment Configuration

It is often helpful to have different configuration values based on the environment where the application is running. For example, you may wish to use a different cache driver locally than you do on your production server.

Your `application.conf` file should not be committed to your application's source control, since each developer / server using your application could require a different environment configuration. Furthermore, this would be a security risk in the event an intruder gains access to your source control repository, since any sensitive credentials would get exposed.

If you are developing with a team, you may wish to continue including a `application.env.conf` file with your application. By putting placeholder values in the example configuration file, other developers on your team can clearly see which environment variables are needed to run your application. 

> {tip} For more items, check the file [ApplicationConfig.d](https://github.com/huntlabs/hunt-framework/blob/master/source/hunt/framework/config/ApplicationConfig.d).

<a name="environment-variable"></a>
### Environment Variable

All variables will in your `application.conf` files are parsed as strings, and you can set the configuration used by setting some environment variables, get environment variables using `environment.get ("key", "default_ value");`, for example, the get name is hunt_ Env environment variables :

```d
    import std.process;
    string huntEnv = environment.get("HUNT_ENV", "test");
```
When environment variable `HUNT_ENV` is empty, the default value `test` will be used, and hunt-framework will use `application.test.conf`;

The hunt-framework presupposes four important environment variables :

`ENV_APP_NAME` is used to set the app name;
`ENV_APP_VERSION` is used to set the app version;
`ENV_APP_ENV` is the same as `HUNT_ENV`, and `HUNT_ENV` has higher priority than `ENV_APP_ENV`;
`ENV_APP_LANG` is used to set the default language pack.

You can also specify the configuration file to use from the command line, for example, to start a project using the command line and specify the configuration file `application.test.conf`:

```sh
./examples serve -e test
```

> {tip} Setting the configuration file in command line has higher priority than set environment variable.

Methods to get the settings in the configuration file, such as getting `application.name` :

```d
module app.controller.IndexController;

import hunt.framework;

class IndexController : Controller {

    mixin MakeController;

    @Action 
    string testApplicationName() {
        ApplicationConfig conf = config();
        return conf.application.name;
    }
```

<a name="acustom-configuration"></a>
## Custom configuration

Take configuring a `GithubConfig` as an example:

1. Create a `source/app/config/basicalapplicationconfig.d` file:

```d
module app.config.BasicApplicationConfig;

import hunt.framework.config.ApplicationConfig;
import hunt.util.Configuration;

class BasicApplicationConfigBase : ApplicationConfig {

}

@ConfigurationFile("application")
class BasicApplicationConfig : BasicApplicationConfigBase {

    struct GithubConfig {
        string appid = "1234";
        string secret = "test";
        string accessTokenUrl = "TokenUrl";
        string userInfoUrl = "InfoUrl";
    }

    GithubConfig github;
}
```

2. Create a `source/app/config/GithubConfig.d` file:

```d
module app.config.GithubConfig;

import hunt.util.Configuration;

@ConfigurationFile("github")
class GithubConfig {
    string appid = "12345";
    string secret = "test-github";
    string accessTokenUrl = "TokenUrl";
    string userInfoUrl = "InfoUrl";
}
```

3. Create a `source/app/config/package.d` file:

```d
module app.config;

public import app.config.BasicApplicationConfig;
public import app.config.GithubConfig;
```

4. Create a file `source/app/controller/IndexController.d` and read the settings in the configuration file :

```d
module app.controller.IndexController;

import app.config;

import hunt.framework;

class IndexController : Controller {

    mixin MakeController;

    @Action 
    string testTokenUrl() {
        GithubConfig githubConfig = configManager().load!GithubConfig();
        string accessTokenUrl = githubConfig.accessTokenUrl;
        return accessTokenUrl;
    }
```