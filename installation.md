# Installation

- [Installation](#installation)
    - [Server Requirements](#server-requirements)
	- [Linux: Installing Dmd](#installing-dmd)
    - [Linux: Installing Hunt](#linux-installing-Hunt)
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
<div class="content-list" markdown="1">
- DMD >= 2.089.1 [Download Page](https://dlang.org/download.html)
</div>

<a name="installing-dmd"></a>
### Linux: Installing Dmd

#### Downloading
Downloading on FreeBSD, Linux and macOS
```
# mkdir -p ~/dlang && wget https://dlang.org/install.sh -O ~/dlang/install.sh
```
Alternatively, specify any version to install:
```
# mkdir -p ~/dlang && wget https://dlang.org/install.sh
# bash ~/dlang/install.sh install dmd-2.091.0
```
Alternatively, the script can be invoked directly:
```
# curl https://dlang.org/install.sh | bash -s
```
#### Activation and Use
Once a compiler is installed, it can be activated for the current session. Useing the -a or --activate option,
This will set up the PATH, LIBRARY_PATH, LD_LIBRARY_PATH, DMD, DC, and PS1 environment variables. It's also possible to combine this into one command:
```
# source ~/dlang/dmd-2.2.091.0/activate
```
Check if the installation is successful，Useing the `--version` option，If version information is displayed, the installation is successful
```
# dmd --version
DMD64 D Compiler v2.091.0
Copyright (C) 1999-2020 by The D Language Foundation, All Rights Reserved written by Walter Bright
# dub --version
DUB version 1.20.0, built on Mar  9 2020
```
The activated compiler can be removed from the current session by restoring the previous environment:
```
# deactivate
```

<a name="linux-installing-Hunt"></a>
### Linux: Installing Hunt

#### Use Dub Create Project
DUB is the D language's official package manager, providing simple and configurable cross-platform builds. DUB can also generate VisualD and Mono-D package files for easy IDE support.
You can use dub to build:
```
# dub init myproject
# cd myproject
# dub add hunt-framework
# rm source/app.d
# touch source/main.d
```
Create your project main file `source/main.d` and install some soruce code
```
module main;

import hunt.framework;

void main(string[] args)
{
    app().run(args);
}
```


Build your project
```
# dub build
```


<a name="configuration"></a>
### Configuration
Add executable to your `dub.json`
```
"targetType": "executable"
```
Add config file to `config/application.conf`
```
http.address = 127.0.0.1
http.port = 8080
```
Add route file to `config/routes`
```
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
```
# mkdir -p source/app/controller
# touch source/app/controller/IndexController.d
```
Add code to `source/app/controller/IndexController.d`
```
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
### Start Program

Use the `dub` command to run your program
```
# dub run
```
Alternatively, Build the project first, After successful construction, executable files will be generated in the current directory, In the `dub.json` file, `name` field is executable name, Run executable
```
# dub build
# ./my-program
```

<a name="browser-access-program"></a>
### Browser Access Program

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
├── dub.json
```
###### If you want to see more information about hunt, you can browse other documents in other directories. Thank you for watching
