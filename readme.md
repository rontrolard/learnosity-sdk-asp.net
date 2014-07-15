Learnosity SDK - ASP.NET / C#
=============

This repository contains full .NET solution.

LearnositySDK library helps you to generate request JSON for following services:

 * assess
 * author
 * data
 * items
 * questions
 * reports
 * schemas

In addition data and schemas services use `Request/Remote` class to perform HTTP Request, and fetch needed data. Data service can also use `Request/DataApi` wrapper making pagination requests easier.

## Contents

Solution consists of 2 projects: LearnositySDK and LearnositySDKExamples

### LearnositySDK

This project is a class library built in compatibility with .NET 3.5.

This project uses only one external dependency: Newtonsoft.Json library (http://james.newtonking.com/json), also fully compatible with .NET 3.5 (other versions are supported as well).

### LearnositySDKExamples

This is .NET MVC 3 project, built on top of .NET 4.0.

It uses LearnositySDK to demonstrate how the SDK works.

## Installation

The only thing developer needs to do to install LearnositySDK is to add 2 references: LearnositySDK.dll and Newtonsoft.Json.dll.

To do it:
 * right-click your project name in Solution Explorer,
 * click `Add Reference...` option,
 * go to `Browse` tab,
 * search for desired DLLs
 * select DLL files you want to add to your project,
 * click OK.

You can find both libraries in `Dist/All` folder in this repository.

## Usage

### Init class

The `Request/Init` class is used to generate the necessary security and request data (in the correct format) to integrate with any of the Learnosity API services.

The Init constructor takes up to 5 arguments:

 * [string]               service type
 * [string/JsonObject]    security details (**no secret**)
 * [string]               secret
 * [string/JsonObject]    request details *(optional)*
 * [string]               action *(optional)*

```
// prepare all the params
string service = "items";

JsonObject security = new JsonObject();
security.set("consumer_key", "yis0TYCu7U9V4o7M");
security.set("domain", "localhost");
security.set("user_id", "12345678");

string secret = "74c5fd430cf1242a527f6223aebd42d30464be22";

JsonObject pwd = new JsonObject();
pwd.set("pwd", "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8");

JsonObject config = new JsonObject();
config.set("administration", pwd);

JsonObject request = new JsonObject();
request.set("activity_template_id", "demo-activity-1");
request.set("activity_id", "my-demo-activity");
request.set("name", "Demo Activity");
request.set("course_id", "demo_yis0TYCu7U9V4o7M");
request.set("session_id", Uuid.generate());
request.set("user_id", "demo_student");
request.set("config", config);

// Instantiate Init class
Init init = new Init(service, security, secret, request);

// Call the generate() method to retrieve a JavaScript object
string JavaScriptObject = init.generate();
```

On JavaScript side:

```
// Pass the object to the initialisation of any Learnosity API
LearnosityApp.init(JavaScriptObject);
```

#### Arguments

**service**<br>
A string representing the Learnosity service (API) you want to integrate with. Valid options are:

* assess
* author
* data
* items
* questions
* reports

**security**<br>
An associative array^ that includes your *consumer_key* but does not include your *secret*. The SDK sets defaults for you, but valid options are:

* consumer_key
* domain (optional)
* timestamp (optional)
* user_id (optional)

^Note – the SDK accepts JSON strings or JsonObject objects.

**secret**<br>
Your secret key, as provided by Learnosity.

**request**<br>
An optional associative array^ of data relevant to the API being used. This will be any data minus the security details that you would normally use to initialise an API.

^Note – the SDK accepts JSON strings or JsonObject objects.

**action**<br>
An optional string used only if integrating with the Data API. Valid options are:

* get
* set
* update
* delete

<hr>

### Remote class

The Remote class is used to make server side, cross domain requests. Think of it as a HTTPWebRequest or WebClient wrapper.

You'll call either get() or post() with the following arguments:

 * [string]                             URL
 * [string/Dictionary<string, string>]  Data payload
 * [JsonObject]                         Options

```
string url = "http://schemas.learnosity.com/stable/questions/templates";
Remote remote = new Remote();
remote.get(url);
string body = remote.getBody();
```

#### Arguments

**URL**<br>
A string URL, including schema and path. Eg:

```
https://schemas.learnosity.com/stable/questions/templates
```

**Data**<br>
An optional associative array^ of data to be sent as a payload. For GET it will be a URL encoded query string.

^Note – the SDK accepts query strings or Dictionary<string, string> objects.

**Options**<br>
An optional associative array^ of request parameters. Valid options are: int timeout (seconds), JsonObject headers, string encoding

^Note – the SDK accepts JsonObject objects.

### Remote methods
The following methods are available after making a `get()` or `post()`.

**getBody()**<br>
Returns the body of the response payload.

**getError()**<br>
Returns an array that includes the error code and message (if an error was thrown)

**getHeader()**<br>
Currently only returns the *content_type* header of the response.

**getSize()**<br>
Returns the size of the response payload in bytes.

**getStatusCode()**<br>
Returns the HTTP status code of the response.

### DataApi class

This class serves as a wrapper for data API calls. It has two public methods:

**request()**<br>

Allows you to perform single request. For now data API calls are limited to 1000 records, which means, that subsequent calls can be required.

You can pass `data.meta.next` token in your requestPacket to load next set of records. See also `requestRecursive` method.

Check `Examples.Data.DataApi()` for an example.

**requestRecursive()**<br>

Allows you to perform subsequent requests recursively, returning the chunk of data after each recursion.

Chunks are passed into callback function defined by following delegate:

```
public delegate bool ProcessData(string data);
```

Check `Examples.Data.DataApiRecursive()` for an example.

### JsonObject class

Serves as an simple implementation of PHP associative arrays. It provides many overloaded `get()` and `set()` methods to allow you create your flexible objects.

Constructor accpets only parameter: `bool isArray = false`. It's required to set this flag to `true` if your object should behave like an array.

Example with both - objects and arrays:

```
JsonObject session_ids = new JsonObject(true);
session_ids.set("AC023456-2C73-44DC-82DA28894FCBC3BF");

JsonObject report = new JsonObject();
report.set("id", "report-1");
report.set("type", "sessions-summary");
report.set("user_id", "brianmoser");
report.set("session_ids", session_ids);

JsonObject reports = new JsonObject(true);
reports.set(report);

JsonObject request = new JsonObject();
request.set("reports", reports);
```

### JsonObjectFactory class

If you already have your data in JSON string, simply run `fromString()` method, to convert it to JsonObject instance:


```
string json = "[\"a\", \"b\"]";
JsonObject jo = JsonObjectFactory.fromString(json);
```

## Version
Version v0.1.0 - June 2014

## Examples

Each service has it's own example - you can find them in `Examples` folder of LearnositySDK project. To run them simply invoke static `Simple` method.

To see all the examples, you can also run the LearnositySDKExamples project in your local IIS server.