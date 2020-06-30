# Configuration

- [Introduction](#introduction)
- [Environment Configuration](#environment-configuration)
    - [Environment Variable Types](#environment-variable-types)
    - [Retrieving Environment Configuration](#retrieving-environment-configuration)
- [Accessing Configuration Values](#accessing-configuration-values)

<a name="introduction"></a>
## Introduction

All of the configuration files for the hunt-framework are stored in the `config` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

<a name="environment-configuration"></a>
## Environment Configuration

It is often helpful to have different configuration values based on the environment where the application is running. For example, you may wish to use a different cache driver locally than you do on your production server.

Your `application.conf` file should not be committed to your application's source control, since each developer / server using your application could require a different environment configuration. Furthermore, this would be a security risk in the event an intruder gains access to your source control repository, since any sensitive credentials would get exposed.

If you are developing with a team, you may wish to continue including a `application.env.conf` file with your application. By putting placeholder values in the example configuration file, other developers on your team can clearly see which environment variables are needed to run your application. 

> {tip} More item value reference file [ApplicationConfig.d](https://github.com/huntlabs/hunt-framework/blob/master/source/hunt/framework/config/ApplicationConfig.d#L41)

<a name="environment-variable-types"></a>
### Environment Variable Types

All variables in your `application.conf` files are parsed as strings, so some reserved values have been created to allow you to return a wider range of types from the `environment.get("KEY", "DEFAULT_VALUE")` function:

    import std.process;
    string huntEnv = environment.get("HUNT_ENV", "");
    if(huntEnv != "") {
        configManager().setAppSection("", DEFAULT_CONFIG_PATH ~ "application." ~ huntEnv ~ ".conf");
    } else {
        configManager().setAppSection("", DEFAULT_CONFIG_PATH ~ "application.conf");
    }

<a name="retrieving-environment-configuration"></a>
### Retrieving Environment Configuration

 You may retrieve values from these variables in your configuration files. In fact, if you review the hunt-framework configuration files, you will notice several of the options already using this helper:

    string path = configManager().config("hunt").file.path.value;

<a name="accessing-configuration-values"></a>
## Accessing Configuration Values

You may easily access your configuration values using the global `config` helper function from anywhere in your application. The configuration values may be accessed using "dot" syntax, which includes the name of the file and option you wish to access. A default value may also be specified and will be returned if the configuration option does not exist:

    string path = configManager().config("hunt").file.path.value;
