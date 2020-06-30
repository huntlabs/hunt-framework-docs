# HTTP Responses

- [Creating Responses](#creating-responses)
    - [Attaching Headers To Responses](#attaching-headers-to-responses)
    - [Attaching Cookies To Responses](#attaching-cookies-to-responses)
    - [JSON Responses](#json-responses)
    - [STRING Responses](#string-responses)
    - [HTML Responses](#html-responses)
    
<a name="creating-responses"></a>
## Creating Responses

Returning a full `Response` instance allows you to customize the response's HTTP status code and headers.

<a name="attaching-headers-to-responses"></a>
#### Attaching Headers To Responses

Keep in mind that most response methods are chainable, allowing for the fluent construction of response instances. For example, you may use the `header` method to add a series of headers to the response before sending it back to the user:

    @Action
    Response myaction5()
    {
        // chained access
        response.html("<h1>show profile </h1>")
        .setHeader("X-Custom-Info", "hello world");

        return response;
    }

<a name="attaching-cookies-to-responses"></a>
#### Attaching Cookies To Responses

The `cookie` method on response instances allows you to easily attach cookies to the response. For example, you may use the `cookie` method to generate a cookie and fluently attach it to the response instance like so:

    @Action
    Response myaction5()
    {
        // chained access
        response.html("<h1>show profile </h1>")
        .setCookie("name", "value", 10000)
        .setCookie("name1", "value", 10000, "/path")
        .setCookie("name2", "value", 10000)

        return response;
    }
    
<a name="json-responses"></a>
### JSON Responses

The `json` method will automatically set the `Content-Type` header to `application/json`, as well as convert the given array to JSON using the `json_encode` ：

    @Action
    Response myaction1()
    {
        auto user = new User;
        user.id = 9;
        user.name = "zoujiaqing";

        return new JsonResponse(user); // respond to a json object
    }
 
<a name="string-responses"></a>
### String Responses

The framework will automatically convert the string into a full HTTP response:

    @Action
    Response myaction3()
    {
        return response.plain("just text"); // return string text
    }
    
<a name="html-responses"></a>
### Html Responses

The framework will automatically convert the html into a full HTTP response:

    @Action
    Response myaction4()
    {
        return response.html("<h1>show profile </h1>"); // return html
    }
