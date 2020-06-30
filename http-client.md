# HTTP Client

- [Introduction](#introduction)
- [Making Requests](#making-requests)
    - [Request Data](#request-data)
    - [Headers](#headers)
    - [Authentication](#authentication)
    - [Timeout](#timeout)
    - [Retries](#retries)
    - [Error Handling](#error-handling)

<a name="introduction"></a>
## Introduction

Hunt Framework provides an expressive, minimal API around the [Hunt HTTP client](https://github.com/huntlabs/hunt-http/), allowing you to quickly make outgoing HTTP requests to communicate with other web applications. Hunt Framework's wrapper around Hunt HTTP client is focused on its most common use cases and a wonderful developer experience.

Before getting started, you should ensure that you have installed the [Hunt HttpClient](https://github.com/huntlabs/hunt-httpclient) package as a dependency of your application. By default, Hunt Framework Does Not include this dependency:
```sh
dub add hunt-httpclient
```

<a name="making-requests"></a>
## Making Requests

To make requests, you may use the `get`, `post`, `put`, `patch`, and `delete` methods. First, let's examine how to make a basic `GET` request:

```d
import hunt.httpclient;

auto response = Http.get("http://test.com");

```

The `get` method returns an instance of `Response` in module `hunt.httpclient.Response`, which provides a variety of methods that may be used to inspect the response:
```d
string content();
JSONValue json();
int status();
bool isOk();
bool isSuccessful();
bool isFailed();
bool isClientError();
bool isServerError();
Cookie[] cookies();
HttpField[] headers();
string[] header(string name);
```
The `Response` object also overloads the `index operator`, allowing you to access JSON response data directly on the response:

    return Http.get("http://test.com/users/1")["name"];

<a name="request-data"></a>
### Request Data

Of course, it is common when using `POST`, `PUT`, and `PATCH` to send additional data with your request. So, these methods accept an array of data as their second argument. By default, data will be sent using the `application/json` content type:
```d
auto response = Http.post("http://test.com/users", [
    "name" : "Steve",
    "role" : "Network Administrator",
]);
```

#### Sending Form URL Encoded Requests

If you would like to send data using the `application/x-www-form-urlencoded` content type, you should call the `asForm` method before making your request:

```d
auto response = Http.asForm().post("http://test.com/users", [
    "name" : "Sara",
    "role" : "Privacy Consultant",
]);
```

#### Multi-Part Requests

If you would like to send files as multi-part requests, you should call the `attach` method before making your request. This method accepts the name of the file. Optionally, you may provide a third argument which will be considered the custom headers:

```d
auto response = Http.attach(
    "attachment", "photo.jpg"
).post("http://test.com/attachments");
```

<a name="headers"></a>
### Headers

Headers may be added to requests using the `withHeaders` method. This `withHeaders` method accepts an array of key / value pairs:

```d
auto response = Http.withHeaders([
    "X-First" : "foo",
    "X-Second" : "bar"
]).post("http://test.com/users", [
    "name" : "Taylor",
]);
```

<a name="authentication"></a>
#### Bearer Tokens

If you would like to quickly add an `Authorization` bearer token header to the request, you may use the `withToken` method:

```d
auto response = Http.withToken("token").post(...);
```

<a name="timeout"></a>
### Timeout

The `timeout` method may be used to specify the maximum time to wait for a response:

```d
auto response = Http.timeout(3.seconds).get(...);
```

If the given timeout is exceeded, an instance of `TimeoutException` will  be thrown.

<a name="retries"></a>
### Retries

If you would like HTTP client to automatically retry the request if a client or server error occurs, you may use the `retry` method. The `retry` method accepts two arguments: the number of times the request should be attempted and the interval that Hunt Client should wait in between attempts:
```d
auto response = Http.retry(3, 100.msecs).post(...);
```
If all of the requests fail, an instance of `Exception` will be thrown.

<a name="error-handling"></a>
### Error Handling

Hunt's HTTP client wrapper does not throw exceptions on client or server errors (`400` and `500` level responses from servers). You may determine if one of these errors was returned using the `successful`, `clientError`, or `serverError` methods:
```d
// Determine if the status code was >= 200 and < 300...
response.successful();

// Determine if the status code was >= 400...
response.failed();

// Determine if the response has a 400 level status code...
response.clientError();

// Determine if the response has a 500 level status code...
response.serverError();
```
