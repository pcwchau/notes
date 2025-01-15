# Index
- [Index](#index)
- [Overview](#overview)
- [Beans and Dependency Injection](#beans-and-dependency-injection)
  - [Constructor-based Dependency Injection](#constructor-based-dependency-injection)
  - [Field-based Dependency Injection](#field-based-dependency-injection)
  - [Annotations](#annotations)
- [Other Feature](#other-feature)
- [Quick Start API Server](#quick-start-api-server)
- [Configuration File](#configuration-file)
- [Web Application](#web-application)
  - [Folder Structure](#folder-structure)
  - [Get Request](#get-request)
    - [@PathVariable](#pathvariable)
    - [@RequestParam](#requestparam)
    - [@RequestBody](#requestbody)
  - [Return Response](#return-response)
    - [ResponseEntity](#responseentity)
- [Validation](#validation)
- [Webflux](#webflux)
  - [Overview](#overview-1)
  - [Methods](#methods)
  - [Testing - Unit test](#testing---unit-test)
  - [Testing - Blockhound](#testing---blockhound)
  - [Service Example](#service-example)
  - [Controller](#controller)
  - [Error Handling](#error-handling)
- [SQL Databases](#sql-databases)
  - [Java Persistence API (JPA)](#java-persistence-api-jpa)
  - [Hibernate (JPA)](#hibernate-jpa)
  - [Spring Data JPA](#spring-data-jpa)
    - [Entity Classes](#entity-classes)
    - [ID generation strategy](#id-generation-strategy)
    - [Dao](#dao)
      - [Query](#query)
      - [Insert](#insert)
      - [Update](#update)
      - [Delete](#delete)
  - [Dao implementation](#dao-implementation)
  - [Service implementation](#service-implementation)
- [MongoDB](#mongodb)
- [SSL](#ssl)
- [MD5 hash of the parameters for Signature](#md5-hash-of-the-parameters-for-signature)
- [Unit Test](#unit-test)
  - [Service](#service)
  - [Webflux Controller](#webflux-controller)
- [Load Test (JMeter)](#load-test-jmeter)
- [Cache](#cache)
  - [Caffeine](#caffeine)
- [Actuator](#actuator)
  - [Set metrics to check cache](#set-metrics-to-check-cache)
- [Rate limiter](#rate-limiter)
  - [Resilience4j](#resilience4j)
- [Scheduler Cron Job](#scheduler-cron-job)
  - [ShedLock](#shedlock)
- [Security](#security)
- [CORS](#cors)
- [Useful website/tutorial](#useful-websitetutorial)
- [Other related concepts](#other-related-concepts)
- [Spring](#spring)
- [Appendix](#appendix)
  - [What is Dependency Injection](#what-is-dependency-injection)

# Overview

- [Spring Framework Documentation (2002 - 2024) - Spring](https://docs.spring.io/spring-framework/reference/index.html)
- [Spring Boot Reference Documentation (2012 - 2024) - Spring](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [Spring Boot Reference Documentation (2012 - 2018) - Spring](https://docs.spring.io/spring-boot/docs/2.0.x/reference/html/index.html)
- [Spring Boot Reference Documentation (2013 - 2014) - Spring](https://docs.spring.io/spring-boot/docs/1.1.0.M1/reference/html/index.html)

Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".

Features:

- Create stand-alone Spring applications
- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)
- Provide opinionated 'starter' dependencies to simplify your build configuration
- Automatically configure Spring and 3rd party libraries whenever possible
- Provide production-ready features such as metrics, health checks, and externalized configuration
- Absolutely no code generation and no requirement for XML configuration

Overall, Spring Boot simplifies and accelerates the development process, enhances productivity, and provides a solid foundation for building scalable and maintainable Java applications.

- command (maven):
  - directly run: `./mvnw spring-boot:run`
  - build jar: `./mvnw clean package` and `java -jar target/gs-serving-web-content-0.1.0.jar`
- **Remark**: application.java (main class) must be in root package (above other classes)

# Beans and Dependency Injection

Please refer to [What is Dependency Injection](#what-is-dependency-injection).

It is generally recommended using constructor injection to wire up dependencies and `@ComponentScan` to find beans.

You can add `@ComponentScan` without any arguments or use the `@SpringBootApplication` annotation which implicitly includes it. All of your application components (`@Component`, `@Service`, `@Repository`, `@Controller`, and others) are automatically registered as Spring Beans.

**Main Application**

```java
// Same as @SpringBootConfiguration @EnableAutoConfiguration @ComponentScan
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

## Constructor-based Dependency Injection

*Constructor-based DI* is accomplished by the container invoking a constructor with a number of arguments, each representing a dependency.

```java
@Service
public class MyAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public MyAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

You might consider using `@RequiredArgsConstructor` provided by Lombok if it is a base class.

If a bean has more than one constructor, you will need to mark the one you want Spring to use with `@Autowired`:

```java
@Service
public class MyAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    private final PrintStream out;

    @Autowired
    public MyAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
        this.out = System.out;
    }

    public MyAccountService(RiskAssessor riskAssessor, PrintStream out) {
        this.riskAssessor = riskAssessor;
        this.out = out;
    }

    // ...

}
```

For abstract class, it is not required to be a bean. Only its subclasses need to be registered as beans and injected dependencies.

```java
public abstract class NotificationService {

    protected final NotificationDao notificationDao;

    public NotificationService(NotificationDao notificationDao) {
        this.notificationDao = notificationDao;
    }

    // ...

}

@Service
public class PushyService extends NotificationService {

    private final PushyAPI pushyAPI;

    public PushyService(PushyAPI pushyAPI, NotificationDao notificationDao)
    {
        super(notificationDao);
        this.pushyAPI = pushyAPI;
    }

    // ...

}
```

## Field-based Dependency Injection

```java
@Service
public class DatabaseAccountService implements AccountService {

    @Autowired
    private RiskAssessor riskAssessor;

    @Autowired
    private TransactionService transactionService;

    // ...

}
```

Why do not use field-based dependency injection?
- It cannot declare immutable dependencies
- You cannot use the class outside the application container, such as for unit testing, you need to use a Spring container additionally.

Reference: https://blog.marcnuri.com/field-injection-is-not-recommended

## Annotations

`@Component`
- It is a generic annotation used to indicate that a class is a Spring-managed component. It is the most basic annotation for declaring a bean in Spring, and it can be used to annotate any class that should be managed by the Spring container.
- `@Service` is a specialization of the `@Component` annotation and is used to indicate that a class is a service component.


`@Qualifier`
- It distinguishes between multiple beans of the same type that are defined in a Spring application context

  For example, you have two implementations of `SessionFactory` interface: `paymentSessionFactory` and `riskSessionFactory`. If you define both beans in your Spring application context, Spring will not know which one to inject into a `PaymentInfoDaoImpl` repository that has a dependency on the `SessionFactory` interface.
  ```java
  @Repository
  public class PaymentInfoDaoImpl implements PaymentInfoDao
  {
      private final SessionFactory sessionFactory;

      public PaymentInfoDaoImpl(@Qualifier("paymentSessionFactory") SessionFactory sessionFactory)
      {
        this.sessionFactory = sessionFactory;
      }

      // ...
  }


  // Here are two different SessionFactory beans
  // To use @Qualifier, you need to define the beans with unique names. 
  @Configuration
  public class DatebaseConfig {
    
      @Bean(name = "paymentSessionFactory")
      @ConfigurationProperties("hibernate.payment")
      public LocalSessionFactoryBean paymentSessionFactory()
      {
        LocalSessionFactoryBean sessionFactoryBean = new LocalSessionFactoryBean();
        sessionFactoryBean.setDataSource(firstDataSource());
        return sessionFactoryBean;
      }
    
      @Bean(name = "riskSessionFactory")
      @ConfigurationProperties("hibernate.risk")
      public LocalSessionFactoryBean riskSessionFactory()
      {
        // ...
      }
  }
  ```

# Other Feature

- Actuator
- Cache
- CORS
- Housekeeping (DB)
- Rate limiter
- Security
- Stat
- Load test (Jmeter, etc)

# Quick Start API Server

- [Spring Initializr](https://start.spring.io/)
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot)

To create a simple web application, follow the steps:

1. Set up the `pom.xml`.
2. Set up the main application class.
3. Create a web controller class.

Set up `pom.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.2.3</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>21</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

Set up the main application class:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Create a web controller class:

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

	@GetMapping("/")
	public String index() {
		return "Greetings from Spring Boot!";
	}
}
```

After finishing the above steps, run the Java, then you have already created a web application.

# Configuration File

- [Externalized Configuration - Spring](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.external-config)
- Common `application.yml` properties: https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties

By default, there is a `application.properties` file in `\src\main\resources`. You may change it to YAML file `application.yml`.

# Web Application

Flow: controller -> service -> dao (database)

To create a web application:

1. Define all API endpoints.
2. Define different controllers.
3. Define request and response models for the controllers.
4. Classify the business logic into different services.
5. Define models and exceptions for the services.

## Folder Structure

```
<root_folder>
└── src
    └── main
        ├── java
        │   └── com.example
        │       ├── annotation
        │       ├── api -------- API call to external system
        │       ├── config
        │       ├── constant --- public static final ...
        │       ├── client ----- Handle all HTTP request / response behind service layer
        │       ├── controller
        │       ├── dao -------- Handle all actions that access DB (query/update) behind service layer
        │       ├── dto -------- Entity class that directly store data from DB, usually POJO
        │       ├── enums ------ Define type if it is only used in argument of switch()
        │       ├── exception
        │       ├── handler ---- E.g. exception handler
        │       ├── job -------- Usually for cron job
        │       ├── model
        │       │   ├── ...
        │       │   ├── request
        │       │   └── response
        │       ├── service
        │       ├── utils ------ E.g. log request
        │       └── Application.java
        └── resources
            └── application.properties / application.yml --- will put to manifest/config in K8S
```

## Get Request

To receive requests, we can use `@RestController` to define a controller class. Number of thread is configured by tomcat.

```java
@RestController
@RequestMapping("/notification")
public class NotificationController
{
	// ...
}
```

Inside the controller class, we can define different API endpoints. We can use the following to receive request data:

- `@PathVariable`
- `@RequestParam`
- `@RequestBody`

Note that for `@PathVariable` and `@RequestParam`, if the target variable is not `String`, it will apply auto conversion.

- Only successful type conversion will continue running the method.
- Failed conversion will throw an exception `MethodArgumentTypeMismatchException`.

If the parameter is required, it must either have a default value or be present in the request, otherwise it will throw an excpetion `MissingServletRequestParameterException`.

If the parameter is not required, and is not present, the target variable will have a `null` value. Note that the target variable must be able to be translated into a null value, which means it cannot be a primitive type.

### @PathVariable

- [Spring @PathVariable Annotation - baeldung](https://www.baeldung.com/spring-pathvariable)

```java
@PostMapping("/send/{companyId}/{serviceId}")
public ResponseEntity send(
    @PathVariable final Integer companyId,
    @PathVariable final Integer serviceId)
{
  log.info("received a new request, company {} service {}", companyId, serviceId);
}


// Result:
// http://localhost:8080/notification/send/1/1

// received a new request, company 1 service 1
```

### @RequestParam

- [@RequestParam - Spring](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/requestparam.html)

```java
@PostMapping("/send")
public ResponseEntity send(
    @RequestParam("company_id") final Integer companyId,
    @RequestParam("service_id") final Integer serviceId)
{
  log.info("received a new request, company {} service {}", companyId, serviceId);
}

// Result:
// http://localhost:8080/notification/send?company_id=1&service_id=1

// received a new request, company 1 service 1
```

### @RequestBody

- [@RequestBody - Spring](https://docs.spring.io/spring-framework/reference/web/webflux/controller/ann-methods/requestbody.html)

```java
@PostMapping("/send")
public void send(@RequestBody final SendRequest sendRequest)
{ 
  // ...
}
```

It uses Jackson Databind library to deserialize the request JSON body.

- If data is missing, no error. Target variable will be created. Missing field will be `null`.
- If unrecognized data is added, no error.

To ensure no required data is missing in the request body, you can use `jakarta.validation.Valid` annotation, as the following example shows. By default, validation errors cause a `MethodArgumentNotValidException`, which is turned into a 400 (BAD_REQUEST) response.

```java
import jakarta.validation.Valid;

@PostMapping("/send")
public void send(@Valid @RequestBody final SendRequest sendRequest)
{ 
	// ...
}
```

```java
import jakarta.validation.constraints.NotEmpty;

public class SendRequest
{
	@NotEmpty
	public String[] targets;

	@NotEmpty
	public String message;

	public String redirectTo;
}
```

For more validation rule, please refer to [Validation](#validation).

## Return Response

To return response:

1. Define response models for success and error/exception responses.
2. Define all customized exceptions.
3. Define an exception handler for all error/exception responses.
4. After the request is processed, either return a `ResponseEntity` or throw an exception, in the controller method.

Define response model:

Since we probably want to return a JSON response, we need to create some response models. Here, we follow the REST API notes to create a uniform response structure for both success and error/exception responses.

```java
// ref to REST API notes

import com.fasterxml.jackson.annotation.JsonInclude;

@JsonInclude(JsonInclude.Include.NON_NULL)
public class Response
{
    public String status;
    public Object data;
    public String code;  // for status == error
    public String message;  // for status == error
}
```

- `@JsonInclude(JsonInclude.Include.NON_NULL)` This is to remove null objects in JSON response.


Suppose all controllers will only throw one type of exception: `ApiErrorException`.

Then, define an exception handler for all error/exception responses.

```java
@Order(Ordered.HIGHEST_PRECEDENCE)
@ControllerAdvice
public class RestResponseEntityExceptionHandler extends ResponseEntityExceptionHandler
{
    @ExceptionHandler(ApiErrorException.class)
    protected ResponseEntity<Response> handleApiError(ApiErrorException e)
    {
        Response responseBody = new Response();
        responseBody.setStatus(Constant.RESPONSE_STATUS_FAIL);
        responseBody.setMessage(e.getMessage());

        return new ResponseEntity<>(responseBody, HttpStatus.BAD_REQUEST);
    }
}
```

- `@ControllerAdvice` This tells Spring that this handler is for controller bean.
- `@Order(Ordered.HIGHEST_PRECEDENCE)` This tells Spring to use this exception handler as default.

Finally, either throw an exception or return a `ResponseEntity` in the controller method.

```java
@PostMapping("/sendMessage")
public ResponseEntity<Response> sendMessage(...) throws ApiErrorException
{
    try
    {
        ...
        Response responseBody = new Response();
        responseBody.setStatus(Constant.RESPONSE_STATUS_SUCCESS);
        responseBody.setData(service.sendMessage(sendMessageRequest));
    }
    catch (Exception e)
    {
        log.error(...);
        throw new ApiErrorException(e.getMessage());
    }
    
    return new ResponseEntity<>(responseBody, HttpStatus.OK);
}
```

### ResponseEntity

- [ResponseEntity - Spring Java Doc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html)


```java
@PostMapping("/sendMessage")
public ResponseEntity<Response> sendMessage(...)
{
    ...
    Response responseBody = new Response();
    responseBody.setStatus(Constant.RESPONSE_STATUS_SUCCESS);
    responseBody.setData(service.sendMessage(sendMessageRequest));

    return new ResponseEntity<>(responseBody, HttpStatus.OK);
}

// Result:
200 OK
{
    "status": "success",
    "data": "{\"success\":true,\"id\":\"65e5a8d713e78e53fe19a1fa\",\"info\":{\"devices\":2}}"
}
```

# Validation

- https://jakarta.ee/specifications/bean-validation/3.0/apidocs/jakarta/validation/constraints/package-summary

1. Add maven dependency.
2. Use annotation to add constraints.
3. Use `@Valid` annotation to validate object.

Add maven dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Use annotation to add constraints:

```java
import jakarta.validation.constraints.NotNull;


public class SendRequest
{
	@NotNull  // Ensure not null
	public String[] targets;

	@NotEmpty  // Ensure not null nor empty (Array length is evaluated)
	public String[] targets;

	@NotEmpty
	public String message;
}
```

Use `@Valid` annotation to validate object, such as `@RequestBody`.

# Webflux

- Unlike Spring boot MVC, webflux is non-blocking
- No need wait for all data finished, can send data as streaming.
- When to use
  - Scalability
    - IO bound operations
    - Efficient CPU utilization
  - Streaming use-cases (Live source of data)
  - Backpressure (ask producer to slow down request)
- Disadvantages
  - End-to-end reactive required
  - Not too useful for CPU-bound flows
- doc: https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html
- operator reference: https://projectreactor.io/docs/core/release/reference/index.html#which-operator

## Overview
**Return to front end**
- If front end support receive streaming data, backend can return `Flux`, otherwise default return `Mono`.
- Auto generate a JSON
- Totally non-blocking >>> wait for completed (eg. `collectList()`) >>> blocking (eg. `block()`)

**Publisher: Flux and Mono**
```
Mono = 0..1 item
Flux = 0..N item
----------------- They both extends Publisher, emit items on a stream

very similar to completableFuture in Java
```

## Methods
- Wait a Flux to finish, can make it become Mono:
  - `fluxA.collect() / fluxA.collectList() / fluxA.collectMap()`
- Combine two Mono together:
  - `monoA.zipWith(monoB, func) / Mono.zip(func, monoA, monoB)`
- Classify the item of a Flux into different groups:
  - `fluxA.groupBy(func)`
- Convert Mono<List> to Flux:
  - `monoA.flatMapIterable(func)`
- Logging:
  - `monoA.doOnNext(func) / fluxA.doOnComplete(func)`
- Error handling for empty publisher:
  - `defaultIfEmpty() / switchIfEmpty()`

`block()` [Mono]
- Subscribe to this Mono and block indefinitely until a next signal is received.

`blockLast() / blockFirst()` [Flux]
- Subscribe to this Flux and block indefinitely until the upstream signals its last/first value or completes.

`collectList()`
```java
// When Flux completes, transform the items in Flux into a Mono<List>, emitting the empty List if the sequence was empty.
Flux<String> flux = Flux.just(
						"Site_0:bezkoder.com", 
						"Description_0:Java Technology",
						"Description_1:Project Reactor");

// use block() method to subscribe to the Mono and block indefinitely until a next signal is received.
// - Flux<String> -> the same stream return Mono<List<String>> -> block() = List<String>
List<String> list1 = flux.collectList().block();
list1.forEach(System.out::println);

// Result:
Site_0:bezkoder.com
Description_0:Java Technology
Description_1:Project Reactor
```

`collectMap()`
```java
// When Flux completes, transform the items in Flux to a Mono<Map<key, value>>, accept two function parameter for key and value, emitting the empty Map if the sequence was empty.
Flux<String> flux = Flux.just(
						"Site_0:bezkoder.com", 
						"Description_0:Java Technology",
						"Description_1:Project Reactor");

// - Flux<String> -> the same stream return Mono<Map<String, String>>
Map<String, String> map1 = flux
		.collectMap(
				item -> item.split(":")[0], 
				item -> item.split(":")[1])
		.block();  // Mono<Map<String, String>> -> Map<String, String>
map1.forEach((key, value) -> System.out.println(key + " -> " + value));

// Result:
Site_0 -> bezkoder.com
Description_1 -> Project Reactor
Description_0 -> Java Technology
```

`defaultIfEmpty()`
- Default value if the sequence is empty
```java
// - empty? -> the same stream return SkuPriceModel -> return Mono<SkuPriceModel>
Mono<SkuPriceModel> skuPriceChartData = skuPriceDao.findPriceBySkuCodeAndMembership(skuCode, membership)
   .defaultIfEmpty(new SkuPriceModel());
```

`doOnComplete()` [Flux]
public final Flux<T> doOnComplete(Runnable onComplete)
- Add behavior (side-effect) triggered when the Flux completes successfully.


`doOnNext()` [Mono]
public final Mono<T> doOnNext(Consumer<? super T> onNext)
- Add behavior triggered when the Mono emits a data successfully, eg. logging
```java
monthlyReportDataRepo.getMonthlyRecordsData(districtsList, month)
   .collectList()
   .doOnNext(list -> {
      logger.info("Get {} rows of monthly record data", list.size());
   })
```

`doOnError()`

`onErrorMap()`

`flatmap()` [Flux]
public final <R> Flux<R> flatMap(Function<? super T,? extends Publisher<? extends R>> mapper)
![](../image/Snipaste_2022-08-17_14-58-09.png)
- Transform the items by an asynchronous function that return a Publisher (eg. a Flux). All of these publishers will emit items to a final publisher (Flux). 
- Since the operator will not wait for one publisher to finish before moving on to the next stream, it is non-blocking and the order of items are not preserved. If the order is important, use `flatMapSequential()`.
- The number of items after transformation can be different. (ie. one to many)
```java
// Flux<String> --> Other streams return many Flux<String> --> Final stream returns Flux<String>
Flux<String> a;
a.flatmap(s -> Flux.just(s.toUpperCase().split("")));
```

`flatmapIterable()` [Mono/Flux]
```java
// - Mono<List<List<String>>> -> Flux<List<String>>
Mono<List<List<String>>> a;
a.flatMapIterable(list -> list);
```

`fromArray()`
```java
// create a flux that emit the items in the provided array
var str = new String[]{"Hello", "World"};
Flux.fromArray(str);
```

`groupBy()` [Flux]
public final <K> Flux<GroupedFlux<K,T>> groupBy(Function<? super T,? extends K> keyMapper)

- Divide this sequence into dynamically created Flux (or groups) for each unique key, as produced by the provided `keyMapper` Function.
- Remark: `public abstract class GroupedFlux<K, V> extends Flux<V>`
```java
public class ClassModel {
   String class;
   List<StudentModel> students;
}

public class StudentModel {
   String class;
   ...
}
// -----------------------
Flux<StudentModel> fluxA          // Flux<StudentModel>
.groupBy(StudentModel::getClass)  // Flux<GroupedFlux<String, StudentModel>>
.flatMap(groupedFlux -> 
   groupedFlux
   .collectList()                 // Mono<List<StudentModel>>
   .map(list -> new ClassModel(groupedFlux.key(), list))  // Flux<ClassModel>
);
```

`just()` [Mono/Flux]
```java
// Create a Mono that emit an item
Mono.just("Hello World");
// Create a flux that emit two String item
Flux.just("Hello", "World");
```

`map()`
```java
// Transform the items by a synchronous function to it
// One to One

// - Flux<String> -> the same stream return Flux<String>
Flux<String> a;

a.map(r -> return r.toUpperCase());

// Eg: map one model to another model
// - Flux<String> -> the same stream return Flux<Model>
a.map(r -> new Model(r));
```

`subscribe()`
- Ask a consumer to do sth on the mono/flux object, and return `void`

`switchIfEmpty()`
```java
// Switch to an alternative Publisher if the sequence is empty
// - empty? -> another stream return Mono<SkuPriceModel> -> return Mono<SkuPriceModel>
Mono<SkuPriceModel> skuPriceChartData = skuPriceDao.findPriceBySkuCodeAndMembership(skuCode, membership)
   .switchIfEmpty(Mono.just(new SkuPriceModel()));
```

`zipWith()` [Mono]
public final <T2,O> Mono<O> zipWith(Mono<? extends T2> other, BiFunction<? super T,? super T2,? extends O> combinator)

- Combine the result from this Mono and another Mono into an arbitrary `O` object
```java
monoA.zipWith(
   monoB,
   (monoA, monoB) -> {
      // do sth
      return list;  // an arbitrary O object
   }
)
```

`zipWith()` [Flux]
public final <T2,V> Flux<V> zipWith(Publisher<? extends T2> source2, BiFunction<? super T,? super T2,? extends V> combinator)
- Zip this Flux with another Publisher source, that is to say **wait for both to emit one element** and combine these elements using a combinator BiFunction The operator will continue doing so until any of the sources completes.

## Testing - Unit test
```java
import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;

@Test
void test() {
   // check if the sequence is completed after all expectNext()
   StepVerifier
      .create(Flux.just("Hello", "World"))
      .expectNext("Hello", "World")
      .verifyComplete();

   // Same
   StepVerifier
      .create(Flux.just("Hello", "World"))
      .expectNext("Hello").expectNext("World")
      .verifyComplete();
}
```

## Testing - Blockhound
- BlockHound will transparently instrument the JVM classes and intercept blocking calls (e.g. IO) if they are performed from threads marked as "non-blocking operations only" (ie. threads implementing Reactor's `NonBlocking` marker interface, like those started by `Schedulers.parallel()`).
- Turn on blockhound plugin and test the reactive code, ensure no blockhound `BlockingOperationError` exception thrown (which means no blocking logic in your code)
- https://github.com/reactor/BlockHound
- https://domenicosibilio.medium.com/blockhound-detect-blocking-calls-in-reactive-code-before-its-too-late-6472f8ad50c1
```xml
<dependency>
   <groupId>io.projectreactor.tools</groupId>
   <artifactId>blockhound</artifactId>
   <version>1.0.6.RELEASE</version>
</dependency>
```
```java
@SpringBootApplication
public class Application {

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
    BlockHound.install();  // <--- add this line
  }
}

// Or you can add in test.java
@BeforeEach
void setUp() {
    BlockHound.install();  // <--- add this line
}
```
```java
// Example of error shown
Mono.delay(Duration.ofSeconds(1))
    .doOnNext(it -> {
        try {
            Thread.sleep(10);
        }
        catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    })
    .block();
```

## Service Example
Use case: combine data in different documents (rows) into a list, then put the list inside a object to render JSON resposne
```java
Mono<List<PriceTimeModel>> productLowestPriceDataList = productPriceDao
      .findPriceBySkuCodeAndMembership(skuCode, membership)
      .filter(productPriceData -> !(productPriceData.getLowestPrice() < 0
            && productPriceData.getYesterdayLatestPrice() < 0))
      .map(productPriceData -> {
         if (productPriceData.getLowestPrice() < 0)
            return new PriceTimeModel(productPriceData.getYesterdayLatestPrice(), productPriceData.getDate());
         else if (productPriceData.getYesterdayLatestPrice() < 0)
            return new PriceTimeModel(productPriceData.getLowestPrice(), productPriceData.getDate());
         else
            return new PriceTimeModel(Math.min(productPriceData.getLowestPrice(), productPriceData.getYesterdayLatestPrice()), productPriceData.getDate());
      })
      .collectList()
      .map(productLowestPriceList -> {
         Collections.sort(productLowestPriceList);
         return productLowestPriceList;
      });
```
## Controller
Return HTTP status
- Use `.map(ResponseEntity::ok)` to wrap the publisher response (a JSON) inside a `HTTP status 200` response body.
  ```java
  @GetMapping("/priceChart")
  public Mono<ResponseEntity<ProductPriceResponseModel>> getPriceChartData(...) {
     return providePriceChartDataServiceImpl
           .getPriceDataBySkuCodeAndMembership(skuCode, userMembershipLevel)
           .map(priceDateList -> new ProductPriceResponseModel(skuCode, priceDateList))
           // return a HTTP status
           .map(ResponseEntity::ok);  // method reference
           // same as .map(e -> ResponseEntity.ok(e));
           // same as .map(e -> ResponseEntity.ok().body(e));

           // If need to return other HTTP status, especially 4xx with body
           // use .map(e -> ResponseEntity.status(HttpStatus.BAD_REQUEST).body(e));
  }
  ```
## Error Handling
`WebExceptionHandler` is a special type of `bean` to handle exceptions from Webflux. `DefaultErrorWebExceptionHandler` is the default Webflux exception handler (extends `AbstractErrorWebExceptionHandler`).
  ![](../image/1_5txX5nJaZH8L8ZuJiBzF_g.png)

`DefaultErrorAttributes` is the default error format used by Webflux (`implements ErrorAttributes`)
```java
private Map<String, Object> getErrorAttributes(ServerRequest request, boolean includeStackTrace) {
   Map<String, Object> errorAttributes = new LinkedHashMap();
   errorAttributes.put("timestamp", new Date());
   errorAttributes.put("path", request.path());
   Throwable error = this.getError(request);
   MergedAnnotation<ResponseStatus> responseStatusAnnotation = MergedAnnotations.from(error.getClass(), SearchStrategy.TYPE_HIERARCHY).get(Re sponseStatus.class);
   HttpStatus errorStatus = this.determineHttpStatus(error, responseStatusAnnotation);
   errorAttributes.put("status", errorStatus.value());
   errorAttributes.put("error", errorStatus.getReasonPhrase());
   errorAttributes.put("message", this.determineMessage(error, responseStatusAnnotation));
   errorAttributes.put("requestId", request.exchange().getRequest().getId());
   this.handleException(errorAttributes, this.determineException(error), includeStackTrace);
   return errorAttributes;
}
```

Flow of the exception:
1. All exception (`Throwable`) will first go to `AbstractErrorWebExceptionHandler.handle(ServerWebExchange exchange, Throwable throwable)` and is stored as `ErrorAttributes`
2. `ErrorAttributes` will go to `getRoutingFunction(ErrorAttributes errorAttributes)`
3. Finally, The error page will be generated by `renderErrorResponse(ServerRequest request)`

Method to handle error:
- **Functional Level** Use exception provided by Spring: `throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "reason");`. The handler `ResponseStatusExceptionHandler` will return the corresponding error code
  ```json
  {
      "timestamp": "2022-10-05T09:31:06.496+00:00",
      "path": "/ajax/sku/priceChart",
      "status": 400,
      "error": "Bad Request",
      "requestId": "b1d9da70-1"
  }
  ```
- **Functional Level** Use exception created by yourself, `throw new XxxxxException()` will return `Error 500`
  - (SHOULD BE AVOIDED) Can use `server.error.include-message=always` in application.yml to show the reason
- **Global Level** Create a global exception handler that overrides `handle(ServerWebExchange serverWebExchange, Throwable throwable)` and gets `Throwable` directly to customize error JSON response
  ```json
  {
      "errors": [
          {
              "message": "ValidationException: signature expired."
          }
      ]
  }
  ```
  ```java
   @Configuration
   @Order(-2)
   public class GlobalErrorHandler implements ErrorWebExceptionHandler {

      @Override
      public Mono<Void> handle(ServerWebExchange serverWebExchange, Throwable throwable) {

         if(throwable instanceof ValidationException) {
            return ...;
            // The most important thing is how to generate a JSON to return
         }
      }
   }

   public static Mono<Void> returnJson(ServerHttpResponse originalResponse, List<ErrorMsgModel> errorDTO, HttpStatus httpStatus) {

      originalResponse.setStatusCode(httpStatus);

      // this actually is just a List<String>
      ErrorMsgArrayModel errorMsgArrayModel = new ErrorMsgArrayModel();
      errorMsgArrayModel.setErrors(errorDTO);

      byte[] bytes = new byte[0];

      try {
         bytes = new ObjectMapper().writeValueAsString(errorMsgArrayModel).getBytes();
      } catch (Exception e) {
         logger.error("convert json exception!!", e);
      }

      DataBuffer buffer = originalResponse.bufferFactory().wrap(bytes);
      originalResponse.getHeaders().add(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);

      return originalResponse.writeWith(Mono.just(buffer));
   }
  ```
- **Global Level** Create a global exception handler that overrides `getRoutingFunction(ErrorAttributes errorAttributes)` and customize `renderErrorResponse(ServerRequest request)` using the default ErrorAttributes
  - Using the same JSON format as default exception handler
  - `@Order(-2)` give it a higher priority than the DefaultErrorWebExceptionHandler, which is registered at `@Order(-1)`
  ```json
  {
     "timestamp": "2022-10-07T10:13:08.112+00:00",
     "path": "/ajax/sku/priceChart",
     "status": 400,
     "error": "ValidationException: signature not match.",
     "requestId": "c19d46bb-3"
  }
  ```
  ```java
  @Component
  @Order(-2)
  public class GlobalErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {
  
    public GlobalErrorWebExceptionHandler(
        ErrorAttributes errorAttributes,
        ApplicationContext applicationContext,
        ServerCodecConfigurer serverCodecConfigurer) {
      super(errorAttributes, new WebProperties.Resources(), applicationContext);
      super.setMessageWriters(serverCodecConfigurer.getWriters());
      super.setMessageReaders(serverCodecConfigurer.getReaders());
    }
  
    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {
      return RouterFunctions.route(RequestPredicates.all(), this::renderErrorResponse);
    }
  
    protected Mono<ServerResponse> renderErrorResponse(ServerRequest request) {
  
      Map<String, Object> errorAttributes =
          getErrorAttributes(request, ErrorAttributeOptions.defaults());
  
      // get back the raw Exception from ErrorAttributes
      Throwable error = getError(request);
  
      // customize the data in ErrorAttributes using the raw Exception
      if(error instanceof ValidationException){
        errorAttributes.put("error", error.getMessage());
        errorAttributes.put("status", HttpStatus.BAD_REQUEST.value());
      }
  
      // customize the HTTP status and JSON response
      return ServerResponse.status((int) errorAttributes.get("status"))
          .contentType(MediaType.APPLICATION_JSON)
          .body(BodyInserters.fromValue(errorAttributes));
    }
  }
  ```

# SQL Databases

- If there is multiple instances of the application, always need to think of synchronization problem of a query/update.
  - Only need to run once -> use shedlock
  - Need to run on each instance but need async 
- If you want to add a unique index after finding some duplicate records (i.e. not in the phase of setting up DB)
  - unique index build would fail, duplicate key error
  - you need to remove all duplicate records beforehand

## Java Persistence API (JPA)

JPA is a specification that provides a standard way to map Java objects to relational databases using **ORM (Object-Relational Mapping)** techniques. ORM is a programming technique that maps data between object-oriented programming languages and relational databases.

JPA provides several **benefits**, including:
- Database independence: With JPA, you can write code that is independent of the underlying database. This means that you can switch databases without changing your code.
- Object-oriented programming: JPA maps Java objects to database tables, which makes it easier to work with databases in an object-oriented way.
- Simplified data access: JPA provides a simplified way to access and manage data in the database, making it easier for developers to work with databases.

Here are the most popular **implementations** of the Java Persistence API (JPA):
- Hibernate
- EclipseLink
- Apache OpenJPA
- DataNucleus

**Persistence**

When an entity object is first created, it is in the "transient" state, meaning that it is not yet associated with a database record. Once the object is persisted (i.e., saved) using JPA's EntityManager, it enters the "persistent" state, and any changes made to the object will be tracked by the JPA provider and eventually synchronized with the database.

Every Hibernate entity naturally has a lifecycle within the framework – it's either in a transient, managed, detached or deleted state.

The persistent state is important because it allows JPA to manage the lifecycle of entity objects, including caching, transaction management, and synchronization with the database.

## Hibernate (JPA)

![](../image/hibernate-orm-mpping.png)

Advantages of Hibernate Framework: https://www.javatpoint.com/hibernate-tutorial
- Reduced boilerplate code: JPA and Spring Boot provide many features out of the box, such as transaction management and connection pooling, which can reduce the amount of boilerplate code you need to write.

Reference: https://www.baeldung.com/learn-jpa-hibernate


**DB Config (Spring)**

application.yml
```yml
datasource:
  master:
    url: jdbc:mysql://<ip_address>:3306/hktv_payment?useUnicode=true&connectionCollation=utf8mb4_unicode_ci&characterEncoding=utf-8&useSSL=false&allowPublicKeyRetrieval=true
    username: root
    password: HKtv2014
    driver-class-name: com.mysql.cj.jdbc.Driver
    pool-name: payment-master-cp
    maximum-pool-size: 20

hibernate:
  payment:
    packages-to-scan: hk.com.hktvmall.payment.model
    hibernate-properties:
      hibernate.dialect: org.hibernate.dialect.MySQL8Dialect
      hibernate.show_sql: false
      hibernate.hbm2ddl.auto: none
      hibernate.current_session_context_class: org.springframework.orm.hibernate5.SpringSessionContext
```

DatabaseConfig.java
```java
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.Profile;
import org.springframework.orm.hibernate5.HibernateTransactionManager;
import org.springframework.orm.hibernate5.LocalSessionFactoryBean;

import javax.sql.DataSource;

@Profile({ LOCAL, DEV, STAGING, PRODUCTION })
@Configuration
public class DatabaseConfig
{
    @Bean
    @Primary
    @ConfigurationProperties("datasource.master")
    public DataSourceProperties firstDataSourceProperties()
    {
      return new DataSourceProperties();
    }

    @Bean
    @Primary
    @ConfigurationProperties("datasource.master")
    public DataSource firstDataSource()
    {
      return firstDataSourceProperties().initializeDataSourceBuilder().build();
    }

    @Bean(name = "paymentSessionFactory")
    @ConfigurationProperties("hibernate.payment")
    public LocalSessionFactoryBean paymentSessionFactory()
    {
      LocalSessionFactoryBean sessionFactoryBean = new LocalSessionFactoryBean();
      sessionFactoryBean.setDataSource(firstDataSource());
      return sessionFactoryBean;
    }

    @Bean(name = "paymentTransactionManager")
    public HibernateTransactionManager paymentTransactionManager()
    {
      HibernateTransactionManager transactionManager = new HibernateTransactionManager();
      transactionManager.setSessionFactory(paymentSessionFactory().getObject());
      return transactionManager;
    }
}
```

**DB Config (Spring boot)**

Application.yml
```yml
spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=jdbc:mysql://${MYSQL_HOST:localhost}:3306/db_example
spring.datasource.username=springuser
spring.datasource.password=ThePassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#spring.jpa.show-sql: true
```
- Reference: https://spring.io/guides/gs/accessing-data-mysql/

## Spring Data JPA

- https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#data.sql.jpa-and-spring-data
- https://spring.io/guides/gs/accessing-data-jpa

Spring Data is a part of Spring Framework. The goal of Spring Data repository abstraction is to significantly reduce the amount of boilerplate code required to implement data access layers for various persistence stores.

**Spring Data JPA is not a JPA provider.** It is a library/framework that adds an extra layer of abstraction on the top of the JPA provider Hibernate.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

To start with, add the configuration in the `application.yml`.

```yml
spring:
  jpa:
    hibernate:
      ddl-auto: update
  datasource:
    url: jdbc:mysql://192.168.124.109:3306/notification_api_gateway
    username: root
    password: mF34266223ok
    driver-class: com.mysql.cj.jdbc.Driver
```

### Entity Classes

- https://www.baeldung.com/hibernate-notnull-vs-nullable - Explain @Column(nullable = false)

Any classes annotated with `@Entity`, `@Embeddable`, or `@MappedSuperclass` are considered as entity classes. You can create entity per Dao or per query.

Consider a table `notification`, in which `notification_id` is the primary key:

| Name               | Datatype         | Allow Null | Default        |
|--------------------|------------------|------------|----------------|
| `notification_id`  | INT(11) UNSIGNED | NOT NULL   | AUTO_INCREMENT |
| `company_id`       | INT(11) UNSIGNED | NOT NULL   |
| `service_id`       | INT(11) UNSIGNED | NOT NULL   |
| `content`          | TEXT             | NOT NULL   |
| `status`           | INT(11)          | NOT NULL   |
| `create_time`      | DATETIME         | NOT NULL   |
| `last_update_time` | DATETIME         | NOT NULL   |

The entity class that represents `notification` objects:

```java
import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.Data;

import java.time.LocalDateTime;


@Entity
@Table(name = "notification")
@Data
public class NotificationDto
{
	@Id
	@GeneratedValue(strategy = GenerationType.AUTO)
	@Column(name = "notification_id", nullable = false)
	private Integer notificationId;

	@Column(name = "company_id", nullable = false)
	private Integer companyId;

	@Column(name = "service_id", nullable = false)
	private Integer serviceId;

	@Column(name = "content", nullable = false)
	private String content;

	@Column(name = "status", nullable = false)
	private Integer status;

	@Column(name = "create_time", nullable = false)
	private LocalDateTime createTime;

	@Column(name = "last_update_time", nullable = false)
	private LocalDateTime lastUpdateTime;
}
```

This class has one constructor - the default constructor, which exists only for the sake of JPA.

| Annotation        | Variable   | Description                                                                                                                                                                                |
|-------------------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `@Entity`         |            | Annotated as a JPA entity.                                                                                                                                                                 |
| `@Table`          | `name`     | Specify the name of the mapped table if it is different from the class name.                                                                                                               |
| `@Id`             |            | JPA recognizes it as the object’s ID.                                                                                                                                                      |
| `@GeneratedValue` | `strategy` | Indicate that the ID should be generated automatically.                                                                                                                                    |
| `@Column`         | `name`     | Specify the name of the mapped column if it is different from the variable name.                                                                                                           |
| `@Column`         | `nullable` | Used only for schema DDL generation. This means that if we let Hibernate generate the database schema automatically, it applies the not null constraint to the particular database column. |

### ID generation strategy

If you want to customize the ID generation strategy:

```java
public class PaymentInfo 
{
    @Id
    @Column(name = "id", insertable = false, nullable = false)
    @GenericGenerator(name = "id", strategy = "hk.com.hktvmall.payment.utils.generator.PrimaryKeyGenerator")
    @GeneratedValue(generator = "id")
    private Long id;

    // ...
}


// The generic generator
import org.hibernate.id.IdentityGenerator;
import org.hibernate.engine.spi.SharedSessionContractImplementor;
// ...

public class PrimaryKeyGenerator extends IdentityGenerator
{
    @Override
    public Serializable generate(SharedSessionContractImplementor s, Object obj)
    {
      // ...
    }
}
```

### Dao

```java
import org.springframework.data.repository.CrudRepository;

// This will be AUTO IMPLEMENTED by Spring into a Bean called userRepository
// CRUD refers Create, Read, Update, Delete

public interface NotificationDao extends CrudRepository<NotificationDto, Integer>
```

```java
public interface NotificationDao extends ReactiveCrudRepository<NotificationDto, Integer>
```

```java
// JpaRepository extends CrudRepository and provides additional JPA-specific functionality.
public interface NotificationDao extends JpaRepository<NotificationDto, Integer>
```

#### Query

```java
@Query(value = "SELECT * FROM `notification` WHERE `service_id` = :serviceId AND `status` = :status ORDER BY `create_time` ASC ;", nativeQuery = true)
	List<NotificationDto> findByServiceIdAndStatus(@Param("serviceId") Integer serviceId, @Param("status") Integer status);
```

#### Insert

You can called the `save()` method inherited from the `CrudRepository`.

```java
private final NotificationDao notificationDao;

public void save()
{
    var notification = new NotificationDto();

    // Set data for the notification variable ...

    notificationDao.save(notification);
}
```

Note that **if a column cannot be null**, the corresponding variable of the entity object also must not be null, even the column has default value setting. One exception is that the variable is annotated with `@GeneratedValue`.

#### Update

#### Delete



## Dao implementation

```java
import org.hibernate.SessionFactory;

import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Root;

@Component
public class PaymentInfoDaoImpl implements PaymentInfoDao
{
    private final SessionFactory sessionFactory;

    public CitiCobrandCardApplyRecordDaoImpl(@Qualifier("paymentSessionFactory") SessionFactory sessionFactory)
    {
      this.sessionFactory = sessionFactory;
    }

    // ...

    // Criteria API Query
    @Override
    public PaymentInfo getPaymentInfo(Long transactionId)
    {
        CriteriaBuilder criteriaBuilder = sessionFactory.getCurrentSession().getCriteriaBuilder();
        CriteriaQuery<PaymentInfo> criteriaQuery = criteriaBuilder.createQuery(PaymentInfo.class);
        Root<PaymentInfo> root = criteriaQuery.from(PaymentInfo.class);

        criteriaQuery.select(root).where(
            criteriaBuilder.equal(root.get("transactionId"), transactionId)
        );

        return sessionFactory.getCurrentSession().createQuery(criteriaQuery).uniqueResult();
    }

    // NativeQuery
    @Override
    public PartialPaymentInfo getPartialPaymentInfo(Long transactionId)
    {
      return (PartialPaymentInfo) sessionFactory.getCurrentSession()
          .createNativeQuery("SELECT first6, last4, card_ref_num AS cardRefNum"
              + " FROM hktv_payment.payment_info"
              + " WHERE transaction_id = :transactionId LIMIT 1")
          .setParameter("transactionId", transactionId)
          .addScalar("first6", StringType.INSTANCE)
          .addScalar("last4", StringType.INSTANCE)
          .addScalar("cardRefNum", StringType.INSTANCE)
          .setResultTransformer(Transformers.aliasToBean(PartialPaymentInfo.class))
          .uniqueResult();
    }

    @Override
    public void updateUncompletedPaymentInfo(Long transactionId, String code, boolean checked3ds)
    {
      sessionFactory.getCurrentSession().createNativeQuery(
          "UPDATE hktv_payment.payment_info "
              + "SET code = :code , "
              + "checked3ds = :checked3ds "
              + "WHERE transaction_id = :transactionId")
          .setParameter("code", code)
          .setParameter("checked3ds", checked3ds)
          .setParameter("transactionId", transactionId)
          .executeUpdate();
    }

    // Query
    @Override
    public PaymentInfo savePaymentInfo(PaymentInfo paymentInfo)
    {
      sessionFactory.getCurrentSession().save(paymentInfo);
      sessionFactory.getCurrentSession().flush();
      sessionFactory.getCurrentSession().refresh(paymentInfo);
      return paymentInfo;
    }

    @Override
    public void updatePaymentInfo(PaymentInfo paymentInfo)
    {
      sessionFactory.getCurrentSession().update(paymentInfo);
    }
  
    @Override
    public void remove(PaymentInfo paymentInfo)
    {
      sessionFactory.getCurrentSession().delete(paymentInfo);
    }
}
```

## Service implementation

```java
@Service
@AllArgsConstructor
public class PaymentInfoServiceImpl implements PaymentInfoService
{
    private final PaymentInfoDao paymentInfoDao;

    @Override
    @Transactional("paymentTransactionManager")
    public void savePaymentInfo(PaymentInfo paymentInfo)
    {
      paymentInfoDao.savePaymentInfo(paymentInfo);
    }

    @Override
    @Transactional("paymentTransactionManager")
    public PartialPaymentInfo getPartialPaymentInfo(Long transactionId)
    {
      return paymentInfoDao.getPartialPaymentInfo(transactionId);
    }

    // ...
}
```
- `@Transactional("paymentTransactionManager")`

# MongoDB
Dependency
- `spring-boot-starter-data-mongodb-reactive` use `ReactiveMongoRepository` (Webflux)
- `spring-boot-starter-data-mongodb` use `MongoRepository`

Database Connection URI format
- ref: https://www.mongodb.com/docs/v5.0/reference/connection-string/
- `mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]]`
  - point to multiple DBs for K8S replica setting (primary/secondary DB)
- `mongodb://127.0.0.1:27018/?directConnection=true`
  - direct connect to a mongodb even it is in a replica set
```yml
# Application.yml
spring:
   data:
      mongodb:
         database: price
         uri: mongodb://admin:Passw0rd@localhost:27017,localhost:27018/price?replicaSet=price
```

Entity class
  ```json
  // example JSON
  {
     "_id": {
        "$oid": "62ecbd983260e9f7de81f129"  // if need, map var with @Id
     },
     "skuCode": "H8224001_S_HKTV_C1",
     "membership": "NORMAL",
     "priceHistory": [
        {
        "price": 144.25,                         // double
        "time": "2022-07-28T00:00:00.000+08:00"  // string
        },
     ]
  }
  ```
  - id / `@Id` fits the standard name for a MongoDB ID
  - `Other unannotated properties` are assumed that they are mapped to fields that share the `same name` as the properties themselves.
  - `@Indexed` can set unique, TTL
  - `@CompoundIndexes` can set unique
  ```java
  /* POJO for query */
  import org.springframework.data.annotation.Id;
  import org.springframework.data.mongodb.core.mapping.Document;

  @Document  // Required, mapping to mongoDB collection
  // If no collection is specified, by default use class name as collection name
  public class SkuPrice {
 
      @Id
      private String id;

		@Indexed
      private String skuCode;

		@Indexed  // tells the mapping framework to call ensureIndex on that property of your document, making searches faster, but it will auto create the index if it does not exist (possible to crash index created manually)
      private String membership;

		// use Instant to store field that is type: Date 
     	private Instant date;

		// change to custom name, use a customized class if nested JSON exists
      @Field("priceHistory")
      private List<PriceTimeModel> data;
  }

  @Data
  public class PriceTimeModel {
      double price;
      String time;
  }

  /* POJO for insert/update */
  @Document
  @CompoundIndexes({
    @CompoundIndex(
        name = "skucCode_membership_date",
        def = "{'skuCode': 1, 'membership': 1, 'date': 1}",
        unique = true),
    @CompoundIndex(
        name = "skuCode_membership", 
        def = "{'skuCode': 1, 'membership': 1}")
  })
  @AllArgsConstructor
  @Getter
  @Setter
  public class ProductPrice
  {
     @Indexed
     private String skuCode;
     private String membership;
     @Indexed(expireAfterSeconds = 31536000)
     private Instant date;
     private Double lowestPrice;
     private Long yesterdayLatestSyncTime;
     private Double yesterdayLatestPrice;
  }
  ```

Query method
- **Query** by JSON based annotation `@Query` (only basic query function, extends interface `ReactiveMongoRepository`)  
https://javatechonline.com/spring-boot-mongodb-query-examples/#Step4_Create_a_Repository_Interface (`@Query` example)  
  ```java
  import org.springframework.data.mongodb.repository.ReactiveMongoRepository;
  import org.springframework.data.mongodb.repository.Query;
  import org.bson.Document;
 
  public interface SkuPriceRepository extends ReactiveMongoRepository<SkuPrice, Integer> {
     
     // ?0 is the first param, ?1 is the second, etc
     @Query(value = "{skuCode: ?0, membership: ?1}", fields="{?1: 1}")  // select membership ... where skuCode = ...
     Flux<SkuPrice> findPriceBySkuCodeAndMembership(String skuCode, String membership);;
     
  }
  ```
- **Query** by method name (extends interface `ReactiveMongoRepository`)  
  https://docs.spring.io/spring-data/data-document/docs/current/reference/html/#mongodb.repositories.queries
  ```java
  Flux<SkuPrice> findPriceBySkuIdAndMembership(String skuId, String membership);
  // auto query by skuId and membership, as it has find keyword in the method name
  ```

- **Query by Java springframework MongoTemplate**  
  https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#repositories.core-concepts
  ```java
  import org.springframework.data.mongodb.core.ReactiveMongoTemplate;

  @Repository
  public class ApiStatisticDao {
    @Autowired private ReactiveMongoTemplate reactiveMongoTemplate;
  
    // update xxx set xxx = xxx where xxx = xxx
    // note that: upsert means either insert (if no record exist) or update. Since it 
    // is possible that multiple instances think they need to do insert, to avoid 
    // duplicate insert, you should use upsert with unique index on the query field
    public Mono<ApiStatistic> insertOrIncrementApiStatistic(
        Instant dateTime, String endpoint, int apiCount) {
      
      Query getPriceChartApiQuery =
          new Query(Criteria.where("endpoint").is(endpoint).and("dateTime").is(dateTime));
      Update getPriceChartApiUpdate = new Update().inc("count", apiCount);
      return reactiveMongoTemplate
          .update(ApiStatistic.class)
          .matching(getPriceChartApiQuery)
          .apply(getPriceChartApiUpdate)
          .withOptions(FindAndModifyOptions.options().upsert(true).returnNew(true))
          .findAndModify();
    }
    
    // select count(*) from xxx where xxx = xxx
    public Mono<Long> countDuplicate(Instant dateTime, String endpoint) {
      Query query = new Query(Criteria.where("endpoint").is(endpoint).and("dateTime").is(dateTime));
      return reactiveMongoTemplate.count(query, ApiStatistic.class);
    }

    // delete from xxx where xxx = xxx
    // note that: must use .subscribe() to handle the Mono<DeleteResult> to complete the delete flow
    public Mono<DeleteResult> delete(Instant dateTime, String endpoint) {
      Query query = new Query(Criteria.where("endpoint").is(endpoint).and("dateTime").is(dateTime));
      return reactiveMongoTemplate.remove(query, ApiStatistic.class);
    }


  }
  ```

- **Query by Java native driver**  
  Java driver docs for MongoDB: https://www.mongodb.com/docs/drivers/java/sync/current/

# SSL
https://www.thomasvitale.com/https-spring-boot-ssl-certificate/

```bash
# Generate an SSL certificate in a keystore file (found in working dir)
$ keytool -genkeypair -alias springboot -keyalg RSA -keysize 4096 -storetype PKCS12 -keystore springboot.p12 -validity 3650 -storepass password

# Verify the keystore content
$ keytool -list -v -keystore springboot.p12

# In application.properties
server.port=8443
server.ssl.key-store-type=PKCS12
server.ssl.key-store=src/main/resources/springboot.p12
server.ssl.key-store-password=password
server.ssl.key-alias=springboot
server.ssl.enabled=true

server:
	ssl:
		enabled: true
		key-store: src/main/resources/springboot.p12
		key-store-password: password
		key-store-type: pkcs12
		key-alias: springboot

# Until now, self-signed cert can be used.
# In advance, better install cert in client side.
```

# MD5 hash of the parameters for Signature
- Prevent URL tampering
```java
// Controller
@Autowired
ValidationUtils validationUtils;

@Autowired
ApiSaltConfig apiSaltConfig;

public Mono<ResponseEntity<SkuPriceResponseModel>> getPriceChartData(
      @RequestParam(value = "skuCode", required = true, defaultValue = "") final String skuCode,
      @RequestParam(value = "lang", required = false) final String lang,
      @RequestParam(value = "userMembershipLevel", required = true, defaultValue = "") final String userMembershipLevel,
      @RequestParam(value = "s", required = true, defaultValue = "") final String signature) {

   if(!validationUtils.isSignatureMatch(skuCode.concat(userMembershipLevel), signature, apiSaltConfig.getApiSalt())) 
      throw new ValidationException("ValidationException: signature not match.");
}

// ApiSaltConfig
@Component
public class ApiSaltConfig {
    
    @Value("${priceChart.secret.apiSalt}")  // stored in application.properties
    private String apiSalt;

    public String getApiSalt() {
        return apiSalt;
    }
}

// ValidationUtils
import org.springframework.util.DigestUtils;

public boolean isSignatureMatch(String messageSource, String signature, String apiSalt) {
   final String hash = _getMD5Hash(messageSource.concat(apiSalt));
   
   return signature.contentEquals(hash);
}

private String _getMD5Hash(String message) {
   return DigestUtils.md5DigestAsHex(message.getBytes(Charset.forName("UTF-8")));
}
```

# Unit Test
## Service
```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;

class ProvidePriceChartDataServiceTest {

   AutoCloseable openMocks;  // init mockito

   // create an instance and injects the mocks which are created with 
   // the @Mock (or @Spy) annotations into this instance.
   @InjectMocks private ProvidePriceChartDataService providePriceChartDataService;

   // create a mock instance to be injected
   // it can mock the result from DB
   @Mock private ProductPriceDao productPriceDao;

   @BeforeEach
   void setUp() {
      openMocks = MockitoAnnotations.openMocks(this);  // init mockito

      BlockHound.install();  // check for web flux project

      // Mock the result as if calling productPriceDao.findPriceBySkuCodeAndMembership()
      Mockito.when(productPriceDao.findPriceBySkuCodeAndMembership("H8224001_S_HKTV_C1", "VIP"))
         .thenReturn(Flux.empty());
   }

   @Test
   void test() {
      StepVerifier
         .create(
            providePriceChartDataService.getPriceDataBySkuIdAndMembership("H8224001_S_HKTV_C1", "VIP"))
         .expectNext(Collections.emptyList())
         .verifyComplete();
   }
}

// Refer to the service
@Service
public class ProvidePriceChartDataService {

   @Autowired private ProductPriceDao productPriceDao;

   public Mono<List<PriceDateModel>> getPriceDataBySkuIdAndMembership(String skuCode, String membership) {
      ...
      Mono<List<PriceDateModel>> productLowestPriceDataList = productPriceDao // mock instance
            .findPriceBySkuCodeAndMembership(skuCode, membership) // mock function
            ...
   }
}
```
## Webflux Controller
```java
import org.springframework.boot.test.autoconfigure.web.reactive.WebFluxTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.context.annotation.Import;
import org.springframework.test.web.reactive.server.WebTestClient;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;

// Since we are using WebTestClient to test, need to create a controller variable using @WebFluxTest
// For beans that do not need to mock result/ serivce beans, use @import (similar to @InjectMocks)
// For beans that need to mock result/ need to be injected, use @MockBean to create a mock instance (similar to @Mock)
@WebFluxTest(controllers = PriceChartController.class)
@Import({ValidationUtils.class})
class PriceChartControllerTest {

  AutoCloseable openMocks;

  @MockBean private ApiSaltConfig apiSaltConfig;

  @MockBean private ProvidePriceChartDataService providePriceChartDataService;

  @Autowired WebTestClient webTestClient;

  @BeforeEach
  void setUp() {
    BlockHound.install();

    openMocks = MockitoAnnotations.openMocks(this);

    // Mock the result as if calling providePriceChartDataService.getPriceDataBySkuIdAndMembership()
    var priceDateList = new ArrayList<PriceDateModel>();
    priceDateList.add(new PriceDateModel(174.25, "2022-08-22T00:00:00.000+08:00"));
 
    // Mock result for existing product
    Mockito.when(
            providePriceChartDataService.getPriceDataBySkuIdAndMembership(
                "H8224001_S_HKTV_C1", "NORMAL"))
        .thenReturn(Mono.just(priceDateList));

    // Mock the result as if calling apiSaltConfig.getApiSalt()
    Mockito.when(apiSaltConfig.getApiSalt()).thenReturn("testApiSalt");
  }

  @Test
  void testGetPriceChartDataWithCorrectSignature() {
    String signature = MD5Generation._getMD5Hash("H8224001_S_HKTV_C1NORMALtestApiSalt");

    webTestClient
        .get()  // can change to post() or delete()
        .uri(
            uriBuilder ->
                uriBuilder
                    .path("/ajax/sku/priceChart")  // path
                    .queryParam("skuCode", "H8224001_S_HKTV_C1")  // param...
                    .queryParam("user", "NORMAL")
                    .queryParam("s", signature)
                    .build())
        .exchange()
        .expectStatus().isOk()
        // ----------
        // expect a customized JSON response, used jsonPath
        // For example:
        //
        // {
        //     "skuCode": "H8224001_S_HKTV_C1",
        //     "data": [
        //         {
        //             "price": 174.25,
        //             "date": "2022-07-28T00:00:00.000+08:00"
        //         }
        //     ]
        // }
        //
        .expectBody()
        .jsonPath("$.skuCode").isEqualTo("H8224001_S_HKTV_C1")
        .jsonPath("$.data[0].price").isEqualTo(174.25)
        .jsonPath("$.data[0].date").isEqualTo("2022-07-28T00:00:00.000+08:00");
        // --- OR ---
        // expect a JSON response directly ref to a model
        .expectBodyList(ProductPriceResponseModel.class);
  }
}

// Refer to the Controller
@RestController
public class PriceChartController {
  @Autowired ProvidePriceChartDataService providePriceChartDataService;

  @Autowired ValidationUtils validationUtils;

  @Autowired ApiSaltConfig apiSaltConfig;

    @GetMapping("/ajax/sku/priceChart")
  public Mono<ResponseEntity<ProductPriceResponseModel>> getPriceChartData(
      @RequestParam(value = "skuCode", required = true, defaultValue = "") final String skuCode,
      @RequestParam(value = "user", required = true, defaultValue = "NORMAL") final String user,
      @RequestParam(value = "s", required = true, defaultValue = "") final String signature) {

    if (!validationUtils.isSignatureMatch(..., apiSaltConfig.getApiSalt()))
    // calling apiSaltConfig.getApiSalt()

    return providePriceChartDataService
          .getPriceDataBySkuIdAndMembership(skuCode, userMembershipLevel)
          .map(priceDateList -> new ProductPriceResponseModel(skuCode, priceDateList))
          .map(ResponseEntity::ok);
  }
}
```
# Load Test (JMeter)
- JMeter function doc: https://jmeter.apache.org/usermanual/functions.html#top
- Use data from csv for each user's loop:
![](../image/Snipaste_2022-09-05_17-55-44.png)
- JMeter best practice doc: https://jmeter.apache.org/usermanual/best-practices.html
  - Don't use "View Results Tree" or "View Results in Table" listeners during the load test, use them only during scripting phase to debug your scripts.
  - use `jmeter -n -t test.jmx` instead of GUI to run test, add `-l result.jtl` for output listener data to file
```java
// How to set path parameter

// random from 0,1 ... 9,10
${__Random(0,10)}

// starting from 1,2,3,4 ... , FALSE = global counter used for all users
${__counter(FASLE)}

// using csv config
${__V(skuCode)}

// Time
${__time(hh:mm:ss dd/MM/yyyy)}
```
# Cache
## Caffeine
- Related link:
  - Java: https://www.baeldung.com/java-caching-caffeine
  - Spring boot: https://www.baeldung.com/spring-boot-caffeine-cache
  - How to unit test caching: https://nickolasfisher.com/blog/How-to-use-Caffeine-Caches-Effectively-in-Spring-Boot-Webflux
  - [Doc] Caffine: https://www.javadoc.io/doc/com.github.ben-manes.caffeine/caffeine/latest/com.github.benmanes.caffeine/com/github/benmanes/caffeine/cache/Caffeine.html
  - [Doc] SimpleCacheManager: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/support/SimpleCacheManager.html
  - [Doc] CaffeineCache: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/caffeine/CaffeineCache.html
- Remark for `maximum size`: In practice, make larger maximum size initialy, then adjust the size accordign to the cache miss rate. 
- **Remark for Webflux:** After adding `@Cacheable` on the method, the result of the method is cached as Mono/Flux Wrapper object. However, the DB result is just emitted once in the Publisher. Therefore, caching the Publisher does not equal to caching the DB result, and the cached Publisher still needs to fetch a new record from the database. We need to use the built-in cache for Mono/Flux `cache()` to cache the actual result inside the Publisher. Therefore, we are using a cache of a cache. We need to set appropriate expiry times on both caches. The rule of thumb is that Flux cache TTL should be longer than `@Cacheable`. 
  - **In short, set `@Cacheable` then set return `.cache()`**
  - https://www.baeldung.com/spring-webflux-cacheable

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		<dependency>
			<groupId>com.github.ben-manes.caffeine</groupId>
			<artifactId>caffeine</artifactId>
		</dependency>
```

```java
// Method to be cached
// @Cacheable help doing CacheManager.get() and CacheManager.put()
// @CacheEvict help doing CacheManager.evict()

@Service
public class ProvideDataServiceImpl {

   // By default, all method parameters are used as cache key.
   // If want to specify key:
   // 1. key = "{#skuCode, #membership}" or key = "#skuCode"
   // 2. key = "#skuCode.concat(#membership)"
   @Cacheable(value = "priceChart", key = "{#skuCode, #membership}")
   public Mono<List<String>> getData(String skuCode, String membership) {

         Mono<List<String>> list;	
         ...
         return list.cache(Duration.ofSeconds(60));
   }

   // Clear cache if present
   @CacheEvict(value = "priceChart", key = "{#skuCode, #membership}")
   public void evictSingleCacheValue(String skuCode, String membership) {}
}
```

```java
/* 
After adding @EnableCaching, Spring boot is finding a bean of CacheManager during initialization.

1. Define Caffeine spec and put it inside CaffeineCache
2. Put list of CaffeineCache inside SimpleCacheManager
3. Return a bean of SimpleCacheManager (implements CacheManager)

For testing, you can set maximumSize(0) or expireAfterWrite(Duration.ofSeconds(0))
*/

@Configuration
@EnableCaching
public class CaffeineCacheConfig
{
  @Bean
  @Primary
  public CacheManager caffeineCacheManager() {
    SimpleCacheManager cacheManager = new SimpleCacheManager();
    ArrayList<CaffeineCache> caches = new ArrayList<>();
    caches.add(
        new CaffeineCache(
            "priceChart",
            Caffeine.newBuilder()
               .maximumSize(20000)
               .expireAfterWrite(Duration.ofHours(12))
               .recordStats()  // this is for actuator
               .build()));
    cacheManager.setCaches(caches);
    return cacheManager;
  }
}
```

```yml
# If you want to enable trace log (cache computation information), add in application.yml:
logging:
  level:
    org:
      springframework:
        cache: TRACE
```
```log
Cache computation information:
----------- (Not yet cached) -----------
2022-09-06 11:23:22.949 TRACE 68148 --- [     parallel-2] o.s.cache.interceptor.CacheInterceptor   : Computed cache key '[H8224001_S_HKTV_C1, NORMAL]' for operation Builder[public reactor.core.publisher.Mono com.hktvmall.hktvpricechartapi.service.impl.ProvidePriceChartDataServiceImpl.getPriceDataBySkuIdAndMembership(java.lang.String,java.lang.String)] caches=[priceChart] | key='{#skuCode, #membership}' | keyGenerator='' | cacheManager='' | cacheResolver='' | condition='' | unless='' | sync='false'

2022-09-06 11:23:22.950 TRACE 68148 --- [     parallel-2] o.s.cache.interceptor.CacheInterceptor   : No cache entry for key '[H8224001_S_HKTV_C1, NORMAL]' in cache(s) [priceChart]

2022-09-06 11:23:22.950 TRACE 68148 --- [     parallel-2] o.s.cache.interceptor.CacheInterceptor   : Computed cache key '[H8224001_S_HKTV_C1, NORMAL]' for operation Builder[public reactor.core.publisher.Mono com.hktvmall.hktvpricechartapi.service.impl.ProvidePriceChartDataServiceImpl.getPriceDataBySkuIdAndMembership(java.lang.String,java.lang.String)] caches=[priceChart] | key='{#skuCode, #membership}' | keyGenerator='' | cacheManager='' | cacheResolver='' | condition='' | unless='' | sync='false'

----------- (Cached) -----------
2022-09-06 11:25:12.534 TRACE 68148 --- [     parallel-4] o.s.cache.interceptor.CacheInterceptor   : Computed cache key '[H8224001_S_HKTV_C1, NORMAL]' for operation Builder[public reactor.core.publisher.Mono com.hktvmall.hktvpricechartapi.service.impl.ProvidePriceChartDataServiceImpl.getPriceDataBySkuIdAndMembership(java.lang.String,java.lang.String)] caches=[priceChart] | key='{#skuCode, #membership}' | keyGenerator='' | cacheManager='' | cacheResolver='' | condition='' | unless='' | sync='false'

2022-09-06 11:25:12.534 TRACE 68148 --- [     parallel-4] o.s.cache.interceptor.CacheInterceptor   : Cache entry for key '[H8224001_S_HKTV_C1, NORMAL]' found in cache 'priceChart'
```
# Actuator
- The main benefit of this library is that we can get production-grade tools without having to actually implement these features ourselves.
- Related link:
  - all endpoints doc: https://docs.spring.io/spring-boot/docs/current/actuator-api/htmlsingle/#overview
  - actuator tutorial: https://www.baeldung.com/spring-boot-actuators
  - /metrics doc: https://docs.spring.io/spring-boot/docs/2.0.0.M3/reference/html/production-ready-metrics.html
- **Actuator comes with most endpoints disabled.** Thus, the only two available by default are /health and /info.
- By default, all Actuator endpoints are now placed under the `/actuator` path. We can tweak this path using the new property management.endpoints.web.base-path.

## Set metrics to check cache
1. Set application.yml
```yml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: metrics,health # set * to show all endpoints
  endpoint:
    health:
      show-details: ALWAYS
      group:
        liveness:
          include: ping,mongo
        readiness:
          include: ping,mongo
```
2. Go to `/actuator/metrics`. Should be able to see `cache.gets`
```json
{
    "names": [
        "cache.eviction.weight",
        "cache.evictions",
        "cache.gets",
        "cache.puts",
        "cache.size",
         ...
    ]
}
```
3. Go to `/actuator/metrics/cache.gets`.
```json
{
    "name": "cache.gets",
    "description": "the number of times cache lookup methods have returned an uncached (newly loaded) value, or null",
    "baseUnit": null,
    "measurements": [
        {
            "statistic": "COUNT",
            "value": 3.0
        }
    ],
    "availableTags": [
        {
            "tag": "result",
            "values": [
                "hit",
                "miss"
            ]
        },
        {
            "tag": "cache.manager",
            "values": [
                "caffeine"
            ]
        },
        {
            "tag": "cache",
            "values": [
                "priceChart"
            ]
        },
        {
            "tag": "name",
            "values": [
                "priceChart"
            ]
        }
    ]
}
```
4. Use result tag to check hit and miss. Go to `/actuator/metrics/cache.gets?tag=result:hit&tag=name:priceChart`

# Rate limiter
- https://vikasverma.tech/post/ratelimiter-with-resilience4j-spring-boot2/ 
- https://reflectoring.io/rate-limiting-with-springboot-resilience4j/(example for spring boot)
- Rate limiting is technique to help to limit the number of requests or type of request received by a server.
  - Server-side Rate Limiting
  - Client-side Rate Limiting
    - resilience4j-ratelimiter

## Resilience4j
A thread that wants to call a remote service first asks the RateLimiter for permission. If the RateLimiter permits it, the thread proceeds. Otherwise, the RateLimiter parks the thread or puts it in a waiting state.

The RateLimiter creates new permissions periodically. When a permission becomes available, the thread is notified and it can then continue.

The number of calls that are permitted during a period is called `limitForPeriod`. How often the RateLimiter refreshes the permissions is specified by `limitRefreshPeriod`. How long a thread can wait to acquire permission is specified by `timeoutDuration`. If no permission is available at the end of the wait time, the RateLimiter throws a `RequestNotPermitted` runtime exception.

Remark: Multiple Rate Limits need more than one beans.

```xml
		<!--Libraries for rate limiter-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>
		<dependency>
			<groupId>io.github.resilience4j</groupId>
			<artifactId>resilience4j-spring-boot2</artifactId>
			<version>1.7.1</version>
		</dependency>
```

```yml
# application.yml
resilience4j:
  ratelimiter:
    instances:
      service:
        limitForPeriod: 20000
        limitRefreshPeriod: 1s
        timeoutDuration: 0  # default is 5 sec
```

```java
@RateLimiter(name = "service", fallbackMethod = "serviceFallback")
public Mono<List<PriceDateModel>> service(
      String skuCode, String membership) { }

// in fallback method, must accept a Throwable class parameter
public Mono<List<PriceDateModel>> serviceFallback(RequestNotPermitted e) {
   return Mono.just(Collections.emptyList());
}
```
Default response if exceed rate limit:
```json
{
    "timestamp": "2022-10-05T03:29:45.924+00:00",
    "path": "/ajax/sku/priceChart",
    "status": 500,
    "error": "Internal Server Error",
    "requestId": "0906106e-3"
}
```

# Scheduler Cron Job

- For cron expression, please refer to notes `common.md`.

## ShedLock
ShedLock makes sure that your scheduled tasks are executed **at most once** at the same time. That is, to avoid multiple instances of our application run the same cron job at the same time, causing some synchronization problems, eg. inserting duplicated record

Please note, that if one task is already being executed on one node, execution on other nodes **does not wait**, it is simply skipped.
- `lockAtMostFor` attribute specifies how long the lock should be kept in case the executing node dies, which should be much longer than normal execution time.
- `lockAtLeastFor` attribute specifies minimum amount of time for which the lock should be kept. Its main purpose is to prevent execution from multiple nodes in case of really short tasks and clock difference between the nodes.

Different type of DB uses different libraries:
- ref: https://github.com/lukas-krecan/ShedLock
- MongoDB uses `<artifactId>shedlock-provider-mongo</artifactId>`
- MySQL uses `<artifactId>shedlock-provider-jdbc-template</artifactId>`

Duration Spec
- All the annotations where you need to specify a duration support the following formats
  - duration+unit - 1s, 5ms, 5m, 1d (Since 4.0.0)
  - duration in ms - 100 (only Spring integration)
  - ISO-8601 - PT15M (see Duration.parse() documentation)

pom.xml
```xml
<!-- MongoDB (not reactive) -->
<dependency>
   <groupId>net.javacrumbs.shedlock</groupId>
   <artifactId>shedlock-spring</artifactId>
   <version>4.42.0</version>
</dependency>
<dependency>
   <groupId>net.javacrumbs.shedlock</groupId>
   <artifactId>shedlock-provider-mongo</artifactId>
   <version>4.42.0</version>
</dependency>
```

Java code
```java
// Main Application
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling  // Need to add this for cron job
@EnableSchedulerLock(defaultLockAtMostFor = "1m")  // Need to add this for shedLock
public class MainApplication {
  public static void main(String[] args) {SpringApplication.run(MainApplication.class, args); }
}

// Service Layer
import org.springframework.scheduling.annotation.Scheduled;

@Scheduled(cron = "0 0/1 * * * *")  // every minute
@SchedulerLock(name = "saveApiStatistic", lockAtLeastFor = "10s", lockAtMostFor = "50s")
public void saveApiStatistic() {}

// ShedLock Config
@Configuration
public class ShedLockConfig {

  // Not reactive MongoDb
  @Bean
  public LockProvider lockProvider(MongoClient mongo) {
    return new MongoLockProvider(mongo.getDatabase("database_name"));
  }
}
```

DB storage
```
_id:"saveApiStatistic"
lockUntil: 2022-10-12T06:05:00.002+00:00
lockedAt: 2022-10-12T05:51:00.003+00:00
lockedBy: "PC1GYSXY"
```

# Security
```java
// Block other paths apart from dedicated services
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {
    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) throws Exception{
        return http.authorizeExchange(
                exchangeSpec ->
                        exchangeSpec
                                .pathMatchers(HttpMethod.GET, "/ajax/sku/priceChart").permitAll()
                                .pathMatchers(HttpMethod.GET, "/internal/getPriceChartApiStatus").permitAll()
                                .pathMatchers(HttpMethod.GET, "/actuator/**").permitAll()
                                .anyExchange()
                                .authenticated())
                .exceptionHandling()
                .authenticationEntryPoint(new HttpStatusServerEntryPoint(HttpStatus.UNAUTHORIZED))
                .and()
                .csrf(csrfSpec -> csrfSpec.disable())
                .httpBasic(httpBasicSpec -> httpBasicSpec.disable())
                .formLogin(formLoginSpec -> formLoginSpec.disable())
                .logout(logoutSpec -> logoutSpec.disable())
                .build();
    }
}
```

# CORS
```java
@RestController
@RequestMapping("/web")
@CrossOrigin(originPatterns = "${third-party.allow-origin}")
public class xxxController {

}
```
```yml
# application.yml
third-party:
  allow-origin: https://www.hktvmall.com

  # if want to allow multiple domain by a pattern, use originPatterns instead of origins
  allow-origin: https://www*.hktvmall.com
```
Testing
- Assume requesting resources from `http://localhost:8080`, go to the origin host that want to test (e.g. `https://www.aia.com.hk`), open F12 and use `fetch("http://localhost:8080")`. If no error 403, then it is not blocked by CORS policy.
  - In response headers, it should have `Access-Control-Allow-Origin: https://www.aia.com.hk`

# Useful website/tutorial
- Spring tutorial
  - [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
  - [Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)
  - [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
- Spring concept
  - [top string annotations](https://medium.com/javarevisited/top-spring-annotations-4f691babe458)
  - [spring beans](https://medium.com/javarevisited/spring-beans-in-depth-a6d8b31db8a1)

# Other related concepts
# Spring
- You can build the java and execute the jar in a tomcat web server. (e.g. intelliJ smart tomcat)

# Appendix

## What is Dependency Injection

- Example ref: https://stackoverflow.com/questions/3058/what-is-inversion-of-control

The *Dependency-Injection* (DI) pattern is a more specific version of *Inversion of Control* (IoC).

It is a process whereby objects define their dependencies (that is, the other objects with which they work) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method.

For example, say your application has a text-editor component, and you want to provide spell checking. Your standard code would look something like this:

```java
public class TextEditor {

    private SpellChecker checker;

    public TextEditor() {
        this.checker = new SpellChecker();
    }
}
```

What we've done here creates a dependency between the `TextEditor` and the `SpellChecker`. We are instantiating `SpellChecker` inside `TextEditor`, which means the `TextEditor` class directly depends on the `SpellChecker` class.

The `main` method can only control `TextEditor`, and let `TextEditor` control `SpellChecker`.

```java
public static void main (string[] args) {

  TextEditor textEditor = new TextEditor();
}
```

In an *DI* scenario we would instead do something like this:

```java
public class TextEditor {

    private SpellChecker checker;

    public TextEditor(SpellChecker checker) {
        this.checker = checker;
    }
}
```

Then, the `main` method has control on both `SpellChecker` and `TextEditor`.

```java
public static void main (string[] args) {

  SpellChecker spellChecker = new SpellChecker();

  TextEditor textEditor = new TextEditor(spellChecker);
}
```

We are creating an abstraction by having the `SpellChecker` dependency class in `TextEditor`'s **constructor** signature (not initializing dependency in class). This allows us to create the dependency, then pass it to the `TextEditor` class.
