# Validation

- [Introduction](#introduction)
- [Form Request Validation](#form-request-validation)
    - [Creating Form Requests](#creating-form-requests)
    - [Customizing The Error Messages](#customizing-the-error-messages)
- [GET Method Validation Params](#get-method-validation-params)
- [Available Validation Rules](#available-validation-rules)
- [Custom Validation Rules](#custom-validation-rules)
    - [Using Rule Objects](#using-rule-objects)

<a name="introduction"></a>
## Introduction

hunt-framework provides several different approaches to validate your application's incoming data. By default, hunt-framework's base controller class uses a `ValidatesRequests` trait which provides a convenient method to validate incoming HTTP requests with a variety of powerful validation rules.

<a name="form-request-validation"></a>
## Form Request Validation

<a name="creating-form-requests"></a>
### Creating Form Requests

Let's add a few validation rules to the `rules` method:
```d
class LoginForm : Form
{
    mixin MakeForm;
    
    @Length(6, 20)
    string username;
    
    @Length(8, 16)
    string password;
}
```

So, how are the validation rules evaluated? All you need to do is type-hint the request on your controller method. The incoming form request is validated before the controller method is called, meaning you do not need to clutter your controller with any validation logic:
```d
module app.controller.UserController;

import hunt.framework;

class UserController : Controller {

    mixin MakeController;

    @Action
    string login(LoginForm loginForm)
    {
        auto validRes = loginForm.valid();
        if(!validRes.isValid)
        {
            logError("Valid error message : " ,validRes.messages());
        }

        // You can edit other logic here
        // And don't forget write "return"
        return "valid is ok";
    }

}
```

Execute valid() to verify. You can get the verification result through isvalid(). Valid returns true, otherwise false. When the validation returns false, you can get the description of the failure reason through messages(); messages() returns an array of strings (eg. string[string] ) :
```d
    if(!validRes.isValid()) {
        auto errors = validRes.messages();
        foreach(error; errors) {
            writeln(error);
        }     
    }
```

<a name="customizing-the-error-messages"></a>
### Customizing The Error Messages

The configuration in form allows you to customize the error messages that the form requests to use:
```d
class LoginForm : Form
{
    mixin MakeForm;
    
    @Length(6, 20, "username vaild false!")
    string username;
    
    @Length(8, 16, "password vaild false!")
    string password;
}
```

<a name="get-method-validation-params"></a>
## GET Method Validation Params

The parameters obtained by get method can also use validation, which is different from 'form validation', and does not need to create a rule file for validation:
```d
module app.controller.UserController;

import hunt.framework;

class UserController : Controller {

    mixin MakeController;

    @Action 
    string addNum(int number, @Length(3, 6) string name) {
        ConstraintValidatorContext context = validate();
        if(context.isValid()) {
            // You can edit other logic here
            return "OK";
        } else {
            return context.toString();
        }
    }

}
```

<a name="available-validation-rules"></a>
## Available Validation Rules

Below is a list of all available validation rules and their function:

    Max(value,message) : Check that the character sequence (e.g. string) validated represents a number, and has a value less than or equal to the maximum value specified.

    Min(value,message) : Check that the character sequence (e.g. string) being validated represents a number, and has a value more than or equal to the minimum value specified.

    NotBlank(message) : Check that a character sequence is not 'null' nor empty after removing any leading or trailing whitespace.

    Email : Checks that a given character sequence (e.g. string) is a well-formed email address.

    AssertFalse(message) : Validates that the value passed is false.

    AssertTrue(message) : Validates that the value passed is true.

    Size(min,max,message) : Check that the length of an array is between min and max.

    Length(min,max,message) : Check that the character sequence length is between min and max.

    Range(min,max,message) : Check that the value of filed is between min and max.

    Pattern(pattern,message) : Check that the character sequence (e.g. string) is match the specified regular expression.

<a name="custom-validation-rules"></a>
## Custom Validation Rules

<a name="using-rule-objects"></a>
### Using Rule Objects

hunt-framework provides a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using rule objects. 

The use method is similar to from validation:
```d
import hunt.validation;

class User : Valid
{
    mixin MakeValid;

    @Length(6, 32,"length must be between {{min}} and {{max}}")
    string username;

    @Range(18, 32)
    int age;

    @email()
    string email;
}

void main()
{
    auto user = new User();

    user.username = "Newton";
    user.age = 377;
    user.email = "newton1643@xxxx.com";

    auto result = user.valid();

    if (result.isValid() == false)
    {
        import std.stdio : writeln;
        foreach(key, message; result.messages())
        {
            writeln("%s: %s", key, message);
        }
    }
}
```
