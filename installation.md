# Installation

- [Server Requirements](#server-requirements)
- [Installing DMD](#installing-dmd)
- [Installing Hunt-Framework](#linux-installing-Hunt)
- [Configuration](#configuration)
- [Coding](#coding)
- [Start Program](#start-program)
- [Browser Access Program](#browser-access-program)
- [Summary](#summary)

<a name="installation"></a>
## Installation

<a name="server-requirements"></a>
### Server Requirements

The Hunt framework has a few system requirements. you will need to make sure your server meets the following requirements:

- DMD >= 2.089.1 [Download Page](https://dlang.org/download.html)

<a name="installing-dmd"></a>
### Installing DMD (on Posix)

#### Using install.sh

```sh
$ mkdir -p ~/dlang && wget https://dlang.org/install.sh -O ~/dlang/install.sh
```

Alternatively, specify any version to install:

```sh
$ mkdir -p ~/dlang && wget https://dlang.org/install.sh
$ bash ~/dlang/install.sh install dmd-2.091.0
```

Alternatively, the script can be invoked directly:
```sh
$ curl https://dlang.org/install.sh | bash -s
```

#### Activation
Once a compiler is installed, it can be activated for the current session. Useing the `-a` or `--activate` option,
This will set up the `PATH`, `LIBRARY_PATH`, `LD_LIBRARY_PATH`, `DMD`, `DC`, and `PS1` environment variables. It's also possible to combine this into one command:
```sh
$ source ~/dlang/dmd-2.091.0/activate
```

Check if the installation is successful，Useing the `--version` option，If version information is displayed, the installation is successful

```sh
$ dmd --version
DMD64 D Compiler v2.091.0
Copyright (C) 1999-2020 by The D Language Foundation, All Rights Reserved written by Walter Bright

$ dub --version
DUB version 1.20.0, built on Mar  9 2020
```

#### Dectivation
The activated compiler can be removed from the current session by restoring the previous environment:
```sh
$ deactivate
```

<a name="linux-installing-Hunt"></a>
### Installing Hunt-Framework

#### Create a empty Hunt Project
The `dub` is the D language's official package manager, which provides a simple and configurable way for cross-platform builds. The `dub` can also generate VisualD and Mono-D package files for easy IDE support.
You can use `dub` to build:

```sh
$ dub init myproject
$ cd myproject
$ dub add hunt-framework
$ rm source/app.d
$ touch source/main.d
```

Create your project main file `source/main.d` and install some soruce code
```d
module main;

import hunt.framework;

void main(string[] args)
{
    app().run(args);
}
```

Build your project
```sh
$ dub build
```

<a name="configuration"></a>
### Configuration
Add executable to your `dub.json`

```json
"targetType": "executable"
```

Add config file to `config/application.conf`
```ini
http.address = 0.0.0.0
http.port = 8080
```

Add route file to `config/routes`
```ini
#
# [GET,POST,PUT,*,...]    path    controller.method
# Symbol* can accept all request method
#

GET   /        index.index
POST  /index   index.index
*    /home     index.index
```

<a name="coding"></a>
### Coding
Create controller file `source/app/controller/IndexController.d`
```sh
$ mkdir -p source/app/controller
$ touch source/app/controller/IndexController.d
```

Add code to `source/app/controller/IndexController.d`
```d
module app.controller.IndexController;

import hunt.framework;

class IndexController : Controller
{
    mixin MakeController;

    @Action
    string index()
    {
        return "Hello world!";
    }
}
```

<a name="start-program"></a>
### Run the Program

Use the `dub` command to run your program
```sh
$ dub run
```
Alternatively, Build the project first, After successful construction, executable files will be generated in the current directory, In the `dub.json` file, `name` field is executable name, Run executable

```sh
$ dub build
$ ./my-program
```

<a name="browser-access-program"></a>
### Acess the Program

View your project using web browser
```
http://127.0.0.1:8080
```
<a name="summary"></a>

### Summary
Project directory structure implemented in this document

```
myproject/
├── config
│   ├── application.conf
│   └── routes
├── source
│   ├── app
│   │   ├── controller
│   │   │   ├── IndexController.d
│   └── main.d
└── dub.json
```

**NOTE**: If you want to see more information about hunt, you can browse other documents in other directories. Thank you for watching
