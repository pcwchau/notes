# Index

- [Index](#index)
- [JSON Libraries](#json-libraries)
  - [Jackson Databind](#jackson-databind)
    - [Serialize](#serialize)
    - [Deserialize](#deserialize)
- [HTTP Client](#http-client)
  - [java.net.http.HttpClient](#javanethttphttpclient)
    - [Create `HttpClient`](#create-httpclient)
    - [Create `HttpRequest`](#create-httprequest)
    - [Send Request](#send-request)
- [Date and Time](#date-and-time)
  - [java.time](#javatime)
- [java.nio](#javanio)
  - [Buffer](#buffer)
  - [ByteBuffer](#bytebuffer)
- [io.netty](#ionetty)
  - [ByteBuf](#bytebuf)
- [Lombok](#lombok)
- [Log info](#log-info)
  - [Logging in console](#logging-in-console)
    - [Write log to file](#write-log-to-file)

# JSON Libraries

## Jackson Databind

- [Jackson Databind - Maven](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind)

### Serialize

Convert object to JSON string.

The following example demonstrates how `int`, `String`, `String[]`, `Object` and `Map<String, Object>` inside an `Object` are converted to JSON string.

```java
package org.example;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.HashMap;
import java.util.Map;


public class Main
{
	public static ObjectMapper mapper = new ObjectMapper();

	public static void main(String[] args)
	{
		try
		{
            String jsonString = mapper.writeValueAsString(new ObjectOne());
			System.out.println(jsonString);
		}
		catch (JsonProcessingException e)
		{
			throw new RuntimeException(e);
		}
	}

	public static class ObjectOne
	{
		public int objectOneInt = 1;
		public String objectOneString = "one";
		public String[] objectOneStringArray = {"two", "three"};
		public ObjectTwo objectTwo = new ObjectTwo();
		public Map<String, Object> objectOneMap = new HashMap<>();
		public Object objectOneNull = null;
		{
			objectOneMap.put("mapString", "four");
			objectOneMap.put("mapInt", 3);
		}
	}

	public static class ObjectTwo
	{
		public int objectTwoInt = 2;
	}
}
```

```json
// The json string is formatted for display. In fact, it is just one line after serializing.
{
    "objectOneInt": 1,
    "objectOneString": "one",
    "objectOneStringArray": ["two", "three"],
    "objectTwo": {
        "objectTwoInt": 2
    },
    "objectOneMap": {
        "mapString": "four",
        "mapInt": 3
    },
    "objectOneNull": null
}
```

### Deserialize

Convert JSON string to Object.

```json
// The json string is formatted for display. In fact, it is just one line after serializing.
{
    "objectOneInt": 1,
    "objectOneString": "one",
    "objectOneStringArray": ["two", "three"],
    "objectTwo": {
        "objectTwoInt": 2
    },
    "objectOneMap": {
        "mapString": "four",
        "mapInt": 3
    },
    "objectOneNull": null
}
```

```java
public static class ObjectOne
{
    public Integer objectOneInt;
    public String objectOneString;
    public String[] objectOneStringArray;
    public ObjectTwo objectTwo;
    public Map<String, Object> objectOneMap;
    public Object objectOneNull;
}

public static class ObjectTwo
{
    public int objectTwoInt;
}
```

```java
try
{
    ObjectOne objectOne = mapper.readValue(jsonString, ObjectOne.class);

    Map<String,Object> result = mapper.readValue(jsonString, Map.class);
    result.containsKey("...");
    result.get("...");
}
catch (JsonProcessingException e)
{
    throw new RuntimeException(e);
}
```

Case 1, data type is *incorrect*:

- Definition of *incorrect*:
  - Requires numeric but get string **that cannot be parsed to numeric**.
  - Requires numeric but get boolean.
  - Requires array but do not get `[...]`.
- `com.fasterxml.jackson.databind.exc.InvalidFormatException: Cannot deserialize value of type ...`
- Object will not be initialized.
- However, the following is *valid*:
  - Requires numeric and get string **that can be parsed to numeric**.
  - Requires integer and get decimal.
  - Requires string and get numeric / boolean.
  - Requires array and get empty array `[]`.

Case 2, data is missing:

- No error. Object will be created. Missing field will be `null` or default value if primitive type.

Case 3, unrecognized data is added:

- `com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException: Unrecognized field ...`
- Object will not be initialized.

Case 4, invalid JSON format:

- `com.fasterxml.jackson.core.JsonParseException: Unrecognized token '...': was expecting (JSON String, Number, Array, Object or token 'null', 'true' or 'false')`
- Object will not be initialized.


# HTTP Client

Some commonly used **HTTP clients** in Java that support synchronous and asynchronous programming models:
- `HttpClient` included from Java 11 for applications written in Java 11 and above
  - native for Java, no need any external libraries
  - support HTTP/2
- Apache `HTTPClient` from Apache HttpComponents project
  - maximum customization and flexibility
- `OkHttpClient` from Square
  - highly  configurable and easier to use
- Spring `WebClient` for Spring Boot applications (included in `webflux`)
  - Spring Boot application and reactive APIs

## java.net.http.HttpClient

- [Class HttpClient - Java 21](https://docs.oracle.com/en/java/javase/21/docs/api/java.net.http/java/net/http/HttpClient.html)
- [Class HttpRequest - Java 21](https://docs.oracle.com/en/java/javase/21/docs/api/java.net.http/java/net/http/HttpRequest.html)

To send a HTTP request:

1. Create `HttpClient`.
2. Create `HttpRequest`.
3. Send request synchronously or asynchronously.

### Create `HttpClient`

In general, we can create a `HttpClient` with the default settings, including: the "GET" request method, a preference of HTTP/2, a redirection policy of NEVER, the default proxy selector, and the default SSL context.

```java
import java.net.http.HttpClient;

HttpClient client = HttpClient.newBuilder().build();
```

Since Java 21, `HttpClient` implements `AutoCloseable`, so you can create and use it inside a try-with-resources block. **However, it will wait until `send` or `sendAsync` have completed execution, which means making `sendAsync` blocking** Not recommended to use it.

### Create `HttpRequest`

```java
import java.net.http.HttpRequest;
import java.net.URI;

// GET
HttpRequest request = HttpRequest.newBuilder()
      .uri(URI.create("http://foo.com/"))
      .timeout(Duration.ofSeconds(30))
      .GET()  // Not support request body
      .build();


// POST
HttpRequest request = HttpRequest.newBuilder()
      .uri(URI.create("http://foo.com/"))
      .timeout(Duration.ofMinutes(2))
      .header("Content-Type", "application/json")                // header
      .POST(HttpRequest.BodyPublishers.ofString("JSON string"))  // request body
      .build();
```

### Send Request

Synchronous Example

```java
// String is the response body type
HttpResponse<String> response = client.send(request, BodyHandlers.ofString());

System.out.println(response.statusCode());
System.out.println(response.body());
```

Asynchronous Example

```java
// String is the response body type
client.sendAsync(request, BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println);
```

# Date and Time

## java.time

- [Package java.time - Java 8](https://docs.oracle.com/javase/8/docs/api/java/time/package-summary.html)

After java 8, do not use `java.util.date`, etc. Instead, use `java.time.xx`, for example `Instant`, `LocalDateTime`, etc

LocalDateTime:
```java
// --- LocalDateTime no concept of time zone ---

// cannot represent an instant on the time-line without additional information such as an offset or time-zone.
import java.time.LocalDateTime;
LocalDateTime.now(); // 2022-08-09T16:59:07.874804600
```

ZonedDateTime:
```java
// --- ZonedDateTime have time zone ---

import java.time.ZonedDateTime;
ZonedDateTime.now(); // 2022-08-09T17:00:30.411807400+08:00[Asia/Shanghai]

ZonedDateTime.now().withHour(0).withMinute(0).withSecond(0).withNano(0) // 2022-08-09T00:00:00.000+08:00[Asia/Shanghai]
```

Instant:
```java
// --- Instant no time zone, always UTC, store as unix timestamp (eg 1658937600000) ---

import java.time.Instant;
ZonedDateTime.now().toInstant(); // 2021-07-15T09:27:04.689450800Z -> now is 17:27 in UTC+8
```

DateTimeFormatter:
```java
// --- DateTimeFormatter ---
// Format time to String

DateTimeFormatter.ISO_OFFSET_DATE_TIME.format(ZonedDateTime.now()) // 2022-08-09T17:28:46.3636532+08:00

// Since Z in original DateTimeFormatter is +0800, not +08:00, so need to hard code '+08:00'
// Whenever need to hardcode time zone in ofPattern(), remember add withZone(ZoneId.of("UTC+?"))

DateTimeFormatter f = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss'+08:00'").withZone(ZoneId.of("UTC+8"));
f.format(ZonedDateTime.now()); // 2022-08-08T17:09:05+08:00

// for Instant / LocalDateTime, the time will change based on withZone()
DateTimeFormatter f = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS'+08:00'").withZone(ZoneId.of("UTC+8"));
f.format(ZonedDateTime.now().toInstant()); // 2022-07-28T00:00:00.000+08:00

DateTimeFormatter f = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS'+08:00'").withZone(ZoneId.of("UTC+9"));
f.format(ZonedDateTime.now().toInstant()); // 2022-07-28T01:00:00.000+08:00


// for YearMonth
// transform 0139 to 203901
DateTimeFormatter f = DateTimeFormatter.ofPattern("yyyyMM");
DateTimeFormatter g = DateTimeFormatter.ofPattern("MMyy");

f.format(YearMonth.from(g.parse("0139")));
```
# java.nio
## Buffer
ref: https://docs.oracle.com/javase/8/docs/api/java/nio/Buffer.html

**Overview**
- A buffer is a linear, `finite` sequence (like array) of elements of a specific primitive type. There is one subclass of this class for each non-boolean primitive type.
- The essential properties of a buffer are its `capacity`, `limit`, `position` and `mark` **(do not exist in Array / List)**:
  - A buffer's `capacity` is the number of elements it contains. The capacity of a buffer is never negative and never changes.
  - A buffer's `limit` is the index of the first element that should not be read or written. A buffer's limit is never negative and is never greater than its capacity.
  - A buffer's `position` is the index of the next element to be read or written. A buffer's position is never negative and is never greater than its limit.
  - A buffer's `mark` is the index to which its position will be reset when the reset method is invoked. The mark is not always defined, but when it is defined it is never negative and is never greater than the position.
  - *0 <= mark <= position <= limit <= capacity*
- Transferring data:
  - `buffer.get()` = channel-write operations = get data from buffer / write data to channel
  - `buffer.put()` = channel-read operations = put data into buffer / read data from channel
  - To put data into a buffer, then get the data out, the operation sequence is `clear` > `put` > `flip` > `get`
- Each subclass of this class defines two categories of `get` and `put` operations:
  - *Relative* operations read or write one or more elements starting at the current position and then increment the position by the number of elements transferred.
  - *Absolute* operations take an explicit element index and do not affect the position.

## ByteBuffer
```java
public abstract class ByteBuffer
extends Buffer
implements Comparable<ByteBuffer>
```
ref: https://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html

**Overview**
- Direct vs. non-direct buffers
  - *Direct* byte buffer - works with memory directly, it is outside JVM (heap) and is not affected by Garbage Collection. To create this buffer, use `ByteBuffer.allocateDirect(int)` which return a new `DirectByteBuffer` (subclass).
  - *Non-direct* byte buffer - it resides in Java Heap memory. To create this buffer, use `ByteBuffer.allocate(int)` which returns a new `HeapByteBuffer` (subclass).

**Example**
```java
public class TestNioRead
{
  private static void readRawBytes() throws IOException
  {
    // This file contains 41 bytes of data
    final FileInputStream fis = new FileInputStream( "D:/Users/peter.chau/work/aPLAYGROUND/testnioread.txt" );
    // allocate a channel to read that file
    FileChannel fc = fis.getChannel();
    // allocate a buffer, as big a chunk as we are willing to handle at a pop.
    ByteBuffer buffer = ByteBuffer.allocate( 1024 * 15 );
    showStats( "newly allocated read", fc, buffer );
    // read a chunk of raw bytes, up to 15K bytes long
    // -1 means eof.
    int bytesRead = fc.read( buffer );
    showStats( "after first read", fc, buffer );
    // flip from filling to emptying
    showStats( "before flip", fc, buffer );
    buffer.flip();
    showStats( "after flip", fc, buffer );
    byte[] receive = new byte[ 20 ];
    buffer.get( receive );
    showStats( "after first get", fc, buffer );
    buffer.get( receive );
    showStats( "after second get", fc, buffer );
    // empty buffer to fill with more data.
    buffer.clear();
    showStats( "after clear", fc, buffer );
    bytesRead = fc.read( buffer );
    showStats( "after second read", fc, buffer );
    // flip from filling to emptying
    showStats( "before flip", fc, buffer );
    buffer.flip();
    showStats( "after flip", fc, buffer );
    fc.close();
  }
  /*
  newly allocated read, channelPosition: 0 bufferPosition: 0 limit: 15360 remaining: 15360 capacity: 15360
  after first read, channelPosition: 41 bufferPosition: 41 limit: 15360 remaining: 15319 capacity: 15360
  before flip, channelPosition: 41 bufferPosition: 41 limit: 15360 remaining: 15319 capacity: 15360
  after flip, channelPosition: 41 bufferPosition: 0 limit: 41 remaining: 41 capacity: 15360
  after first get, channelPosition: 41 bufferPosition: 20 limit: 41 remaining: 21 capacity: 15360
  after second get, channelPosition: 41 bufferPosition: 40 limit: 41 remaining: 1 capacity: 15360
  after clear, channelPosition: 41 bufferPosition: 0 limit: 15360 remaining: 15360 capacity: 15360
  after second read, channelPosition: 41 bufferPosition: 0 limit: 15360 remaining: 15360 capacity: 15360
  before flip, channelPosition: 41 bufferPosition: 0 limit: 15360 remaining: 15360 capacity: 15360
  after flip, channelPosition: 41 bufferPosition: 0 limit: 0 remaining: 0 capacity: 15360
  */
	private static void showStats( String where, FileChannel fc, Buffer b ) throws IOException
	{
		out.println( where +
				", channelPosition: " +
				fc.position() +
				" bufferPosition: " +
				b.position() +
				" limit: " +
				b.limit() +
				" remaining: " +
				b.remaining() +
				" capacity: " +
				b.capacity() );
	}

	public static void main( String[] args ) throws IOException
	{
		readRawBytes();
	}
}
```

# io.netty
## ByteBuf
```java
public abstract class ByteBuf
extends Object
implements ReferenceCounted, Comparable<ByteBuf>, ByteBufConvertible
```
ref: https://netty.io/4.1/api/io/netty/buffer/ByteBuf.html

# Lombok
```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.extern.slf4j.Slf4j;

import lombok.Builder;

@Data  // auto generate getter and setter for a class
@NoArgsConstructor
@RequiredArgsConstructor // only construct final arguments which is not initialized
@AllArgsConstructor
@Slf4j // auto create org.slf4j.Logger logger;
public class Price {}

@Builder
public class PaymentFlow {}
```
- `@Data` already includes `@Getter` and `@Setter`
- For logging, please refer to [log info](#log-info).


# Log info

## Logging in console

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.lang.invoke.MethodHandles;
import lombok.extern.slf4j.Slf4j;

private static final Logger logger = LoggerFactory.getLogger(MethodHandles.lookup().lookupClass());
// --- OR ---
private static final Logger logger = LoggerFactory.getLogger(xxx.class);
// --- OR ---
@Slf4j  // Lombok annotation: private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(xxx.class);


public void functionA() {
   final String METHOD = new Throwable().getStackTrace()[0].getMethodName();  // add method name in each log

   logger.info("{} - Starting doing sth...", METHOD);
   logger.debug("{} - Execution time: {} sec", METHOD, timeElapsed);
   logger.error("{} - CleanClosedRecord Exception: ", METHOD, e);

   // by default, Lombok use variable "log" as logging provider
   log.info("{} - Simple log statement with inputs {}, {} and {}", METHOD, 1, 2, 3);
}
```
- Always add method name at the beginning of your log message (using logger pattern `%M` to add method name may cause effect to performance)
- Add placeholder `{}` to your log message to print value
  - Avoid using string concatenation because it is slower. The `{}` allows delaying the `toString()` call and string construction until the log event is really needed.
  - Note that exception `e` does not need a `{}`, logger will automatically output the exception and the stack trace.
- Declaring loggers as static VS instance variable
  - Static: less CPU and memory overhead
  - Instance variable: possible to take advantage of repository selectors even for libraries shared between applications. IOC-friendly.
    - Not for the `SLF4J`+`log4j` combination.
- `slf4j` is just a log API, `log4j` is a concrete implementation for application layer to configurate log.

### Write log to file

**Background:** Normally, if using K8S, the log printed on shell (console) will not be uploaded to EFK automatically. Therefore, we need to write the log into some files (eg. `xxx.log`), which then are uploaded to EFK. We can use `Log4j 2` to manage these log statements.

For a maven project, we can directly add `log4j2` into `pom.xml`. By default, it will search `log4j2.xml` under classpath (i.e. `/src`).
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn">
    <Properties>
        <Property name="basePath">/apps/logs</Property>
    </Properties>

    <Appenders>
        <RollingFile name="fileLogger" fileName="${basePath}/hktv-finp.log" filePattern="${basePath}/hktv-finp-%d{yyyy-MM-dd}.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5p %c{1}:%L [%t] - %m%n" />
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true" />
            </Policies>
        </RollingFile>

        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5p %c{1}:%L [%t] - %m%n" />
        </Console>
    </Appenders>

    <Loggers>
        <Logger name="org.hibernate" level="info" additivity="true">
            <appender-ref ref="fileLogger" />
        </Logger>
        <Logger name="hk.com.hktvmall.finp" level="debug" additivity="true">
            <appender-ref ref="fileLogger" />
        </Logger>
        <Root level="error">
            <appender-ref ref="console" />
        </Root>
    </Loggers>
</Configuration>
```
- Log level: `TRACE < DEBUG < INFO < WARN < ERROR < FATAL` (The most serious)
- Log pattern layout: https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/PatternLayout.html
- Each logger must append to one appender (eg. `console` and `fileLogger`). Here, all logs with level `debug` or below, which are under the package of `hk.com.hktvmall.finp`, will be written to `fileLogger`. Since its `additivity` is true, all logs with level `error` or below, which are under the package of `hk.com.hktvmall.finp`, will also be appended to the root logger and be written to `console`.

**Reference:**
- https://stackoverflow.com/questions/21881846/where-does-the-slf4j-log-file-get-saved
- (official) https://logging.apache.org/log4j/2.x/manual/configuration.html
