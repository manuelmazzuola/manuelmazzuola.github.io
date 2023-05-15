---
layout: post
show_badges: false
title: My Spring Professional 2021 certification study notes
description: My Spring Professional 2021 certification study notes to keep track of everything I have learned
summary:  My Spring Professional 2021 certification study notes to keep track everything I have learned
tags: [spring, certification, java]
---

The company I work for has a department called _The School_ which allows employees
to sign up for internal courses and obtain certifications if necessary.  

The courses list is extensive, and this year I have chosen to take
the _Spring Professional_ training course to consolidate my knowledge of the framework.  
The course allowed receiving a voucher to attend the `VMware Spring Professional` exam certification,
so I decided to give it a try.  

**I don't really believe in certifications**, and I even tend to
consider them a contraindication.  
As expected the exam was full of nitty-gritty detail questions on _Spring_ internals
and stuff like that, **But**, I must admit that I've learned a lot while I was preparing for the exam.

Now I am aware of how `Spring` and `Spring Boot` work under the hood, and I better understand
`Spring` features that I don't use daily at work. I am satisfied with the choice.

# Study material

I spent 30 hours to prepare the exam, ~1 hour per weekday.  
The most useful resource has been the official
[VMware Spring study guide](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/certification/vmw-spring-professional-certification-study-guide.pdf).  
It contains a list of questions that lead you through the topics addressed by the exam.
Below the answers I wrote while studying.

# Container, Dependency and IOC

## What is dependency injection and what are the advantages of using it?

The _dependency injection_ is the linking action between dependents and dependencies.
The dependent component is not allowed to inject dependencies by itself.
The DI is delegated to an external authority called _Spring IoC container_ to achieve decoupling between components.
This behaviour is commonly called _Inversion of Control_.
The injection process happens at runtime, allowing flexibility because the application behaviour
can be modified by an external configuration.

Other advantages are:
- Loosely coupled components
- High cohesion
- Code is more readable
- Code is cleaner
- Easy testing by mocking dependencies

## What is an interface and what are the advantages?

A java interface specifies behaviour that the implementing class must implement
and separates what the caller expects from the implementation.
- Interfaces allow an object to be referenced by the methods they support without considering
the implementation.
- Interfaces allow multi-inheritance of types, which is the ability of a class to implement
more than one interface.
- Interfaces can help the testing process, by creating implementations mocks

## What is an ApplicationContext?

An application context is an instance of any class that implements the
[ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html)
interface.
- It provides configuration for the Spring application.
- It manages all the beans, created and initialized by the _Spring IoC Container_.


## How are you going to create a new instance of an ApplicationContext?

To create a new instance of an application context you have to instantiate
a class that implements the `ApplicationContext` interface, for example
the `AnnotationConfigApplicationContext` class.  
__Spring boot__ expose a convenient method `SpringApplication.run`
to bootstrap the application by creating an `ApplicationContext` instance.


## Can you describe the lifecycle of a Spring Bean in an ApplicationContext?

<p></p>
1. Constructor is called
   - If the bean is created using the constructor DI the 1st and 2nd steps are a single step
2. Dependencies are injected
3. `@PostConstruct` (jsr-250) annoted method is called, it must be void with no arguments
4. Call method `setBeanName(name)` declared in the _BeanNameAware_ interface
1. Call method `postProcessBeforeInitialization(bean, beanName)` declared in the _BeanPostProcess_ interface
5. Call method `afterPropertiesSet` declared in the _InitializingBean_ interface
1. Call the init method defined in `@Bean.initMethod`
2. Call method `postProcessAfterInitialization(bean, beanName)` declared in the _BeanPostProcess_ interface
8. Call method `postProcessBeforeDestruction(bean, beanName)` declared _DestructionAwareBeanPostProcessors_ interface
9. `@PreDestroy` (jsr-250) annoted method is called
10. Call method `destroy()` declared in the _DisposableBean_ interface
11. Call destroy method defined in `@Bean.destroyMethod`

## How are you going to create an ApplicationContext in an integration test?

1. Annotate the test class with `@RunWith(SpringJUnit4ClassRunner.class)` if using JUnit4,
or with `@ExtendWith(SpringExtension.class)` if using JUnit 5
2. Annotate the test class with `@ContextConfiguration` to tell the runner where the definitionsof the beans are
    - `@ContextConfiguration` accepts the _classes_ and _locations_ attributes

In **Spring Boot** just annotate the test class with `@SpringBootTest`.

## What is the preferred way to close an application context? Does Spring Boot do this for you?

- Call the `close()` or the `registerShutdownHook()` of the `ConfigurableApplicationContext` implementation class.
- Call the `exit(...)` method of the `SpringApplication` class

**Spring Boot** registers a shutdown hook with the JVM to make sure the application exits appropriately.
Furthermore, it exposes an actuator endpoint to shutdown the context on demand.

## Are beans lazily or eagerly instantiated by default? How do you alter this behaviour?

_Spring_ instantiate beans eagerly by default.

To make a bean lazy:
- Create it as `Prototype`
- Enable lazy component scanning by using the `@ComponentScan(lazyInit = true)` annotation
- Use the `@Lazy` annotation
  - The _lazy_ annotation can be used on `@Configuration` classes, `@Bean` methods, `@Component` classes, and along with
the `@Autowired` annotation.

## What is a property source? How would you use @PropertySource?

A property source represents a source of name/value property pairs.
The `@PropertySource` annotation is used for adding a property source to _Spring's Environment_.
It accepts the list of resource locations of the properties file to be loaded.

```java
@Configuration
@PropertySource("classpath:/it/me/my.properties")
public class AppConfig {
  @Autowired
  Environment env;

  @Bean
  public TestBean testBean() {
    TestBean testBean = new TestBean();
    testBean.setName(env.getProperty("testbean.name"));
    return testBean;
  }
}
 ```

## What is a BeanFactoryPostProcessor and what is it used for?<br>When is it invoked?

`BeanFactoryPostProcessor` is an interface and beans that implement it can alter others
bean **definitions**.  
It is invoked during startup of the Spring context, **after** all bean definitions are loaded.
For example, it is used by the property source mechanism to resolve `@Value(${placeholder})` placeholders
by injecting values read from the _Spring_ environment and its set of property sources, declared using the
`@PropertySource` annotation.
The interface define the method `postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)`.

```java
public class CustomBeanFactory implements BeanFactoryPostProcessor {
  @Override
  public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
    for (String beanName : beanFactory.getBeanDefinitionNames()) {
      BeanDefinition beanDefinition = beanFactory.getBeanDefinition(beanName);
      // manipulate bean definition
    }
  }
}
```

## What is a BeanPostProcessor and how is it different to a BeanFactoryPostProcessor? What do they do?<br>When are they called?

`BeanPostProcess` is an interface that adds some processing logic before and after executing the bean
initialization method after the _Spring IoC Container_ instantiates the bean.
The interface defines the methods `postProcessBeforeInitialization(Object bean, String beanName)`
and `postProcessAfterInitialization(Object bean, String beanName)`.

## What does component scanning do?

The process of searching the classpath for classes that can contribute to the application context is called component scanning.
A class must be annotated with a _Stereotype_ annotation in order to be scanned and registered by component scanning.

## What is the behaviour of the annotation @Autowired with regards to field injection, constructor injection and method injection?

`@Autowired` annotation populates the annotated attribute with the desired bean.
The _Bean Factory_ first looks for a bean of the annotated attribute type, then it tries using the name
if none or more than one are returned it throws a `NoSuchBeanDefinitionException` exception.  
If multiple matching it tries to match name and type.
`@Qualifier` or `@Primary` annotations can be used to alter that behaviour.  
When `@Autowired` is used on a field, `Reflection` is used to populate the field.
`@Autowired` cannot be used to autowire primitive types.

## How does the @Qualifier annotation complement the use of @Autowired?

`@Qualifier` annotation is used to give a name to a bean.
When _Spring_ cannot decide what to autowire based on type, it looks for a bean named with
`@Qualifier` annotation value.

## What is a proxy object and what are the two different types of proxies Spring can create?

A proxy is a wrapper around the target object, it is used to alter the behaviour of the target object.
Spring has two types of proxy, the prefered one is the _JDK Dynamic Proxy_, the other one
is the _CGLIB_.
The former can only proxy by interface, the latter is used when the target class does
not implement an interface, by subclassing it.
The disadvantage of these proxies is that they can only intercept public methods, final classes and
final methods are not supported too.

## What does the @Bean annotation do?

`@Bean` annotation indicates that a method produces a bean to be managed by the _Spring container_.
Typically it is used in `@Configuration` classes, in this case the bean methods may reference other
`@Bean` methods in the same class by calling them directly, this mechanism is called _inter-bean references_.

## What is the default bean id if you only use @Bean? How can you override this?

The default bean id is the name of the method.
You can override it by using the name/value attribute of the `@Bean` annotation.

## Why are you not allowed to annotate a final class with @Configuration

Spring uses CGLIB to proxy the `@Configuration` annotated class, and CGLIB proxy does not
support final classes because they cannot be subclassed.

## How do you configure profiles? What are possible use cases where they might be useful?

A profile can be configured by using the `@Profile` annotation.
The set of profile names for which the annotated component should be registered is indicated
by the value attribute of the annotation.
The profile name may contain the NOT operator, !, the annotated component is registered if the
profile is not active.  
`@Profile` can be used:
- on any class directly or indirectly annotated with @Component
- on custom annotations
- on any @Bean method

To select the active profiles the property `spring.profiles.active` can be used.
Profiles may also be activated in integration tests via the `@ActiveProfile` annotation.  
Profiles are used to modify the application behaviour, like loading an implementation instead
of another, by simple modifying the external configuration.


## Can you use @Bean together with @Profile?

Yes.

## Can you use @Component together with @Profile?

Sure.

## How many profiles can you have?

Many as you want.


## How do you inject scalar/literal values into Spring beans?

To inject primitive types you must use the `@Value` annotation.  
It supports _SpEL_ language `#{systemProperties.myProp}`.
Alternatively, values may be injected using `${my.app.myProp}` style property placeholders.  
The `@Value` processing is performed by a _BeanPostProcessor_, meaning that `@Value` annotation
can't be used within _BeanPostProcessor_ or _BeanFactoryPostProcessor_ types.

## What is Spring Expression Language (SpEL for short)?

_SpEL_ is a powerful expression language that supports manipulating an object at runtime.  
It supports the following functionality:
- Literal expressions
- Boolean and relational operators
- Regular expressions
- Class expressions
- Accessing properties, arrays, lists, maps
- Method invocation
- Relational operators
- Assignment
- Calling constructors
- Bean references
- Array construction
- Inline lists
- Ternary operator
- Variables
- User-defined functions
- Collection projection
- Collection selection
- Templated expressions

## What is the Environment abstraction in Spring?

The _Spring Environment_ represents the environment in which the application is running.
It unifies access to all types of property sources, such as property files, JVM system properties,
system environment variables and servlet context parameters.  

## Where can properties in the environment come from – there are many sources for properties – check the documentation if not sure. Spring Boot adds even more.

The Spring boot `PropertySource` order is:
- Devtools global settings properties on your home directory (~/.spring-boot-devtools.properties when devtools is active).
- `@TestPropertySource` annotations on your tests.
- `@SpringBootTest#properties` annotation attribute on your tests.
- Command line arguments.
- Properties from `SPRING_APPLICATION_JSON` (inline JSON embedded in an environment variable or system property)
- ServletConfig init parameters.
- ServletContext init parameters.
- JNDI attributes from java:comp/env.
- Java System properties (System.getProperties()).
- OS environment variables.
- A RandomValuePropertySource that only has properties in random.*.
- Profile-specific application properties outside of your packaged jar (`application-{profile}.properties` and YAML variants)
- Profile-specific application properties packaged inside your jar (`application-{profile}.properties` and YAML variants)
- Application properties outside of your packaged jar (application.properties and YAML variants).
- Application properties packaged inside your jar (application.properties and YAML variants).
- @PropertySource annotations on your `@Configuration` classes.
- Default properties (specified using SpringApplication.setDefaultProperties).

## What can you reference using SpEL?

Beans.

## What is the difference between $ and # in @Value expressions?

The former is the property placeholder style, used to inject primitives types,
the latter indicates that it is a _SpEL_ expression.

# Aspect-Oriented Programming (AOP)

## What is the concept of AOP? Which problem does it solve? What is a cross-cutting concern?

_AOP_, Aspect-Oriented Programming, is a programming paradigm where is possible to
alter the behaviour of the existing code without modification of the code
itself.  
_AOP_ help to avoid _tangling_ and _scattering_.
```java
// tangling and scattering example, logging is mixed with business-logic
// and scattered across the codebase.
public void aMethodService() {
  logger.entering("MyService","aMethodService");
  anotherService.myMethod();
  logger.exiting("MyService","aMethodService");
}

public void anotherMethodService() {
  logger.entering("MyService","anotherMethodService");
  suchService.myMethod();
  logger.exiting("MyService","anotherMethodService");
}
```

A cross-cutting concern is a functionality shared across multiple application modules.
Such as logging, because is something that can happen across our application.

## What is a pointcut, a join point, an advice, an aspect, weaving?

A __pointcut__ is an expression that matches join points, for example the execution
of a method with a certain name.  

A __join point__ is a point during the execution of a program, in _Spring AOP_ is
always a method execution.  

An __advice__ is the code to run by an _aspect_ at a particular _join point_.  

An __aspect__ is the class that contains the cross-cutting concern logic.
It is annotated with the `@Aspect` annotation.  

The __weaving__ is the mechanism used by `Spring AOP` to proxy the target object to create an advised object.
`Spring AOP` only supports runtime weaving by using `JDK Dynamic Proxy` or `CGLIB`, the former is prefered, but
you can switch to the latter by using `@EnableAspectJAutoProxy(proxyTargetClass=true)`.

## How does Spring solve (implement) a cross-cutting concern?

Spring implement a _cross-cutting concern_ by weaving the _aspect_ to the target object, creating a proxy that
can intercept the execution of the method as specified by the _pointcuts_.

## Which are the limitations of the two proxy-types?

_JDK Dynamic Proxy_ limitations:
- Target class must implement an interface
- *-private, protected and final methods are not supported
- Methods not defined in the interface are not supported
- Does not support self-referencing, if a matched _join point_ method calls
another method of the same class matched by a _join point_, the advice is called
just once.

_CGLIB_ limitations:
- Target class can't be _final_, as _CGLIB_ can't subclass it
- Private and final methods are not supported
- Protected methods are supported but not recommended

## How many advice types does Spring support? Can you name each one?

In _Spring AOP_ there are multiple advice types:
- `@Before` advice, will execute before the _join point_, it can't prevent the execution
of the method unless it throws an exception
- `@AfterReturning` advice, will execute after the _join point_ completes normally , meaning that
the target method returns normally without throwing an exception.
- `@AfterThrowing` advice, will execute after the _join point_ execution that ended by throwing
an exception
- `@After` advice, will execute after the _join point_ execution, no matter the how execution ended.
- `@Around` advice, which is the most powerful advice, will intercept the target method and surround
the _join point_. It can perform custom behaviour before and after target invocation, it can execute or
not the target method and stop the propagation of a possible exception raised by the target.

## If shown pointcut expressions, would you understand them?

Possible _pointcut_ expressions are:
- `execution([Modifiers] [ReturnType] [FullClassName].[MethodName]([Arguments]) throws [ExceptionType])`
- `whithin()`, limits maching to _join point_ of certain types, `@Pointcut("within(it.manuel..*)")`
- `target()` and `this()`, the former is used to match a class that implements the specified interface, and
the latter is used to match a specific class type.
- `@target` limits matching to _join point_ where the class of the target method has a specific annotation,
`@Pointcut("@target(org.springframework.stereotype.Service)")`
- `@args` limits matching to _join point_ where the argument of the target method has a specific annotation,
`@Pointcut("@args(it.manuel.annotations.MyArgAnnotation)")`
- `@annotation` limits matching to _join point_ where the target method has a specific annotation,
`@Pointcut("@annotation(org.springframework.beans.factory.annotation.Qualifier)")`

The _pointcut_ expression supports wildcards such as `+` and `*`, and logical operators like `&&`, `||` and `!`.
The `+` wildcard indicates that the method can also be matched in subclasses `it.manuel.+.helloWorld`.  

## What is the JoinPoint argument used for?

The `JoinPoint` type is the argument type used for all the _advice_ annotated methods, except `@Around`,
which argument type is `ProceedingJoinPoint`.

## What is a ProceedingJoinPoint? Which advice type is it used with?

The `ProceedingJoinPoint` is the argument type for `@Around` advice.
It exposes the `.proceed` method used to call the method matched by the _pointcut_.

# Data Management: JDBC, Transactions

## What is the difference between checked and unchecked exceptions?

A checked exception is a compile-time exception that must be managed.
An unchecked exception is a runtime exception, it's not required to manage it.
The latter is used by the data access mechanism to report errors, by throwing
exceptions that extend the `DataAccessException` class.  
Rollback is automatic only on unchecked exceptions.

## How do you configure a DataSource in Spring?

To access a database using _JDBC_, the following are needed:
- A DB driver to dialogue with the database
- The DB connection url
- The DB credentials

A _DataSource_ is created using the previous information, and each database library
must implement it.  

In Spring to programmatically create a `DataSource` you have to define a `DataSource` bean
and return an instance of the implementation provided by the DB vendor you are using.

In __Spring Boot__, is possible to define a _DataSource_ by setting properties
```yaml
app.datasource.url=jdbc:h2:mem:hello
app.datasource.username=hello
app.datasource.password=world
```

__Spring Boot__ also provide a class builder named `DataSourceBuilder`, it auto-detect the driver
based on what is available on the classpath or based on the _JDBC_ URL.

## What is the Template design pattern and what is the JDBC template?

The template design pattern is one of the most popular, it
enables the developer to implement a complex functionality by encapsulating
the logic in a single method, the so-called template method.
The effective implementation is defined later or delegated to subclasses.

```java
public abstract class MyTemplateBuilder {
  public final JdbcTemplate buildHouse() {
    setupFoundation();
    buildWalls();
    buildRoof();
    paintWalls();
  }

  public abstract setupFoundation();
  public abstract buildWalls();
  public abstract buildRoof();
  public abstract paintWalls();
}
```

The _JDBC Template_ is the central class to manage DB operations.
Under the hood it manages all the repetitive actions to execute a query,
such as opening and closing the connection, handling exceptions and transactions,
leaving application code to only provide SQL and extract results.

## What is a callback? What are the JdbcTemplate callback interfaces that can be used with queries? What is each used for? (You would not have to remember the interface names in the exam, but you should know what they do if you see them in a code sample). 

A callback is a method called by another method when this one finished the execution,
usually, the callback method argument is the return value of the former.

The `JdbcTemplate` query method accepts 3 callback interface types:
- `ResultSetExtractor<T>` interface expose the `<T> T extractData(ResultSet set)` method
- `RowMapper<T>`expose the `<T> List<T> mapRow(ResultSet set, int rowNum)` method
- `RowCallbackHandler` expose the `void processRow(ResultSet set)`  method

`ResultSetExtractor` is supposed to extract the whole `ResultSet`, it may contain multiple rows,
while `RowMapper` is fed with a row at a time.
Both are typically _stateless_.

`RowCallbackHandler` is fed with a row at time, and it is tipically _stateful_.

## Can you execute a plain SQL statement with the JDBC template?

`JdbcTemplate` execute plain SQL queries.

## When does the JDBC template acquire (and release) a connection, for every method called or once per template? Why?

_JDBC template_ acquire a connection for every method called.

## How does the JdbcTemplate support queries? How does it return objects and lists/maps of objects?

`JdbcTemplate` support queries for any type.  
It exposes many overloaded methods:
- `<T> List<T> query(...)`
- `Map<String, Object> queryForMap(...)`
- `<T> T queryForObject(...)`
- `<T> List<T> queryForList(...)`

## What is a transaction? What is the difference between a local and a global transaction?

In a DB context, a transaction is an atomic set of operations, when all operations have completed successfully
the changes are persisted, else if one operation fails, the transaction is _rolled back_ leaving the database pristine.  

A local transaction is a transaction that involves only local operations,
whereas a global transaction involves external operations, such as an external database operation
or a message on an external queue.

## What does declarative transaction management mean?

Declarative transaction management is a non-invasive approach for managing transactions provided
by _Spring AOP_.
It means that you can enable and tag a class or a method transactional by using annotations.  
To use transactions is sufficient to add `@EnableTransactionManagement` to your configuration
and annotate your classes or methods with the `@Transactional` annotation.

## What is the default rollback policy? How can you override it?

By default, a transaction is rolled back when an unchecked exception occurs within
the transactional code.  
It is possible to override that policy by configuring the `@Transactional` annotation parameters:
- `noRollbackFor` to define which exception types must not cause a transaction rollback
- `noRollbackForClassName` to define which exception types must not cause a transaction rollback
- `rollbackFor` to define which exception types must cause a transaction rollback
- `rollbackForClassName` to define which exception types must cause a transaction rollback

## What is the default rollback policy in a JUnit test, when you use the @RunWith(SpringJUnit4ClassRunner.class) in JUnit 4 or ExtendWith(SpringExtension. class) in JUnit 5, and annotate your @Test annotated method with @Transactional?

By default tests do not commit, they always require a rollback.
That behaviour can be altered by using the `@Rollback(false)` annotation or
the `@Commit` annotation.

## Are you able to participate in a given transaction in Spring while working with JPA?

Yes, by using the `TransactionAwareDataSourceProxy` class.

## Which PlatformTransactionManager(s) can you use with JPA?

- `JpaTransactionManager`, also supports direct `DataSource` access within transaction
- `JtaTransactionManager`, necessary for accessing multiple transaction resources within the same transaction
- `HibernateTransactionManager`, ref
[https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#orm-jpa-tx](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#orm-jpa-tx)

## What do you have to configure to use JPA with Spring? How does Spring Boot make this easier?
To enable _JPA_ must be used the `@EnableJpaRepositories` annotation, it will scan the package
of the annotated configuration class for _Spring Data_ repositories by default.
To change the package to scan the `basePackages` or `basePackageClasses` attributes can be used.  

To configure _JPA_ an `EntityManagerFactory` instance must be provided, for example by using the
`LocalContainerEntityManagerFactoryBean` class.
The `FactoryBean` reads configuration settings from the `META-INF/persistence.xml` config file.  

_Spring Boot_ automagically setup _JPA_ and transactions if the `spring-boot-starter-data-jpa` module is on the
classpath.

# Spring Data JPA

## What is a Spring Data Repository interface?
## How do you define a Spring Data Repository interface? Why is it an interface not a class?

A _Spring Data Repository_ interface typically extends one of the interfaces in the `Repository<T, ID extends Serializable>`
hierarchy.
Such interfaces expose a ready-to-use set of methods to manipulate entities.
Beans of this type are called _instant repositories_, because Spring looks for these interfaces types and implements
them at runtime to create beans.  

Alternatively can be used the `@RepositoryDefinition` annotation to create a repository, passing to the
`domainClass` attribute the type of entity and to the `idClass` attribute the id type.

## What is the naming convention for finder methods in a Spring Data Repository interface?

The supported query keywords are
- find...By
- read...By
- get...By
- query...By
- search...By
- stream...By

To specify the number of returned records:
- ...First<number>
- ...Top<number>
  
To return only unique results:
- ...Distinct...

## How are Spring Data repositories implemented by Spring at runtime?

Spring uses a factory bean named `RepositoryFactorySupport` to create instances
of a given repository interface.
It creates a _JDK Dynamic_ proxy implementing the repository interface and apply an _advice_ handling the control to the
`QueryExecutorMethodInterceptor` class.

## What is @Query used for?

The `Query` annotation is used to declare finder queries directly on repository methods.
It accepts both _JPQL_ (only _JPA_) and native _SQL_ queries.
The `@Query` annotation takes precedence over `@NamedQuery` annotation.
To create update queries add the `@Modifying` annotation to the `@Query` annotated method.

#  Spring MVC Basics

## What is the @Controller annotation used for?

The `@Controller` _stereotype_ annotation is used to indicate that the annotated class provides handler methods
for _HTTP_ requests.
Typically it is used in combination with handler methods annotated with `@RequestMapping`.

## How is an incoming request mapped to a controller and mapped to a method?

The _Spring Web MVC_ entry point is the `DispatcherServlet` class, it dispatches HTTP requests
to handlers that match the request pattern.

## What is the difference between @RequestMapping and @GetMapping?

The `@GetMapping` is equal to `@RequestMapping(method = RequestMethod.GET)`.

## What is @RequestParam used for?

The `@RequestParam` annotation is used to indicate a query parameter in the request URL.
To indicate an optional query parameter set to false the `required` attribute.
A default value can be provided by using the `defaultValue` attribute.

## What are the differences between @RequestParam and @PathVariable?

The `@PathVariable` annotation is used to indicate a path parameter in the request URL.

## What are the ready-to-use argument types you can use in a controller method?

The signatures of the handler methods are very flexible, they can receive almost any type-specific
to a web application.
The following are some parameters types:
- `@RequestHeader` annotated parameters to gather a request header
- `WebRequest`, `NativeWebRequest` and `ServletRequest` provide access to headers, request and session attributes
- `ServletRespose` to enrich the response
- `HttpSession` to access the session
- `Principal` to access the credentials
- `HttpMethod` which represents the request method
- `Locale` to get the current locale
- `BindingResult` to access the validation reports
- `Error` to access the validation errors
- `InputStream`, `Reader` to access the raw body of the request
- `OutputStream`, `Writer` to access the raw body of the response
- `@RequestParam` annotated parameter to get the query parameters
- `@PathVariable` annotated parameter to access the path variables
- `@MatrixVariable` annotated parameter to access the matrix variables, ex. `http://hello.world/;g=1;u=3;=9`
- `@CookieValue` annotated parameter to access the cookies
- `Map`, `Model` or `ModelMap` that contain the data that the view template will render
- `RedirectAttributes` along with `@ModelAttribute` to specify data to be used by a view which was redirected to
- `@SessionAttribute` or `@RequestAttribute` to access to request and session attributes

## What are some of the valid return types of a controller method?

- `HttpEntity`, `ResponseEntity` to return a full response
- `HttpHeaders` to return a response without a body
- `String` to specify a logical view name
- `View` 
- `Map` or  `Model` to specify attributes for the model
- `ModelAndView` to specify model and view to use
- `void` to return an empty response or select the default view
- `DeferredResult`, `CompletableFuture`, etc. to produce an async response from any thread
- `Callable<V>` to produce an async response in a _Spring MVC_ thread

# Spring MVC REST

## What does REST stand for?

REpresentational  
State  
Transfer

## What is a resource?

A _REST_ resource is something referenced by an URL.
I can get the apple number 1 by using the
url `/api/apple/1`.

## Is REST secure? What can you do to secure it?

It is secure if used safely.
For example by using the `HTTPS` protocol and require some credentials to access
a protected resource.

## Is REST scalable and/or interoperable?

Yes, it is scalable and interoperable.
It does not mandate a specific technology either for the client or server.

## Which HTTP methods does REST use?

_REST_ use the following methods
- _GET_, to get a resource, it is idempotent
- _POST_, to create a resource, it is not idempotent
- _PUT_, to update a resource, it is idempotent
- _PATCH_, to partially update a resource, it is not idempotent
- _DELETE_, to delete a resource, it is idempotent
- _OPTIONS_, to list the supported methods of a resource, it is idempotent
- _HEAD_, to only get the headers related to a resource, it is idempotent

## What is an HttpMessageConverter?

The `HttpMessageConverter` is the interface used for converting from and to _HTTP_ requests and responses.  
By default the following converters are enabled:
- `ByteArrayHttpMessageConverter`, converts byte arrays
- `StringHttpMessageConverter`, converts Strings
- `ResourceHttpMessageConverter`, converts org.springframework.core.io.Resource for any type of octet stream
- `SourceHttpMessageConverter`, converts javax.xml.transform.Source
- `FormHttpMessageConverter`, converts form data to/from a MultiValueMap<String, String>.
- `Jaxb2RootElementHttpMessageConverter`, converts Java objects to/from XML (added only if JAXB2 is present on the classpath)
- `MappingJackson2HttpMessageConverter`, converts JSON (added only if Jackson 2 is present on the classpath)
- `MappingJacksonHttpMessageConverter`, converts JSON (added only if Jackson is present on the classpath)
- `AtomFeedHttpMessageConverter`, converts Atom feeds (added only if Rome is present on the classpath)
- `RssChannelHttpMessageConverter`, converts RSS feeds (added only if Rome is present on the classpath)

## Is @Controller a stereotype? Is @RestController a stereotype?

`@Controller` is a stereotype annotation.
`@RestController` is a specialization of the `@controller` annotation, but it can't be considered
a stereotype annotation.

## What is the difference between @Controller and @RestController?
## When do you need to use @ResponseBody?

`@RestController` is meta-annotated with `@ResponseBody` annotation to indicate that
the value returned by a handler method needs to be bind to the response body, thus serialized.

## What are the HTTP status return codes for a successful GET, POST, PUT or DELETE operation?

- 200 Ok for a _GET_
- 201 Created for a _POST_
- 200 Ok or 204 No Content for a _PUT_
- 200 Ok, 202 Accepted or 204 No Content for a _DELETE_

## When do you need to use @ResponseStatus?

The `@ResponseStatus` annotation is used to specify the response status:
- when a controller method completes successfully
- along with `@ExceptionHandler` and `@ControllerAdvice` for a method that returns an exception
- for a specific exception by directly annotating it

## Where do you need to use @ResponseBody? What about @RequestBody?

The `@RequestBody` annotation is used to indicate a method parameter should be bound to the body
of the request.
The `@ResponseBody` annotation indicates the return value should be bound to the web response body.

## What Spring Boot starter would you use for a Spring REST application?

`spring-boot-starter-web`

# Security

## What are authentication and authorization? Which must come first?

`Authentication` is the process used to check if the credentials provided by the user
are valid.
`Authorization` is the process used to check if the authenticated user has the permissions
to access the requested resource.

## Is security a cross-cutting concern? How is it implemented internally?

Security is a cross-cutting concern, it is applied across multiple modules of our
application.
Spring uses _AOP_ to implement the security mechanism.

## What is the delegating filter proxy?

The `DelegatingFilterProxy` acts as an interceptor for secured request, delegating the calls
to the chained security filter beans, called `SpringSecurityFilterChain`.

## What is the security filter chain?

The `SecurityFilterChain` is a list of filters, each one with a particular responsibility.
The filters are added or removed from the chain depending on which services are required.
The filters ordering is important as there are dependencies between them.
It is used by the `FilterChainProxy` to match the url with the filters to apply.

## What is a security context?

The security context is where are stored the details of the current security context, such as
the details of the _Principal_ that is using the application.

## What does the ** pattern in an antMatcher or mvcMatcher do?

Taking the following pattern example `/admin/user/**`,
they both matches any path that starts with `/admin/user/`,
farther the `mvcMatcher` also matches the `/admin/user/123.json` path.


## Why is the usage of mvcMatcher recommended over antMatcher?

`mvcMatcher` is considered more secure than `antMatcher`.
That is because `mvcMatcher` use the same rules that _Spring MVC_ uses for matching.

## Does Spring Security support password encoding?

Yes, just define a bean of the encoder type you want to use and use it.

## Why do you need method security? What type of object is typically secured at the method level (think of its purpose not its Java type).

To protect sensitive resources, for example the resource that represents the users.

## What do @PreAuthorize and @RolesAllowed do? What is the difference between them?

The `@PreAuthorize` annotation is used to run a check before a method invocation to decide if
the user has the permissions to call the method.
I can decide to deny a method execution to users which names start with an m.  
The `@RolesAllowed` annotation is used to check if the _Principal_ role is one of the roles
specified by the annotation.
The `@Secured` and `@RolesAllowed` annotations are equivalent, the latter is part of the
JSR 250 spec.  

To enable the security annotations the `@EnableGlobalMethodSecurity` annotation must be used.
It exposes some properties:

- the `prePostEnabled` property enables _@pre_/_@post_ annotations
- the `securedEnabled` property determines if the `@Secured` annotation should be enabled
- the `jsr250Enabled` property allows us to use the `@RoleAllowed` annotation

## How are these annotations implemented?

_Spring AOP_.

## In which security annotation, are you allowed to use SpEL?

Yes, all the _@pre_ and _@post_ annotations support _SpEL_:
- `@PreAuthorize`
- `@PostAuthorize`
- `@PreFilter`
- `@PostFilter`

# Testing

## What type of tests typically use Spring?

Tipically, a _Spring_ application has unit and integration tests.
__Unit__ tests are used to test the smallest testable parts of an
application, isolated from any others units.
The dependencies are replaced by stubs and mocks to reproduce the expected behaviour.
By default _Spring Boot_ uses _Junit 5_, but it supports _Junit 4_ too.
__Integration__ tests are used to test the multiple units interactions of an
application.

## How can you create a shared application context in a JUnit integration test?

The so-called `TestContext` encapsulates the context of each test, and provide context management
and caching support for the test instance.  
By default, the `ApplicationContext` is not accessible by the test instances.
To expose the `ApplicationContext` to the test instances the test class must implement
the `ApplicationContextAware` interface.
As an alternative to implementing the `ApplicationContextAware` interface, it is possible to
inject the application context through the `@Autowired` annotation.  
Once a `TestContext` loads an `ApplicationContext`, that context is cached and reused for all
subsequent tests within the same test suite.

## When and where do you use @Transactional in testing?

The `@Transactional` annotation can be used on the test class or
directly on the method.
By default a rollback occurs after completition of the test.
To change that behaviour you can use the `@Rollback(false)` annotation or
the `@Commit` annotation.
Also, the `defaultRollback` property of the `@TransactionConfiguration` annotation
can be used to change that behaviour.

## How are mock frameworks such as Mockito or EasyMock used?

To run a test that uses _Mockito_ you have to:
1. Declare the mock by using the `@Mock` annotation
2. Inject the mock in the tested unit by using the `@InjectMocks` annotation
3. Configure the mock by using the `when` statement
4. Run test
5. Assert results

_Mockito_ provides a _Junit 5_ extension class `MockitoExtension` 
that automatically initialize mocks.
In alternative, you can use the static method `MockitoAnnotations.initMocks`.

_Spring Boot_ simplifies the mocking process by providing the
`@MockBean` annotation.
It adds the mock to the _Spring_ `ApplicationContext`.
Can be used directly on the test class or `@Configuration` classes and fields.
When used on a field of the test class, the mock is injected into the field.

## How is @ContextConfiguration used?

The `@ContextConfiguration` annotation is used on a test class to specify the components classes
to use to configure the `ApplicationContext`.  
A component class is:
- A class annotated with `@Configuration`
- A component (i.e., a class annotated with @Component, @Service, @Repository, etc.)
- A `JSR-330` compliant class that is annotated with _javax.inject_ annotations
- Any class that contains `@Bean`-methods
- Any other class that is intended to be registered as a Spring component (i.e., a Spring bean in the ApplicationContext), potentially taking advantage of automatic autowiring of a single constructor without the use of Spring annotations

When `@ContextConfiguration` is used without properties it searches for a file named
`testClassName-context.xml` in the same location of the test class.  

## How does Spring Boot simplify writing tests?

_Spring Boot_ provides all the libraries needed to run a well-rounded test by just
using the `spring-boot-starter-test` dependency:
- JUnit , the de-facto standard for unit testing Java applications
- Spring Test & Spring Boot Test, utilities and integration test support for Spring Boot applications
- AssertJ , a fluent assertion library
- Hamcrest , alibrary of matcher objects (also known as constraints or predicates)
- Mockito , a Java mocking framework
- JSONassert , an assertion library for JSON
- JsonPath ,  XPath for JSON

## What does @SpringBootTest do? How does it interact with @SpringBootApplication and @SpringBootConfiguration?

_Spring Boot_ exposes the `@SpringBootTest` annotation to configure an `ApplicationContext`
by using the `SpringBootContextLoader` loader class.  
It provides the following features:
- automatically searches for any `@SpringBootConfiguration`-classes when no `@ContextConfiguration` 
annotation is defined
- allow to customize the `Environment` properties to be defined using the `properties` attribute
- allow application arguments to be defined using the `args` attribute
- support for different webEnvironment modes, including the ability to start a fully running webserver listening on a defined or random port
- register a `TestRestTemplate` bean

# Spring Boot Basics

## What is Spring Boot?

_Spring Boot_ is a way to ease create production-ready, stand-alone Spring-based applications.
It provides an opinionated starter configuration for code and annotation, to quickly start new Spring projects with no time
with minimal or zero configurations.

## What are the advantages of using Spring Boot?

- Very easy to develop Spring based applications
- It reduces development time and increase productivity
- It avoids writing boilerplate code, annotations and xml configuration
- Easy integration with Spring ecosystem, such as Spring JDBC, Spring ORM, Spring Data, Security...
- It reduces developer effort by providing opinionated defaults configuration
- It provides embedded HTTP servers, Tomcat by default
- It provides a CLI to develop and test Spring applications

## What things affect what Spring Boot sets up?

_Spring Boot_ attempts to automatically configure the Spring application based on which
libraries are available in the classpath.
The auto-configuration is enabled by adding the `@EnableAutoConfiguration` or `@SpringBootApplication` annotations
to one of the `@Configuration` classes.

## What is a Spring Boot starter? Why is it useful?

A starter is a set of convenient dependency descriptors that can be included in a Spring application.
For example, if you want to get started using Spring and JPA for database access, include the
`spring-boot-starter-data-jpa` dependency in the project.
All Spring starters follow the naming pattern `spring-boot-starter-*`.

## Can you control logging with Spring Boot? How?

 The default auto-configured logger in _Spring Boot_ is logback.
 Appropriate logback routing is also included to ensure that dependant libraries that use others
 providers work correctly.  
 Logging can be configured by using some well-known properties in the `logging.*` group:
 - `logging.file.name` or `logging.file.path` to indicate the file where to log
 - `logging.logback.rollingpolicy.*` to configure the file rotation policy
 - `logging.level.<logger-name>=<level>`, where level is one of `TRACE`, `DEBUG`, `INFO`,
`WARN`, `ERROR`, `FATAL` or `OFF`, to configure the log level of a specific logger.
- `logging.group.<group-name>=<loggers>` to group loggers together.
_Spring Boot_ provides two pre-defined logging groups, `web` and `sql`.  

Another option to configure logging is to use a dedicated configuration file:
- `logback-spring.xml`
- `logback.xml`
- `log4j-spring.xml` or `log4j.xml`
- `logging.properties`

## Where does Spring Boot look for application.properties file by default?

Ordered by precedence:
1. In the classpath root
2. In the `/config` package classpath
3. In the current directory
4. In the `/config` subdirectory in the current directory
5. In the immediate child directories of the `/config` subdirectories

## How do you define profile specific property files?

_Spring Boot_ attempt to load profile-specific files using the naming convention
`application-<profile>`.
For example, if the `dev` profile is enabled, then both `application.yml` and `application-dev.yml`
will be considered.
Profile specific files always override the non-specific ones.
If several profiles are specified, a last-wins strategy applies.
If no profiles are activated, the properties from `application-default` are considered.

## How do you access the properties defined in the property files?

You can access to properties values defined in property files by:
- retrieving the current `Environment`
- using the `@Value` annotation withing the `${<property-name>}` placeholder
- using the `@ConfigurationProperties` annotation

## What properties do you have to define in order to configure external MySQL?

1. `spring.datasource.url=jdbc:mysql://mysql-hostname/db-name`
2. `spring.datasource.username=hello`
3. `spring.datasource.password=world`

## How do you configure default schema and initial data?

_Spring Boot_ loads _SQL_ from the standard root classpath locations:
- `schema.sql`
- `data.sql`

In addition it processes the files:
- `schema-<platform>.sql`, where _platform_ is the value of the `spring.datasource.platform` property
- `data-<platform>.sql`

_Spring Boot_ automatically creates the schema for embedded data sources.
This behaviour can be customized by using the `spring.datasource.initialization-mode` property.

## What is a fat jar? How is it different from the original jar?

A fat jar is a jar that contains all the application dependencies.

## What embedded containers does Spring Boot support?

- Tomcat
- Jetty
- Undertow
- Reactor Netty

# Spring Boot Auto Configuration

## How does Spring Boot know what to configure?

_Spring_ auto-configuration attempts to automatically configure the _Spring_ application
based on the jar dependencies that are present in the classpath.

## What does @EnableAutoConfiguration do?

It enables the _Spring_ auto-configuration of the _Spring Application Context_.
Auto-configuration classes are usually applied based on the classpath content and
what beans are available.
Auto-configuration classes are Spring `@Configuration` beans located by the
`SpringFactoriesLoadeer` mechanism.
Those beans are usually `@Conditional` beans,
most often using `@ConditionalOnClass` and `@ConditionalOnMissingBean` annotations.

## What does @SpringBootApplication do?

The `@SpringBootApplication` annotation is meta-annotated with
- `@SpringBootConfiguration`, an alias for the `@Configuration` annotation
- `@EnableAutoConfiguration`, to enable the _Spring_ auto-configuration
- `@ComponentScan` to scan the current packages and all its children for component beans

## Does Spring Boot do component scanning? Where does it look by default?

The `@SpringBootApplication` enable component scanning for the current package
and all its sub-packages.

## How are DataSource and JdbcTemplate auto-configured?

_Spring Boot_ automatically creates a JdbcTemplate.
The `DataSource` is auto-configured only if in the classpath
is present an in-memory database dependency.

## What is spring.factories file for?

The `spring.factories` file is used by _Spring Boot_ to locate the auto-configuration candidates and
to register `ApplicationListener`s.  

A sample of the `spring.factories` file is:
```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

## How do you customize Spring Boot auto configuration?

The `@EnableAutoConfiguration` annotation supports the following properties:
- `exclude`, to specify the list of classes to skip
- `excludeName`, to specify the list of classes names to skip

Alternatively:
- Use the `spring.autoconfigure.exclude` property to exclude
some auto-configuration classes.
- Use the `spring.factories` file to specify the auto-configuration classes
- Override the beans created by the auto-configuration classes

## What are the examples of @Conditional annotations? How are they used?

Conditional annotations are used to skip beans definitions under certain conditions.
For example the `JdbcTemplateAutoConfiguration` is annotated with the following annotations:
```java
 @ConditionalOnClass(value={javax.sql.DataSource.class,org.springframework.jdbc.core.JdbcTemplate.class})
 @ConditionalOnSingleCandidate(value=javax.sql.DataSource.class)
 ```
 Meaning that a `JdbcTemplate` is auto-configured only if:
 - A **single** datasource exists
 - The `JdbcTemplate` class is on the classpath

# Spring Boot Actuator

## What value does Spring Boot Actuator provide?

It provides production-ready features to help you monitor and manage the _Spring Boot_ application.

## What are the two protocols you can use to access actuator endpoints?

- JMX
- HTTP

## What are the actuator endpoints that are provided out of the box?

An actuator endpoint can be enabled and exposed, when both are true then the
actuator endpoint is available.

By default all endpoints except for `shutdown` are enabled.
To enable/disable an actuator use the property pattern `management.endpoint.<name>.enabled`.  
To disable all endpoints by default use the property `management.endpoints.enabled-by-default=false`.  

All jmx endpoints are exposed out-of-the-box.
Instead only few http endpoints are exposed by default:
- `health` endpoint
- `info` endpoint

Additional endpoints are enabled if the application is a web application:
- `heapdump`, returns an `hprof` heap dump file
- `jolokia`, when jolokia is in the classpath, it exposes jmx beans over http
- `logfile`, if `logging.file.name` or `logging.file.path` properties have been set, returns the content of the logfile
- `prometheus`, exposes metrics in a format that can be scraped by _Prometheus_

To change which endpoints are exposed, use the the following properties:
- `management.endpoints.jmx.exposure.exclude`
- `management.endpoints.jmx.exposure.include`, default value is `*`
- `management.endpoints.web.exposure.exclude`
- `management.endpoints.web.exposure.include`, default value is `info, health`

You can customize the actuator web endpoints path by using the following properties
- `management.endpoints.web.base-path`
- `management.endpoints.web.path-mapping.<id>`

## What is info endpoint for? How do you supply data?

The info endpoint exposes arbitrary data.
_Spring Boot_ by default exposes the following data:
- any key from the `Environment` under the `info` key
- git information if a `git.properties` file is available
- build information if a `META-INF/build-info.properties` file is available

You can create a component class that implements the `InfoContributor` interface
to add additional data to the info endpoint.

## How do you change logging level of a package using loggers endpoint?

By doing a `POST` to the `/actuator/loggers` endpoint with the following body:
```json
{
  "configuredLevel": "DEBUG"
}
```

Pass null to reset the logger level.

## How do you access an endpoint using a tag?
## What is metrics for?

_Spring Boot_ provides a `metrics` endpoint that can be used to gather
the metrics collected by the application.  
Navigating to the `/actuator/metrics` display the list of available metrics.
Provide the metric name to retrieve the data of a particular metric, e.g. `/actuator/metrics/jvm.memory.max`.
You can use tags to drill down a specific metric data, e.g. `/actuator/metrics/jvm.memory.max?tag=area:nonheap`.

## How do you create a custom metric?

Inject the `MeterRegistry` bean in your component and use it to register a custom metric.

## What is Health Indicator?

It provides the health status of the current application.

## What is the Health Indicator status?

The HTTP status code in the response reflects the overall health status.

## What are the Health Indicator statuses that are provided out of the box?

By default it supports the following statuses:
- `UP`, no mapping by default
- `UNKNOWN`, no mapping by default
- `DOWN`, mapped to the code 503
- `OUT_OF_SERVICE`, mapped to the code 503

Any unmapped health status map to 200.

## How do you change the Health Indicator status severity order?

Using the `management.endpoint.health.status.order` property

# Spring Boot Testing

## When do you want to use @SpringBootTest annotation?

The `@SpringBootTest` annotation is an alternative to the `spring-test` `@ContextConfiguration`
annotation when you need _Spring Boot_ features.
The annotation works by creating the `ApplicationContext` used by the tests through `SpringApplication`.  
By default it will not start a server.
You can use the `webEnvironment` property to configure the test class web environment:
- `MOCK`, the default value, loads a `WebApplicationContext` and provides a
mocked web environment. It can be used with the `@AutoConfigureMockMvc` for mock-based testing of your web application.
- `RANDOM_PORT`, loads a `WebServerApplicationContext` and provides a real web environment.
The embedded server port can be injected by using the `@LocalServerPort` annotation or by injecting the `local.server.port`
property.
- `DEFINED_PORT`, is like `RANDOM_PORT` but the listening port is gathered from the `Environment`.
- `NONE`, Loads an `ApplicationContext` but does not provide any web environment.

## What does @SpringBootTest auto-configure?

- It uses the `SpringBootContextLoader` to confiure the `ApplicationContext`
- It registers a `TestRestTemplate` bean for use in web tests that are using a fully running webserver

## What dependencies does spring-boot-starter-test brings to the classpath?

- _JUnit_ , the de-facto standard for unit testing Java applications
- _Spring Test_ & _Spring Boot Test_, utilities and integration test support for Spring Boot applications
- _AssertJ_ , a fluent assertion library
- _Hamcrest_ , a library of matcher objects
- _Mockito _, a Java mocking framework
- _JSONassert_ , an assertion library for JSON
- _JsonPath_ , XPath for JSON

## How do you perform integration testing with @SpringBootTest for a web application?

- Use `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)` to start the embedded server
- Use `@LocalServerPort` annotation on a field to inject the listening port
- Use the `@TestRestTemplate` to perform requests against the webserver
- Assert the results

## When do you want to use @WebMvcTest? What does it auto-configure?

The `@WebMvcTest` annotation is used for tests that focus **only** on _Spring MVC_ components.
Using this annotation the auto-configuration is disabled and only _MVC_ related configurations
are configured.
Tests annotated with `@WebMvcTest` annotation will also auto-configure _Spring Security_ and `MockMvc`.
Use the `@WithMockUser` annotation on a test method to mock the `Principal`
used by the `mockMvc` requests.
The `@WithMockUser` annotation supports the following properties:
- `username`, to set the principal username
- `roles`, to set the principal roles
- `password`, to set the principal password

## What are the differences between @MockBean and @Mock?

- `@Mock` can only be applied to fields and parameters
- `@MockBean` can only be applied to classes and fields
- `@Mock` can be used to mock java classes or interfaces
- `@MockBean` only allows for  mocking _Spring_ beans or creation of mock _Spring_ beans
- `@MockBean` can be used only with a running `SpringRunner` or `SpringExtension`
- `@MockBean` can be used for meta-annotate other beans
- `@MockBean` and `@Mock` are both included in `spring-boot-starter-test`

## When do you want @DataJpaTest for? What does it auto-configure?

Like `@WebMvcTest` it disables the full auto-configuration, and apply only configurations
relevant to JPA.
By default tests annotated with `@DataJpaTest` are transactional and roll back at the end of each test execution.
It forces the use of an embedded in-memory database.