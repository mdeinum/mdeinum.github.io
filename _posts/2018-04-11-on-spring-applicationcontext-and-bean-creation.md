---
layout: post
title: "On Spring ApplicationContext and Bean Creation"
date: 2018-04-12
tags: [spring]
published: true
---

There appears to be a lot of confusion in the community on how to expression bean configuration or rather how to convert bean configuration from XML to Java (or Groovy, or Kotlin, or...) as it appears as if the different ways of configuration are completely different things.

But in fact they are more or less the same it is **configuration expressed in a format (Java, XML, Properties) to instruct the Spring `ApplicationContext` (well actually the `BeanFactory`) to create beans based on those definitions** (aka [`BeanDefinition`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanDefinition.html)). A `BeanDefinition` is basically the recipe on how to prepare/create a bean.

Lets create a small sample using a few classes and lets take a look under the hood on what is happening with those class.

```java
public class Person {

  private String name;

  public void setName(String name) {
    this.name=name;
  }

  public String getName() {
    return this.name;
  }
}
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
    System.out.printf("Greetings %s!%n", person.getName());
  }
}
```

### Using Property files for configuration

Now lets step into a time machine and go back 15 years to the inception of Spring. A day and age before XML and annotations but with the existence of the good old properties file. Create an `application-context.properties` and lets use that to bootstrap our `ApplicationContext`. Yes still today with Spring 5 we can use property files to create an `ApplicationContext`. It is even actively used inside the framework by classes such as [`ResourceBundleViewResolver`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/view/ResourceBundleViewResolver.html)

```
person.(class)=biz.deinum.blog.context.springcontexts.Person
person.name=Marten Deinum

greeter.(class)=biz.deinum.blog.context.springcontexts.SystemOutGreeter
```

Now that we have all the moving parts, for now, lets create an `ApplicationContext` with it.

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    GenericApplicationContext contextFromProperties =
      new GenericApplicationContext();

    BeanDefinitionReader reader =
      new PropertiesBeanDefinitionReader(contextFromProperties);
    reader.loadBeanDefinitions("classpath:application-context.properties");
    contextFromProperties.refresh();

    doGreeting(contextFromProperties);

    contextFromProperties.stop();
  }

  private static void doGreeting(ApplicationContext ctx) {
    Greeter greeter = ctx.getBean(Greeter.class);
    Person person = ctx.getBean(Person.class);
    greeter.greet(person);
  }
}
```

So what does all of this do. First we need an `ApplicationContext` and for this we can use the `GenericApplicationContext` class. Then we need something to load our `BeanDefinition`s with, which is where the <code>BeanDefinitionReader</code> comes into play. As we want to use a property file specifically the [`PropertiesBeanDefinitionReader`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/support/PropertiesBeanDefinitionReader.html). The `PropertiesBeanDefinitionReader` takes a property file and converts that into `BeanDefinition` instances. In our case it will create 2 `BeanDefinition` instances 1 for the `Person` and another for the `SystemOutGreeter`. The special `.(class)` notation specifies what the type of the bean is going to be (there are more see the javadoc for that). There is also `person.name=Marten Deinum` this will set the `name` property of the `Person` to the specified value. It will use the data binding support in Spring for that which ultimately comes done to using reflection.



The loaded `BeanDefinition`s are added to the `BeanFactory` (the `ApplicationContext` is a `BeanFactory` on steroids) so that it can create the actual bean instances.

Before we can use an `ApplicationContext` we have to call the `refresh()` method on it. **NOTE:** in most cases this is done automatically!

After that we get the `Greeter` and `Person` from the `ApplicationContext` and print the greeting.

Output: `Greetings, Marten Deinum!`

What ultimately happens to create the `person` and `greeter` beans (through reflection as mentioned earlier) is the following:

```java
Person person = new Person();
person.setName("Marten Deinum");
```

and

```java
Greeter greeter = new SystemOutGreeter();
```

### Using XML for configuration

Now lets do the same with XML. Create an `applicationContext.xml` to configure a `Person` and `SystemOutGreeter`.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="greeter" class="biz.deinum.blog.context.springcontexts.SystemOutGreeter" />

  <bean id="person" class="biz.deinum.blog.context.springcontexts.Person">
    <property name="name" value="Marten Deinum" />
  </bean>
</beans>
```

The XML is more verbose then the property file. We have a `bean`element which in turn has a `class` attribute to specify the type of the bean. Inside the `bean` element we can specify `property` elements to specify which properties to set, in our case we are setting the `name` property again to the given value.

To load this file we will need a `BeanDefinitionReader` which can load XML files, thus the `XmlBeanDefinitionReader`.

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    GenericApplicationContext contextFromXml =
      new GenericApplicationContext();

    BeanDefinitionReader reader =
      new XmlBeanDefinitionReader(contextFromXml);
    reader.loadBeanDefinitions("classpath:applicationContext.xml");
    contextFromXml.refresh();

    doGreeting(contextFromXml);

    contextFromXml.stop();
  }

  private static void doGreeting(ApplicationContext ctx) {
    Greeter greeter = ctx.getBean(Greeter.class);
    Person person = ctx.getBean(Person.class);
    greeter.greet(person);
  }
}
```

The code is more or less the same as the one with the property file, except for the highlighted lines which instead of loading property file now use a XML file for the configuration. The output is still the same.

The creation of the `person` and `greeter` is also still the same and again boils down to:

```java
Person person = new Person();
person.setName("Marten Deinum");
```

and

```java
Greeter greeter = new SystemOutGreeter();
```

However generally you will not be writing this code but instead you would be using the `ClassPathXmlApplicationContext` to do all this work for you.

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    ConfigurableApplicationContext contextFromXml =
      new ClassPathXmlApplicationContext("applicationContext.xml");

    doGreeting(contextFromXml);

    contextFromXml.stop();
 }
  ... // Method omitted
}
```

This code and the code above are basically identical in what happens internally, however this is a lot shorter.

### Using Java for Configuration
Finally we could use the Java language to express the configuration. In Spring a Java configuration class is a regular class annotated with `@Configuration` and containing methods annotated with `@Bean`.

```java
@Configuration
public class ApplicationContext {

  @Bean
  public Person person() {
    Person person = new Person();
    person.setName("Marten Deinum");
    return person;
  }

  @Bean
  public SystemOutGreeter greeter() {
    return new SystemOutGreeter();
  }
}
```

**NOTE:** The following code will **NOT** compile as it is using inaccessible classes from Spring it is merely here to show how it internally works in Spring (in a simplified form!).

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    GenericApplicationContext contextFromJava =
      new GenericApplicationContext();

    ConfigurationClassBeanDefinitionReader reader =
      new ConfigurationClassBeanDefinitionReader(contextFromJava);
    reader.loadBeanDefinitions(Collections.singletonSet(
      new ConfigurationClass(ApplicationContext.class, "applicationContext")));
    contextFromJava.refresh();

    doGreeting(contextFromXml);

    contextFromJava.stop();
  }

  private static void doGreeting(ApplicationContext ctx) {
    Greeter greeter = ctx.getBean(Greeter.class);
    Person person = ctx.getBean(Person.class);
    greeter.greet(person);
  }
}
```

This is more or less what happens when Spring finds a Java configuration class. However all of that is hidden inside thte `AnnotationConfigApplicationContext` class.

```java
public class SpringContextsApplication {

  public static void main(String[] args) throws Exception {
    AnnotationConfigApplicationContext contextFromJava =
      new AnnotationConfigApplicationContext(ApplicationContext.class);

    doGreeting(contextFromXml);

    contextFromJava.stop();
  }

  private static void doGreeting(ApplicationContext ctx) {
    Greeter greeter = ctx.getBean(Greeter.class);
    Person person = ctx.getBean(Person.class);
    greeter.greet(person);
  }
```

### Conclusion

Now what is the conclusion when seeing all 3 different configuration mechanisms. The first 2 mechanisms translate into the same as what is happening inside an `@Bean` annotated method.

Based on the `.(class)` for property file based and `class` attribute for XML based configuration you can conclude that Spring does `new <class-value-here>()` itself. It then will call `set<property-name-here>(<property-value-here>)` just as in Java based configuration, or passing constructor arguments when using `<constructor-arg>` in the XML configuration).

Properties | XML attribute | Java Configuration |
-----------|-----|------|
`.(class)=Person` | `class=Person` | Name of the class
`person.name=Marten Deinum` | `<property name="name" value="Marten Deinum"` | `setName("Marten Deinum")`
[Translation Table]
