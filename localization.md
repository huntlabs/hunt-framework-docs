# Localization

- [Introduction](#introduction)
    - [Configuring The Locale](#configuring-the-locale)
- [Defining Translation Strings](#defining-translation-strings)
    - [Using Short Keys](#using-short-keys)
    - [Using Translation Strings As Keys](#using-translation-strings-as-keys)
- [Retrieving Translation Strings](#retrieving-translation-strings)
    - [Replacing Parameters In Translation Strings](#replacing-parameters-in-translation-strings)
- [Overriding Package Language Files](#overriding-package-language-files)

<a name="introduction"></a>
## Introduction

Laravel's localization features provide a convenient way to retrieve strings in various languages, allowing you to easily support multiple languages within your application. Language strings are stored in files within the `resources/translations` directory. Within this directory there should be a subdirectory for each language supported by the application:

    /resources
        /translations
            /en-us
                messages.ini
            /zh-cn
                messages.ini

All language files return an array of key-value strings. For example:

```ini
welcome=Hello, world
title = %s Demo
email = Email address
password = Password
submit = Submit
```

> {note} For languages that differ by territory, you should name the language directories according to the ISO 15897. For example, "en_US" should be used for British English rather than "en-us".

<a name="configuring-the-locale"></a>
### Configuring The Locale

The default language for your application is stored in the `config/application.conf` configuration file. You may modify this value to suit the needs of your application. You may also change the active language at runtime using the `setLocale` method in  `hunt.framework.Simplify`.

#### Determining The Current Locale

You may use the `getLocale` to determine the current locale:

    string locale = getLocale();

    if (locale == "en-us") {
        //
    }

<a name="defining-translation-strings"></a>
## Defining Translation Strings

<a name="using-short-keys"></a>
### Using Short Keys

Typically, translation strings are stored in files within the `resources/translations` directory. Within this directory there should be a subdirectory for each language supported by the application:

    /resources
        /translations
            /en-us
                messages.ini
            /zh-cn
                messages.ini

All language files return an array of keyed strings. For example:

```ini
welcome=Hello, world
title = %s Demo
email = Email address
password = Password
submit = Submit
```

<a name="using-translation-strings-as-keys"></a>
### Using Translation Strings As Keys

For applications with heavy translation requirements, defining every string with a "short key" can become quickly confusing when referencing them in your views. For this reason, Laravel also provides support for defining translation strings using the "default" translation of the string as the key.

Translation files that use translation strings as keys are stored as INI files in the `resources/translations` directory. For example, if your application has a Spanish translation, you should create a `resources/translations/es/message.ini` file:

```ini
I-love-programming = Me encanta programar.
```

<a name="retrieving-translation-strings"></a>
## Retrieving Translation Strings

You may retrieve lines from language files using the `trans` helper function. The `trans` method accepts the key of the translation string as its first argument. For example, let's retrieve the `welcome` translation string from the `resources/translations/en-us/messages.ini` language file:
```d
writeln(trans("welcome"));
```

If you are using the [View template](/views.md), you may use the `{{ }}` syntax to echo the translation string:
```html
<title>{{trans("welcome")}}</title>
```

If the specified translation string does not exist, the `trans` function will return the translation string key. So, using the example above, the `trans` function would return `welcome` if the translation string does not exist.


<a name="replacing-parameters-in-translation-strings"></a>
### Replacing Parameters In Translation Strings

If you wish, you may define placeholders in your translation strings. All placeholders are samme format specifications as std.format. For example, you may define a welcome message with a placeholder name:

```ini
welcome=Welcome, %s
```

To replace the placeholders when retrieving a translation string, pass an array of replacements as the second argument to the `__` function:
```d
writeln(trans("welcome", "dayle"));
```

<a name="overriding-package-language-files"></a>
## Overriding Package Language Files

Some packages may ship with their own language files. Instead of changing the package's core files to tweak these lines, you may override them by placing files (ordered by the file name) in the same directory.
