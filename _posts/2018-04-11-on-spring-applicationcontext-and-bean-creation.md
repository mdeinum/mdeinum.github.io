---
layout: post
title: "On Spring ApplicationContext and Bean Creation"
date: 2018-04-12
last-updated: 2022-12-15
tags: [spring]
published: true
---

There appears to be a lot of confusion in the community on how to expression bean configuration or rather how to convert bean configuration from XML to Java (or Groovy, or Kotlin, or...) as it appears as if the different ways of configuration are completely different things.

But in fact they are more or less the same it is **configuration expressed in a format (Java, XML, Properties) to instruct the Spring `ApplicationContext` (well actually the `BeanFactory`) to create beans based on those definitions** (aka [`BeanDefinition`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanDefinition.html)). A `BeanDefinition` is the recipe on how to prepare/create a bean.

Lets create a small sample using a few classes and lets take a look under the hood on what is happening with those class.

```java
public record Person(String name) {}
```

An interface for a service to greet the <code>Person</code>.

```java
public interface Greeter {

  void greet(Person person);
}
```

The implementation that prints a greeting to the standard out.

```java
public class SystemOutGreeter implements Greeter {

  public void greet(Person person) {
    System.out.printf("Greetings %s!%n", person.name());
  }
}
```

### Using Property files for configuration

Now lets step into a time machine and go back 15 years to the inception of Spring. A day and age before XML and annotations but with the existence of the good old properties file. Create an `application-context.properties` and lets use that to bootstrap our `ApplicationContext`. Yes still today with Spring 6 we can use property files to create an `ApplicationContext`. 

**NOTE:** As of Spring Framework 5.3 this feature has been marked as deprecated and might (and probably will be) removed in some future version of Spring. 

```
person.(class)=biz.deinum.blog.context.springcontexts.Person
person.$0=Marten Deinum

greeter.(class)=biz.deinum.blog.context.springcontexts.SystemOutGreeter
```

Now that we have all the moving parts, for now, lets create an `ApplicationContext` with it.

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    try (var contextFromProperties =new GenericApplicationContext()) {

      var reader = new PropertiesBeanDefinitionReader(contextFromProperties);
      reader.loadBeanDefinitions("classpath:application-context.properties");
      contextFromProperties.refresh();

      doGreeting(contextFromProperties);
  }   
}

  private static void doGreeting(ApplicationContext ctx) {
    var greeter = ctx.getBean(Greeter.class);
    var person = ctx.getBean(Person.class);
    greeter.greet(person);
  }
}
```

So what does all of this do. First we need an `ApplicationContext` and for this we use the `GenericApplicationContext` class. Then we need something to load our `BeanDefinition`s with, which is where the <code>BeanDefinitionReader</code> comes into play. As we want to use a property file specifically the [`PropertiesBeanDefinitionReader`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/support/PropertiesBeanDefinitionReader.html). The `PropertiesBeanDefinitionReader` takes a property file and converts that into `BeanDefinition` instances. In our case it will create 2 `BeanDefinition` instances 1 for the `Person` and another for the `SystemOutGreeter`. The special `.(class)` notation specifies what the type of the bean is going to be (there are more see the javadoc for that). There is also `person.$0=Marten Deinum` this will set the first constructor argument of the `Person` to the specified value. It will use the data binding support in Spring for that which ultimately comes done to using reflection.

The loaded `BeanDefinition`s are added to the `BeanFactory` (the `ApplicationContext` is a `BeanFactory` on steroids) so that it can create the actual bean instances.

Before we can use an `ApplicationContext` we have to call the `refresh()` method on it. **NOTE:** in most cases this is done automatically!

After that we get the `Greeter` and `Person` from the `ApplicationContext` and print the greeting.

Output: `Greetings, Marten Deinum!`

What ultimately happens to create the `person` and `greeter` beans (through reflection as mentioned earlier) is the following:

```java
var person = new Person("Marten Deinum");
```

and

```java
var greeter = new SystemOutGreeter();
```

### Using XML for configuration

Now lets do the same with XML. Create an `applicationContext.xml` to configure a `Person` and `SystemOutGreeter`.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="greeter" class="biz.deinum.blog.context.springcontexts.SystemOutGreeter" />

  <bean id="person" class="biz.deinum.blog.context.springcontexts.Person">
    <constructor-arg name="name" value="Marten Deinum" />
  </bean>
</beans>
```

The XML is more verbose then the property file. We have a `bean`element which in turn has a `class` attribute to specify the type of the bean. Inside the `bean` element we can specify `property` or `constructor-arg` elements to specify which properties/constructor arguments to set, in our case we are setting the constructor argument `name` again to the given value.

To load this file we will need a `BeanDefinitionReader` which can load XML files, thus the `XmlBeanDefinitionReader`.

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    try (var contextFromXml = new GenericApplicationContext()) {}

      var reader = new XmlBeanDefinitionReader(contextFromXml);
      reader.loadBeanDefinitions("classpath:applicationContext.xml");
      contextFromXml.refresh();

      doGreeting(contextFromXml);
    }
  }

  private static void doGreeting(ApplicationContext ctx) {
    var greeter = ctx.getBean(Greeter.class);
    var person = ctx.getBean(Person.class);
    greeter.greet(person);
  }
}
```

The code is almost the same as the one with the property file, except for the highlighted lines which instead of loading property file now use a XML file for the configuration. The output is still the same.

The creation of the `person` and `greeter` is also still the same and again boils down to:

```java
var person = new Person("Marten Deinum");
```

and

```java
var greeter = new SystemOutGreeter();
```

However generally you will not be writing this code but instead you would be using the `ClassPathXmlApplicationContext` to do all this work for you.

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    try (var contextFromXml = new ClassPathXmlApplicationContext("applicationContext.xml")) {}
      doGreeting(contextFromXml);
  }    
 }
  ... // Method omitted
}
```

This code and the code above are basically identical in what happens internally, however this is a lot shorter.

### Using Java for Configuration
We could also use the Java language to express the configuration. In Spring a Java configuration class is a regular class annotated with `@Configuration` and containing methods annotated with `@Bean`.

```java
@Configuration
public class ApplicationContext {

  @Bean
  public Person person() {
    return new Person("Marten Deinum");
  }

  @Bean
  public Greeter greeter() {
    return new SystemOutGreeter();
  }
}
```

**NOTE:** The following code will **NOT** compile as it is using inaccessible classes from Spring it is merely here to show how it internally works in Spring (in a simplified form!).

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    try (var contextFromJava =new GenericApplicationContext()) {
      var reader =new ConfigurationClassBeanDefinitionReader(contextFromJava);
      reader.loadBeanDefinitions(Set.of(new ConfigurationClass(ApplicationConfig.class, "applicationContext")));
      contextFromJava.refresh();
      doGreeting(contextFromJava);
    }
  }

  private static void doGreeting(ApplicationContext ctx) {
    var greeter = ctx.getBean(Greeter.class);
    var person = ctx.getBean(Person.class);
    greeter.greet(person);
  }
}
```

This is more or less what happens when Spring finds a Java configuration class. However all of that is hidden inside the `AnnotationConfigApplicationContext` class.

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    try (var contextFromJava = new AnnotationConfigApplicationContext(ApplicationContext.class)) {
      doGreeting(contextFromJava);
    }
  }

  private static void doGreeting(ApplicationContext ctx) {
    var greeter = ctx.getBean(Greeter.class);
    var person = ctx.getBean(Person.class);
    greeter.greet(person);
  }
```

### Using Java with Functional Bean Registration for Configuration
Another option with Java based configuration is to use a functional style of registering beans. The `GenericApplicationContext` has a couple of `registerBean` methods which allow for a functional style of bean registration. Instead of writing a configuration class, xml or properties file the registration is done by calling the methods. Underneath it will still create a `BeanDefinition` and use the passed in `Supplier` as a factory method for the bean instance. 

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    try (var functionalContext = new GenericApplicationContext()) {
      functionalContext.registerBean("person", Person.class, () -> new Person("Marten Deinum"));
      functionalContext.registerBean("greeter", Greeter.class, SystemOutGreeter::new);
      functionalContext.refresh();
      doGreeting(functionalContext);
    }
  }

  private static void doGreeting(ApplicationContext ctx) {
    var greeter = ctx.getBean(Greeter.class);
    var person = ctx.getBean(Person.class);
    greeter.greet(person);
  }
}
```

The code above will register 2 beans, just as the preceeding samples. One `Person` and one `Greeter` but in a functional approach. What happens is that Spring will create a specialized `BeanDefinition` the `ClassDerivedBeanDefinition` which gets the `Supplier` passed in as the factory (or as internally called the instance supplier)

### Conclusion

Now what is the conclusion when seeing 4 different configuration mechanisms. The first 2 mechanisms translate into the same as what is happening inside an `@Bean` annotated method, the latter is a more functional and plain Java approach

Based on the `.(class)` for property file based and `class` attribute for XML based configuration you can conclude that Spring does `new <class-value-here>()` itself. It then will call `set<property-name-here>(<property-value-here>)` just as in Java based configuration, or passing constructor arguments when using `<constructor-arg>` in the XML configuration).

|Properties | XML | Java | Java (Functional) |
|-----------|-----|------| ----------------- |
|`.(class)=Person` | `class=Person` | Name of the class | Passed in name or name of class when `null` |
|`person.$0=Marten Deinum` | `<constructor-arg name="name" value="Marten Deinum"` | `new Person("Marten Deinum")` | `() -> new Person("Marten Deinum")`

For those interested the source can be found on [GitHub](https://github.com/mdeinum/blog-spring-contexts).
