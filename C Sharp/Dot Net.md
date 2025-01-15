# .NET - Index
- [.NET - Index](#net---index)
- [Introduction](#introduction)
  - [Implementations](#implementations)
    - [.NET Core, .NET 5 and later versions](#net-core-net-5-and-later-versions)
    - [.NET Framework](#net-framework)
  - [Application Frameworks](#application-frameworks)
    - [ASP.NET](#aspnet)
    - [ASP.NET Core](#aspnet-core)
- [.NET System Library](#net-system-library)
  - [HTTP Client](#http-client)
    - [HTTP service class](#http-service-class)
    - [API Class](#api-class)
  - [JSON](#json)
    - [Serialize](#serialize)
    - [Deserialize](#deserialize)
    - [Other](#other)
  - [System Directory](#system-directory)
  - [Guid](#guid)
  - [Count Time Elapsed](#count-time-elapsed)
    - [Stopwatch](#stopwatch)
  - [DateTimeOffset](#datetimeoffset)
    - [Unix Timestamp](#unix-timestamp)
    - [Call a web API](#call-a-web-api)
- [NuGet Packages](#nuget-packages)
  - [Configuration File](#configuration-file)
  - [NLog](#nlog)
  - [LiteDB](#litedb)
  - [Quartz.NET](#quartznet)
    - [Job scheduling](#job-scheduling)
  - [MySQL or Related Database](#mysql-or-related-database)
    - [Connection](#connection)
    - [Execute Command](#execute-command)
    - [Retry Logic](#retry-logic)
- [ASP.NET Core](#aspnet-core-1)
  - [Overview](#overview)
  - [Configuration](#configuration)
  - [Middleware](#middleware)
  - [Host](#host)
    - [ASP.NET Core WebApplication](#aspnet-core-webapplication)
      - [Controller](#controller)
      - [Other](#other-1)
    - [ASP.NET Core WebHost](#aspnet-core-webhost)
  - [HttpContext](#httpcontext)
      - [ConnectionInfo](#connectioninfo)
      - [HttpRequest](#httprequest)
      - [HttpResponse](#httpresponse)

# Introduction

- [Document](https://learn.microsoft.com/en-us/dotnet/core/introduction)
- [Glossary 術語](https://learn.microsoft.com/en-us/dotnet/standard/glossary)

.NET is a free, cross-platform, open-source *developer platform* for building many kinds of applications.

Microsoft offers 3 languages on the .NET platform – C#, F# and Visual Basic. With millions of developers, C# is the most popular .NET language.

A .NET app is developed for one or more *implementations* of .NET. There are four .NET *implementations* that Microsoft supports:
- .NET Core, .NET 5 and later versions
- .NET Framework
- Mono
- UWP

Each *implementation* of .NET includes the following components:
- One or more runtimes - for example, .NET Framework CLR and .NET 5 CLR.
- A class library - for example, .NET Framework Base Class Library and .NET 5 Base Class Library.
- Optionally, one or more *application frameworks* - for example, ASP.NET, Windows Forms, and Windows Presentation Foundation (WPF) are included in .NET Framework and .NET 5+.
- Optionally, development tools. Some development tools are shared among multiple implementations.

## Implementations

### .NET Core, .NET 5 and later versions

- [Version support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core)
- [.NET vs. .NET Framework for server apps](https://learn.microsoft.com/en-us/dotnet/standard/choosing-core-framework-server)

.NET 5+, previously referred to as *.NET Core*, is a cross-platform implementation of .NET that's designed to handle server and cloud workloads at scale. It also supports other workloads, including desktop apps. **It runs on Windows, macOS, and Linux.** It implements .NET Standard, so code that targets .NET Standard can run on .NET 5+. ASP.NET Core, Windows Forms, and Windows Presentation Foundation (WPF) all run on .NET 5+.

### .NET Framework

- [Supported versions](https://dotnet.microsoft.com/en-us/learn/dotnet/what-is-dotnet-framework)

.NET Framework is the original .NET implementation that has existed since 2002. **It runs only on Windows.** Versions 4.5 and later implement .NET Standard, so code that targets .NET Standard can run on those versions of .NET Framework. It contains additional Windows-specific APIs, such as APIs for Windows desktop development with Windows Forms and WPF. **.NET Framework is optimized for building Windows desktop applications.**

## Application Frameworks

### ASP.NET

- [Document](https://learn.microsoft.com/en-us/aspnet/overview)

The original ASP.NET implementation that ships with the .NET Framework, also known as ASP.NET 4.x. It uses .NET Framework runtime.

### ASP.NET Core

- [Document](https://learn.microsoft.com/en-us/aspnet/core/)

A cross-platform, high-performance, open-source implementation of ASP.NET. It uses .NET Core runtime.

# .NET System Library

## HTTP Client

- [HttpClient Class - Microsoft](https://learn.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=net-8.0)
- [Make HTTP requests with the HttpClient class - Microsoft](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient)

`HttpClient` is intended to be instantiated once and reused throughout the life of an application to benefit from connection pooling and avoid resource leakage. Therefore, the design architecture:

- One HTTP service class contains the `HttpClient` instance, and methods for sending GET / POST / ... requests.
- API classes that contain methods for different API endpoints.

### HTTP service class

Coding consideration:

- Should the exception be thrown to the caller?
- Should the method return `string` or `HttpResponseMessage`?
- Do not use `JsonContent` for request body. It may have bugs, leading to incorrect cases in the request body. Instead, use `StringContent`.

```cs
using System.Net.Http.Json;
using System.Text.Json;

public class HttpService
{
    private static readonly NLog.Logger logger = NLog.LogManager.GetCurrentClassLogger();

    private static readonly HttpClient _client = new HttpClient();

    public static async Task<string> SendRequest(
        HttpMethod method, string url, object? body, Dictionary<string, string>? header)
    {
        using HttpRequestMessage request = new HttpRequestMessage(method, url);

        if (body != null)
        {
            string jsonBody = JsonSerializer.Serialize(body);
            request.Content = new StringContent(jsonBody, Encoding.UTF8, "application/json");
        }
        if (header != null)
        {
            foreach (var item in header)
            {
                request.Headers.Add(item.Key, item.Value);  // Add header
            }
        }
        HttpResponseMessage response = await _client.SendAsync(request);
        response.EnsureSuccessStatusCode();  // Will throw exception

        string resposneBody = await response.Content.ReadAsStringAsync();
        logger.Info($"Send request - [SUCCESS]" +
            $"\nRequest: {request} \nRequest body: {JsonSerializer.Serialize(body)}" +
            $"\nResponse: {response} \nResponse body: {resposneBody}");

        return resposneBody;
    }

    public static async Task<string> SendGetRequest(
        string url, object? body = null, Dictionary<string, string>? header = null)
    {
        return await SendRequest(HttpMethod.Get, url, body, header);
    }

    public static async Task<string> SendPostRequest(
        string url, object? body = null, Dictionary<string, string>? header = null)
    {
        return await SendRequest(HttpMethod.Post, url, body, header);
    }
}
```

### API Class

When coding for API class, you can use anonymous type or normal `object` as the request body:

```cs
object requestBody = new
{
	appId = AppId
};
```

You can use `Dictionary<string, string>` as the headers:

```cs
Dictionary<string, string> header = new Dictionary<string, string>
{
    { "accessToken", "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9" }
};
```

## JSON

- [How to write .NET objects as JSON (serialize) - Microsoft](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/how-to?pivots=dotnet-8-0)

### Serialize

Convert object to JSON string.

```cs
using System.Text.Json;

class Request
{
    public string pid { get; set; }
    public string action { get; set; }
    public LoginRequestData data { get; set; }
}

class LoginRequestData
{
    public string username { get; set; }
    public string password { get; set; }
    public int type { get; set; }
}


Request obj = ... ;
string jsonString = JsonSerializer.Serialize(obj);
```

```json
// The json string is formatted for display. In fact, it is just one line after serializing.
{
    "pid": "43eae41d-7572-444c-b280-b8f8722ce13d",
    "action": "login",
    "data": {
        "username": "mft",
        "password": "mF123456",
        "type": 1
    }
}
```

- If the type of a property / field is numeric, it would also be numeric type in the JSON string.

### Deserialize

Convert JSON string to object.

This example demonstrates how to convert a client request JSON string into a request object. Based on different action, the structure of the `data` field can be different.

Ideal case, where the JSON string can be deserialized to objects without error:

```json
// The json string is formatted for display. In fact, it is just one line before deserializing.
{
    "Action": "subscribe",
    "Pid": 1,
    "Data": {
        "Symbols": ["EURUSD", "USDCAD"]
    },
    "Comment": "testing"
}
```

```cs
public class Request
{
    public required string Action { get; set; }
    public required uint Pid { get; set; }
    public required object Data { get; set; }
    public          string? Comment {get; set; }
}

public class SubscriptionRequest
{
    public required List<string> Symbols { get; set; }
}
```

```cs
Request? request = JsonSerializer.Deserialize<Request>(jsonString);

// Here just convert the request.Data to JSON, and then convert it to another object type.
SubscriptionRequest = JsonSerializer.Deserialize<SubscriptionRequest>(JsonSerializer.Serialize(request?.Data));
```

Case 1, data type is *incorrect* (e.g. string vs numeric, list needs to use [], etc.):

- Definition of *incorrect*:
  - Require string but get numeric.
  - Require numeric / enum but get string.
  - Require list but do not get `[...]`.
- `System.Text.Json.JsonException: The JSON value could not be converted to ...`
- Object will be `null`.
- Note that for list, it accepts empty list `[]`.
- Note that for enum, it does not check if the value is defined.

Case 2, required data is missing:

- `System.Text.Json.JsonException: JSON deserialization for type ... was missing required properties`
- Object will be `null`.

Case 3, non-required data is missing:

- No error. Object will be created.

Case 4, invalid JSON format:

- `System.Text.Json.JsonException: '"' is invalid after a value. Expected either ',', '}', or ']'.`
- Object will be `null`.

Case 5, useless data is added:

- No error. Object will be created.

### Other

To ignore null property when serializing:
```cs
JsonSerializerOptions jsonOption = new() { DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull };
string message = JsonSerializer.Serialize(response, jsonOption);
```

To use other property name when serializing or deserializing:
```cs
[JsonPropertyName("action")]
public required string Action { get; set; }
```

## System Directory

- [GetCurrentDirectory](https://learn.microsoft.com/en-us/dotnet/api/system.io.directory.getcurrentdirectory?view=net-7.0)

```cs
string path = Directory.GetCurrentDirectory();
// D:\Users\peter.chau\work\mf-mt5-trading-gateway\TradingGateway\bin\x64\Debug\net7.0

// Make sure the directory exists before read / write file inside
if (!Directory.Exists(path))
{
    Directory.CreateDirectory(path);
}
```

## Guid

```cs
using System;

string pid = Guid.NewGuid().ToString();
```

## Count Time Elapsed

### Stopwatch

- [Document](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch.elapsed?view=net-7.0)

It is designed to measure time elapsed with high precision.

```cs
using System;
using System.Diagnostics;

class Program
{
    static void Main(string[] args)
    {
        Stopwatch stopwatch = new Stopwatch();
        stopwatch.Start();

        // Code to be timed goes here
        for (int i = 0; i < 1000000; i++)
        {
            // Do some work
        }

        stopwatch.Stop();
        Console.WriteLine("Elapsed time: {0}", stopwatch.Elapsed);
        Console.WriteLine("Elapsed time: {0}", stopwatch.ElapsedMilliseconds);
    }
}
```

**For multithread program:**
```cs
using System;
using System.Diagnostics;
using System.Threading;

class Program
{
    static void Main(string[] args)
    {
        Stopwatch stopwatch = new Stopwatch();
        stopwatch.Start();

        // Create two threads and start them
        Thread thread1 = new Thread(Worker);
        Thread thread2 = new Thread(Worker);
        thread1.Start();
        thread2.Start();

        // Wait for both threads to finish
        thread1.Join();
        thread2.Join();

        stopwatch.Stop();
        Console.WriteLine("Elapsed time: {0}", stopwatch.Elapsed);
    }

    static void Worker()
    {
        // Code to be timed goes here
        for (int i = 0; i < 1000000; i++)
        {
            // Do some work
        }
    }
}
```

## DateTimeOffset

- [Document](https://learn.microsoft.com/en-us/dotnet/api/system.datetimeoffset)

Suppose the server time now is 2023-11-27 11:03:20 AM, in time zone +8.
```cs
var sw = Stopwatch.StartNew();
sw.Restart();
var a = DateTimeOffset.Now.ToString("G");
Console.WriteLine($"    now: {a}, time: {sw.ElapsedMilliseconds}");

sw.Restart();
var b = DateTimeOffset.UtcNow.ToString("G");
Console.WriteLine($"utc now: {b}, time: {sw.ElapsedMilliseconds}");

//     now: 2023-11-27 11:03:20 AM, time: 18
// utc now: 2023-11-27 3:03:20 AM, time: 0
```

### Unix Timestamp

```cs
var now = DateTimeOffset.Now;
Console.WriteLine(now.ToString());   // 2023-11-28 5:21:26 PM +08:00

var utcnow = DateTimeOffset.UtcNow;
Console.WriteLine(utcnow.ToString());// 2023-11-28 9:21:26 AM +00:00

long time = now.ToUnixTimeSeconds();
Console.WriteLine(time);             // 1701163286

long utctime = utcnow.ToUnixTimeSeconds();
Console.WriteLine(utctime);          // 1701163286

var a = DateTimeOffset.FromUnixTimeSeconds(1701163286);
Console.WriteLine(a.ToString());     // 2023-11-28 9:21:26 AM +00:00
```
- For `ToUnixTimeSeconds`, it will automatically convert the timezone to UTC, so all unix timestamp would be the same even the time zone is different.

### Call a web API

```cs
private static HttpClient client = new HttpClient();

public static async Task<Uri> NotifyAdmin(string msg)
{
    try
    {
        client.BaseAddress = new Uri($"https://rgwapi.m-finance.net/whatsapp/telegram.php?mobileno=-871522503&msg={msg}");

        HttpResponseMessage response = await client.GetAsync("");
        // if success, should be 200 OK
        // cannot send many times in a short period
        response.EnsureSuccessStatusCode();

        return response.Headers.Location;
    }
    catch (Exception ex)
    {
        logger.Warn($"NotifyAdmin [Failed] - {msg}");
        return null;
    }
}
```

# NuGet Packages

## Configuration File

- [Document](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration)

Install the following packages from NuGet:
- Microsoft.Extensions.Configuration
- Microsoft.Extensions.Configuration.Json
- Microsoft.Extensions.Configuration.Binder - it is for binding JSON pattern to data class, i.e. the method `Get<>()`.

Set `appsettings.json` to *copy if newer*.

```cs
using Microsoft.Extensions.Configuration;
using TradingGateway.config;

namespace TradingGateway
{
    public class Program
    {
        public static void Main(string[] args)
        {
            IConfigurationRoot configRoot = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .Build();

            MT5Config? mt5Config = configRoot.GetRequiredSection("MetaTrader5").Get<MT5Config>();
            Console.WriteLine(mt5Config?.Login);  // 1003
        }
    }
}
```

```json
// appsettings.json
{
  "MetaTrader5": {
    "HostName": "192.168.124.221:443",
    "Login": 1003,
    "Password": "D*DyX2Uf"
  }
}
```

```cs
namespace TradingGateway.config
{
    internal class MT5Config
    {
        public required string HostName { get; set; }
        public required int Login { get; set; }
        public required string Password { get; set; }
    }
}
```

## NLog

- [NLog config file](https://github.com/nlog/NLog/wiki/Configuration-file)

Install the following packages from NuGet:
- NLog
- NLog.Web.AspNetCore (if you have web application of ASP.NET Core)

Add a file `NLog.config` in the main directory, set it to *copy if newer*.
```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <!-- If you use ASP.NET core -->
    <extensions>
    <add assembly="NLog.Web.AspNetCore"/>
    </extensions>

	<targets>
		<target name="logColoredConsole"
				xsi:type="ColoredConsole"
				useDefaultRowHighlightingRules="false"
				layout="${longdate}|${pad:padding=5:inner=${level:uppercase=true}}|${callsite} - ${message}${exception:format=tostring}" >
			<highlight-row condition="level == LogLevel.Debug" foregroundColor="DarkGray" />
			<highlight-row condition="level == LogLevel.Info" foregroundColor="Gray" />
			<highlight-row condition="level == LogLevel.Warn" foregroundColor="Yellow" />
			<highlight-row condition="level == LogLevel.Error" foregroundColor="Red" />
			<highlight-row condition="level == LogLevel.Fatal" foregroundColor="Red" backgroundColor="White" />
		</target>

		<target name="logFileWebSocketAudit" 
				xsi:type="File" 
				fileName="${currentdir}/logs/webSocketAudit-${date:format=yyyy-MM-dd-HH}.log"
				layout="${longdate}|${pad:padding=5:inner=${level:uppercase=true}}|${callsite} - ${message}${exception:format=tostring}" />
		
		<target name="logFile" 
				xsi:type="File" 
				fileName="${currentdir}/logs/log-${date:format=yyyy-MM-dd-HH}.log"
				layout="${longdate}|${pad:padding=5:inner=${level:uppercase=true}}|${callsite} - ${message}${exception:format=tostring}" />
	</targets>

	<rules>
		<logger name="*" minlevel="Info" writeTo="logColoredConsole" />
		<logger name="TradingGateway.WebSocketServer.Audit" minlevel="Info" writeTo="logFileWebSocketAudit" final="true" />
		<logger name="*" minlevel="Info" writeTo="logFile" />
	</rules>
</nlog>
```

- `${logger}` includes namespace and class.
- `${callsite}` includes namespace, class and method name.
- Audit log is separated becasue the web socket request / response log is too many.

```cs
namespace TradingGateway
{
    public class Program
    {
        private static readonly NLog.Logger Logger = NLog.LogManager.GetCurrentClassLogger();
        private static readonly NLog.Logger auditlogger = NLog.LogManager.GetLogger("TradingGateway.WebSocketServer.Audit");

        public static void Main(string[] args)
        {
            Logger.Info("hihihi");
            // 2023-10-26 12:15:21.9078|INFO|TradingGateway.Program|hihi

            try {...}
            catch (Exception ex)
            {
                Logger.Error(ex);
                // 2023-10-26 12:15:21.9190|ERROR|TradingGateway.Program|System.Exception: Exception of type 'System.Exception' was thrown.   at TradingGateway.Program.Main(String[] args) in D:\Users\peter.chau\work\mf-mt5-trading-gateway\TradingGateway\Program.cs:line 21
            }
        }
    }
}
```

If you use ASP.NET core:

```cs
var builder = WebApplication.CreateBuilder();
builder.Logging.ClearProviders();  // Add this
builder.Host.UseNLog();  // Add this

vap app = builder.Build();
```

## LiteDB

- [Document](https://www.litedb.org/docs/)

LiteDB is a simple, fast and lightweight embedded .NET **NoSQL database**. LiteDB was inspired by the MongoDB database and its API is very similar to the official MongoDB .NET API.

Install the following package from NuGet:
- LiteDB

Remarks:
- To update a document, add `[BsonId]` in the property of the model as the object ID.

## Quartz.NET

- [Document](https://www.quartz-scheduler.net/)

Any job that isn’t an arbitrarily simple task is going to require some processing power that could affect our application performance. This is not ideal for users of our application as it can affect their user experience and interactions.

For example, if we have a file upload function in our application that allows users to upload a profile picture, we don’t want the user to wait around and be unable to interact with the application. Instead, we can schedule a job to upload this in the background, and the user can continue to use the application without any performance implications.

There are many other use cases for job scheduling, but the important thing to understand is these jobs can be long-running, and the best practice is to run them in the background, usually initiated by some sort of trigger.

Install the following package from NuGet:
- Quartz 3.8.0

### Job scheduling
```cs
using Quartz;
using Quartz.Impl;
using System;
using System.Threading.Tasks;

public class program
{
    public static void Main(string[] args)
    {
        ...
        var housekeepTask = Task.Run(() => JobStart(mt5ChartServer));
    }

    public async Task JobStart()
    {
        ISchedulerFactory schedulerFactory = new StdSchedulerFactory();
        IScheduler scheduler = await schedulerFactory.GetScheduler();
        await scheduler.Start();

        IJobDetail jobDetail = JobBuilder.Create<SomeJob>().Build();
        // here is passing parameter to the job class
        jobDetail.JobDataMap["pumpManager"] = pumpManager;
        jobDetail.JobDataMap["databaseManager"] = databaseManager;

        ITrigger trigger = TriggerBuilder.Create()
            .WithIdentity("SomeJob ", "GreetingGroup")
            .WithCronSchedule("0/5 * * * * ? *")
            .StartAt(DateTime.UtcNow)
            .WithPriority(1)
            .Build();

        await scheduler.ScheduleJob(jobDetail, trigger);
    }

    public class SomeJob : IJob
    {
        public async Task Execute(IJobExecutionContext context)
        {
            // here is receiving the parameter
            JobDataMap jobDataMap = context.JobDetail.JobDataMap;
            PumpManager pumpManager = (PumpManager)jobDataMap["pumpManager"];
            DatabaseManager databaseManager = (DatabaseManager)jobDataMap["databaseManager"];

            await Console.Out.WriteLineAsync("EEEEEE testing");
        }
    }
}
```
- For cron expression, please refer to notes `common.md`.

## MySQL or Related Database

- [MySqlConnector Document](https://mysqlconnector.net/)
- [MySqlConnector vs MySql.Data](https://mysqlconnector.net/tutorials/migrating-from-connector-net/)

MySqlConnector is a truly async MySQL ADO.NET provider, supporting MySQL Server, MariaDB, Amazon Aurora, Azure Database for MySQL, Google Cloud SQL, and more. 

Install the following package from NuGet:
- MySqlConnector 2.3.1 https://www.nuget.org/packages/MySqlConnector/2.3.1

### Connection

- Remember to separate the database logic in a datebase class.
-  Connection pooling is used automatically by ADO.NET, you can change the pool size `MaximumPoolsize` in the connection string, which is `100` by default. Larger `MaximumPoolsize`, should be better performance.

```cs
using MySqlConnector;

public class DatabaseManager
{
    private readonly string ConnectionString = 
    "server=192.168.124.109;port=3306;database=algobd;uid=root;" + 
    "password=mF34266223ok;SslMode=none;Pooling=true;MaximumPoolsize=100;"

    public Task SomeOperationAsync()
    {
        // Please refer to the next section of execute command ...
    }
}
```

### Execute Command

- Always use `*Async` when possible.
- Always use `using` to handle the lifetime of connection. This may avoid the error of `Too many connections`.
- You may add retry in case of error `Too many connections`.
- [Connection Reuse](https://mysqlconnector.net/troubleshooting/connection-reuse/) :
  - In most case, for each operation, use a new `MySqlConnection` and call `OpenAsync`. Enable the connection pool. This may avoid the error of `This MySqlConnection is already in use`.
  - If you ensure the code sequentially awaits each asychronous operation (e.g. in the same method), you may use a `MySqlConnection` for multiple operations. **It can be faster.**

```cs
// SELECT
try
{
    using var connection = new MySqlConnection(ConnectionString);
    await connection.OpenAsync();
    using var command = connection.CreateCommand();

    command.CommandText = @"SELECT * FROM orders WHERE order_id = @OrderId;";
    command.Parameters.AddWithValue("@OrderId", orderId);

    using var reader = await command.ExecuteReaderAsync();
    while (reader.Read())
    {
        var id = reader.GetInt32("order_id");
        var date = reader.GetDateTime("order_date");
        // ...
    }
}
catch (Exception ex)
{
    // return something special e.g. `-1`.
    Logger.Error(ex);
    return -1;
}
```

- If no row is returned when `select`, the `reader.Read` will return `false` at the first time.

```cs
// INSERT, UPDATE, DELETE
try
{
    using var connection = new MySqlConnection(ConnectionString);
    await connection.OpenAsync();
    using var command = connection.CreateCommand();

    command.CommandText = @"INSERT INTO account SET (`user_id`, `money`) VALUES (@user_id, 1000)";
    command.Parameters.AddWithValue("@user_id", uuid);

    int result = await command.ExecuteNonQueryAsync();  // the return value is the number of affected rows
}
catch (Exception ex)
{
    // return something special e.g. `-1`.
}
```

- For many insert, consider bulk insert.

### Retry Logic

Before adding retry logic, your program must distinguish between transient errors versus persistent errors. If you program has a misspelling of the target database name, the "No such database found" error will persist and will reoccur every time you rerun the program.

# ASP.NET Core

- Before development, read this [Use the ASP.NET Core shared framework in .NET 5+](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/target-aspnetcore#use-the-aspnet-core-shared-framework).

## Overview

- [Document](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/)

```cs
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.Add...();

// Dependency injection ...

var app = builder.Build();

// Configure the HTTP request pipeline / middleware.
app.Use(...);

// Middleware ...

app.Run();
```

## Configuration

```json
// appsettings.json
{
  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://localhost:9999"
      }
    }
  },
  "...": {

  }
} 
```

## Middleware

- [Document](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/)

The request handling pipeline is composed as a series of request delegates (middleware components) using `Run`, `Map`, and `Use` extension methods. Each component:
- Performs operations on an `HttpContext`
- Either invokes the next delegate in the pipeline or terminates the request.
- Can perform work before and after the next delegate in the pipeline.

`Use` adds the middleware to the application request pipeline. The `next` parameter represents the next delegate in the pipeline.

`Run` adds a terminal middleware delegate to the application's request pipeline. It doesn't receive a next parameter. If another `Use` or `Run` delegate is added after the Run delegate, it's not called.

```cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Other middlewares

app.Use(async (context, next) =>
{
    // Do work that can write to the Response.
    await next.Invoke();
    // Do logging or other work that doesn't write to the Response.
});

app.Run(async context =>
{
    await context.Response.WriteAsync("Hello from 2nd delegate.");
});

app.Run();  // This is not related to request delegate
```

When a delegate doesn't pass a request to the next delegate, it's called *short-circuiting the request pipeline*. For example:

- Using `Use` without calling `next.Invoke()`.
- Using `Run`.

Short-circuiting is often desirable because it avoids unnecessary work. For example, **Static File Middleware** can act as a terminal middleware by processing a request for a static file and short-circuiting the rest of the pipeline. Middleware added to the pipeline before the middleware that terminates further processing still processes code after their `next.Invoke` statements.

The order that middleware components are added in the `Program.cs` file defines the order in which the middleware components are invoked on requests and the reverse order for the response. The order is critical for security, performance, and functionality.

## Host

There are three different hosts capable of running an ASP.NET Core app:

- ASP.NET Core `WebApplication`, also known as the Minimal Host.
- .NET Generic Host combined with ASP.NET Core's ConfigureWebHostDefaults
- ASP.NET Core WebHost

The ASP.NET Core `WebApplication` is recommended and used in all the ASP.NET Core templates. `WebApplication` behaves similarly to the .NET Generic Host and exposes many of the same interfaces but requires less callbacks to configure. The ASP.NET Core WebHost is available only for backward compatibility.

### ASP.NET Core WebApplication

- [Document](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/webapplication)
- [Tutorial: Create a minimal API with ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/tutorials/min-web-api)
- [Tutorial: Create a controller-based web API with ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/web-api/)
- quick ref: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
  - Parameter binding: route value, query parameter, body (JSON)
- create responses: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/responses

```cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.MapGet("/users/{login}/items/{itemName}", (int login, string itemName) =>
        {
            return Results.Ok();
        });

app.Run();
```

The `WebApplicationBuilder.Build` method configures a host with a set of default options, such as:
- Use Kestrel as the web server and enable IIS integration.
- Load configuration from `appsettings.json`, environment variables, command line arguments, and other configuration sources.
- Send logging output to the console and debug providers.

#### Controller



#### Other

Do not change the property name when giving JSON response.
```cs
builder.Services.Configure<Microsoft.AspNetCore.Http.Json.JsonOptions>(options =>
{
	options.SerializerOptions.PropertyNamingPolicy = null;
});
```


### ASP.NET Core WebHost

- [Document](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/web-host)

```cs
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}

public class Startup
{
    // This method gets called by the runtime. Use this method to add services to the container.
    // For more information on how to configure your application, visit http://go.microsoft.com/fwlink/?LinkID=398940
    public void ConfigureServices(IServiceCollection services)
    {
        ...
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
        app.Use(async (context, next) =>
        {
            using (WebSocket webSocket = await context.WebSockets.AcceptWebSocketAsync())
            {
                var socketFinishedTcs = new TaskCompletionSource<object>();
                BackgroundSocketProcessor.AddSocket(webSocket, socketFinishedTcs);
                await socketFinishedTcs.Task;
            }
        });
    }
}
```

## HttpContext

- [Document](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/use-http-context)

#### ConnectionInfo

`HttpContext.Connection` returns a `ConnectionInfo` object.

```cs
{
    app.Run(async context => {
        // Get IP address
        string ipAddress = context.Connection.RemoteIpAddress.ToString();

    })
}
```

#### HttpRequest

`HttpContext.Request` returns a `HttpRequest` object.

`HttpRequest` has information about the incoming HTTP request, and it's initialized when an HTTP request is received by the server. `HttpRequest` isn't read-only, and middleware can change request values in the middleware pipeline.

```cs
{
    app.Run(async context => {
        // Get request path
        string path = context.Request.Path.Value;

        // Get request method: "GET", "POST"
        string method = context.Request.Method;

        // Get request headers
        string authHeader = context.Request.Headers["Authorization"];

    })
}
```

#### HttpResponse

`HttpContext.Response` returns a `HttpResponse` object.

`HttpResponse` is used to set information on the HTTP response sent back to the client.

```cs
using System.Net;

{
    app.Run(async context => {
        // Set response status code
        context.Response.StatusCode = (int)HttpStatusCode.Forbidden;

        // Set response header
        context.Response.Headers.Add("Content-Type", "application/json");

        // Set resposne body
        await context.Response.WriteAsync(JsonSerializer.Serialize(new
        {
            success = false,
            errorMsg = "system error"
        }));

        // Send response
        await context.Response.CompleteAsync();
    })
}
```
