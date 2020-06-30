# HTTP Requests

- [Accessing The Request](#accessing-the-request)
    - [Request Path & Method](#request-path-and-method)
- [Files](#files)
    - [Retrieving Uploaded Files](#retrieving-uploaded-files)
    - [Storing Uploaded Files](#storing-uploaded-files)
    
<a name="accessing-the-request"></a>
## Accessing The Request

To obtain an instance of the current HTTP request via dependency injection, you should type-hint the `Request` class on your controller method. The incoming request instance will automatically be injected by the [service container]:
   
    module app.controller.index;

    import hunt.framework;

    class IndexController : Controller
    {
        mixin MakeController;

        this()
        {
        }

        @Action
        void addPost()
        {
            auto name = request.session().get("name");
            response.html(name ~ "<br> <a href=\"/list\">list</a>");
        }

        @Action
        void del()
        {
            auto id = request.session().get("id");
            response.html(id ~ "<br> <a href=\"/list\">list</a>");
        }
    }
    
<a name="request-path-and-method"></a>
### Request Path & Method

We will discuss a few of the most important methods below.

#### Retrieving The Request Path

The `path` method returns the request's path information. So, if the incoming request is targeted at `http://domain.com/foo/bar`, the `path` method will return `foo/bar`:

    request.path;

#### Retrieving The Request URL

To retrieve the full URL for the incoming request you may use the `url` or `fullUrl` methods. The `url` method will return the URL without the query string, while the `fullUrl` method includes the query string:

    string url = request.url();
    
    string fullUrl = request.fullUrl();

#### Retrieving The Request Method

The `method` method will return the HTTP verb for the request:

    HttpMethod method = request.method();
    
    enum HttpMethod Null = HttpMethod("Null");
    enum HttpMethod GET = HttpMethod("GET");
    enum HttpMethod POST = HttpMethod("POST");
    enum HttpMethod HEAD = HttpMethod("HEAD");
    enum HttpMethod PUT = HttpMethod("PUT");
    enum HttpMethod OPTIONS = HttpMethod("OPTIONS");
    enum HttpMethod DELETE = HttpMethod("DELETE");
    enum HttpMethod TRACE = HttpMethod("TRACE");
    enum HttpMethod CONNECT = HttpMethod("CONNECT");
    enum HttpMethod MOVE = HttpMethod("MOVE");
    enum HttpMethod PROXY = HttpMethod("PROXY");
    enum HttpMethod PRI = HttpMethod("PRI");
    
#### Retrieving The Request session

    request.session().get("key");
 
#### Retrieving The Request cookie
    
    request.cookie().get("name");

<a name="files"></a>
## Files

<a name="retrieving-uploaded-files"></a>
### Retrieving Uploaded Files

You may access uploaded files from a `hunt.framework.http.Request` instance using the `file` method or using dynamic properties. The `file` method returns an instance of the `hunt.framework.file.UploadedFile` class:

    UploadedFile file = request.file("key");
    
    UploadedFile files = request.files("key");
    
You may determine if a file is present on the request using the `hasFile` method:

    if (request.hasFile("key"))
    {
        //    
    }
    
#### Validating Successful Uploads

In addition to checking if the file is present, you may verify that there were no problems uploading the file via the `isValid` method:

    if (request.file("key").isValid()) {
        //
    }    
    
#### File Paths & Extensions

The `UploadedFile` class also contains methods for accessing the file's fully-qualified path and its extension. The `extension` method will attempt to guess the file's extension based on its contents. This extension may be different from the extension that was supplied by the client:

     string extension = request.path("key").extension();
     
#### Other File Methods

There are a variety of other methods available on `UploadedFile` instances.

<a name="storing-uploaded-files"></a>
### Storing Uploaded Files

To store an uploaded file, you will typically use one of your configured . The `UploadedFile` class has a `store` method which will move an uploaded file to one of your disks, which may be a location on your local filesystem or even a cloud storage location like Amazon S3.

The `store` method accepts the path where the file should be stored relative to the filesystem's configured root directory. This path should not contain a file name, since a unique ID will automatically be generated to serve as the file name.

The `store` method also accepts an optional second argument for the name of the disk that should be used to store the file. The method will return the path of the file relative to the disk's root:

    bool stored = request.path("key").store("images");
        
        
