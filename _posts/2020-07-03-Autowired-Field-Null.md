---
layout: post
title: Why are my autowired fields null
date: 2020-07-03
last_modified_at: 2020-07-22
categories:
- Java
- Spring
tags:
- configuration
- dependency injection
- autowiring
published: true
gh-repo: mdeinum/blog-autowiring-null
gh-badge: star	
---

A commonly asked question on sites like StackOverflow is ["Why is the field i'm autowiring `null`"](https://stackoverflow.com/questions/19896870/why-is-my-spring-autowired-field-null). When this happens it generally is an error of the user as an autowired field in Spring cannot be `null`. At startup Spring will try to satisfy all the dependencies of a bean, if the depenencies cannot be satisfied starting the application will stop with an `UnsatisfiedDependencyException`.

The following `HelloWorldService` is what this blog post will be using (or a slight variation thereof). It takes a name and uses a `java.io.Writer` to write a hello message. This `java.io.Writer` needs to be introduced as a dependency. 

```java
package biz.deinum.blog.autowiring.unsatisfied;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.Writer;

@Service
public class HelloWorldService {

    @Autowired
    private Writer writer;

    public void sayHello(String name) throws IOException {
        writer.write("Hello " + name);
    }
}
```

The following configuration enables component-scanning but does not provide a `java.io.Writer` dependency. 

```java
package biz.deinum.blog.autowiring.unsatisfied;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class HelloWorldConfig {}
```

Now to load all of this use the following clas

```java
package biz.deinum.blog.autowiring.unsatisfied;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class HelloWorldUnsatisfied {

    public static void main(String[] args) throws Exception {
        var ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class);
    }
}
```
When running this class Spring will prevent it from properly starting due to not being able to find a `java.io.Writer` and will throw an `UnsatisfiedDependencyException`.

```
WARNING: Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'helloWorldService': Unsatisfied dependency expressed through field 'writer'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'java.io.Writer' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
Exception in thread "main" org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'helloWorldService': Unsatisfied dependency expressed through field 'writer'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'java.io.Writer' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:643)
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:130)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:399)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1422)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:594)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:517)
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:323)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:226)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:321)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:893)
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:879)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:551)
	at org.springframework.context.annotation.AnnotationConfigApplicationContext.<init>(AnnotationConfigApplicationContext.java:89)
	at biz.deinum.blog.autowiring.unsatisfied.HelloWorldUnsatisfied.main(HelloWorldUnsatisfied.java:17)
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'java.io.Writer' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.raiseNoMatchingBeanFound(DefaultListableBeanFactory.java:1714)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1270)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1224)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:640)
	... 14 more
```

If the application starts and your field **appears** to be `null` it generally due to on of the following issues:

1. [Using `@Autowired` on a `static` field](#static)
2. [Omitted `@Autowired` on a field](#omitted)
3. [Instance of bean not visible to Spring](#instance)
4. [Using AOP and invoking a `final`, `private` or default access method](#aop)
5. [Using XML and haven't enabled annotation processing](#xml)

<a href="#static"></a>
## Using `@Autowired` on a `static` field

Using the `HelloWorldService` below as a bean in a Spring application would fail, as dependency injection on `static` fields isn't supported. 

```java
package biz.deinum.blog.autowiring.statik;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.Writer;

@Service
public class HelloWorldService {

    @Autowired
    private static Writer writer;

    public void sayHello(String name) throws IOException {
        writer.write("Hello " + name);
    }
}
```

The following configuration will load the class and setup the needed `Writer` dependency. 

```java
package biz.deinum.blog.autowiring.statik;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import java.io.PrintWriter;
import java.io.Writer;

@Configuration
@ComponentScan
public class HelloWorldConfig {

    @Bean
    public Writer consoleWriter() {
        return new PrintWriter(System.out);
    }
}
```

The following class with a `main` method will load the `HelloWorldConfig`, obtain the `HelloWorldService` from the `ApplicationContext` and invoke the `sayHello` method. The result will be a `NullPointerException` as the field isn't autowired. 

```java
package biz.deinum.blog.autowiring.statik;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class HelloWorldStatic {

    public static void main(String[] args) throws Exception {

        var ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class);
        var service = ctx.getBean(HelloWorldService.class);
        service.sayHello("World"); // Throws NullPointerException due to injection on static field
    }
}
```

The solution, as often, depends, the most obvious one would be to remove the `static` keyword and it would work as it is now a regular field. However there might be a compeling reason to make the field `static`. If that is the case you can either use constructor, setter or method injection to set the `static` field. Although this should be considered a hack, imho, instead of a solution. 

<a href="#static"></a>
## Omitted `@Autowired` on a field

Using the `HelloWorldService` below as a bean in a Spring application would fail. As there is no `@Autowired` (or `@Inject` or `@Resource`) on the field, Spring doesn't know it needs to inject a dependency into the field. So the field remains `null`.

```java
package biz.deinum.blog.autowiring.statik;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.Writer;

@Service
public class HelloWorldService {

    private Writer writer;

    public void sayHello(String name) throws IOException {
        writer.write("Hello " + name);
    }
}
```

The following configuration will load the class and setup the needed `Writer` dependency. 

```java
package biz.deinum.blog.autowiring.statik;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import java.io.PrintWriter;
import java.io.Writer;

@Configuration
@ComponentScan
public class HelloWorldConfig {

    @Bean
    public Writer consoleWriter() {
        return new PrintWriter(System.out);
    }
}
```

The following class with a `main` method will load the `HelloWorldConfig`, obtain the `HelloWorldService` from the `ApplicationContext` and invoke the `sayHello` method. The result will be a `NullPointerException` as the field isn't autowired. 

```java
package biz.deinum.blog.autowiring.statik;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class HelloWorldStatic {

    public static void main(String[] args) throws Exception {

        var ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class);
        var service = ctx.getBean(HelloWorldService.class);
        service.sayHello("World"); // Throws NullPointerException due to no injection metadata present
    }
}
```

To fix you can add the `@Autowired` dependency to the field or even better use constructor based injection (which doesn't require an annotation!).



<a href="#instance"></a>
## Instance of bean not visible to Spring
For Spring to be able to do dependency injection it needs to know about the beans inside the `ApplicationContext`. If a bean is created outside of the context or not as bean Spring will not be able to do dependency injection. 

Taking the following `HelloWorldService`

```java
package biz.deinum.blog.autowiring.instance;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.Writer;

@Service
public class HelloWorldService {

    @Autowired
    private Writer writer;

    public void sayHello(String name) throws IOException {
        writer.write("Hello " + name);
    }
}
```

and the following starter

```java
package biz.deinum.blog.autowiring.instance;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class HelloWorldNewInstance {

    public static void main(String[] args) throws Exception {

        var ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class);
        var service = new HelloWorldService();
        service.sayHello("World");
    }
}
```

Will also result in a `NullPointerException`. This small sample is propably quite obvious as the new instance is created outside of the Spring `ApplicationContext`. Here you actually have 2 instances of the `HelloWorldService`, 1 inside the `ApplicationContext` and the one freshly constructed. Sometimes it might be more subtle and the bean creation happens inside an `@Bean` method in an `@Configuration` class. Like as part of tcreating a `FilterRegistrationBean` in Spring Boot or a configuration enhancement in Spring Security. 

```java
@Bean
public FilterRegistrationBean myFilter() {
  var registration = new FilterRegistrationBean(new MyFilter());
  return registration;
}
```

The configuration above creates a new instance of `MyFilter` but this isn't seen as a bean. If autowiring inside `MyFilter` is needed, it needs to have a dedicated `@Bean` method or be detected through component-scanning. 

<a href="#aop"></a>
## Using AOP and invoking a `final`, `private` or default access method

When using Spring AOP this by default uses proxies and generally there are no issues with that. However there are some things to understand about proxy based AOP. Only `public` and `protected` methods can be enhanched with this type op AOP. When using `private` or default access modifiers AOP won't be applied, the same applies to `final` methods or classes when using class-based proxies (the default in Spring Boot!). 

The aformentioned methods are an issue because when using proxy based AOP, spring will create an additional instance of the class which acts like, in this case, the `HelloWorldService`. This additional instance wraps the actual instance. When invoking a method on the class it will first call the different aspects, before passing it on to the actual instance. However as there is no way to proxy a `final`, `private` or default access method the method is called on the created proxy. This proxy class will not have the dependencies injected (as it doesn't need it).

The following aspect will add a before and after logging line upon invocation of a method

```java
package biz.deinum.blog.autowiring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* biz.deinum.blog..*.*(..))")
    public Object log(ProceedingJoinPoint pjp) throws Throwable{
        try {
            System.out.println("Before method: " + pjp.getSignature().getName());
            return pjp.proceed();
        } finally {
            System.out.println("After method: " + pjp.getSignature().getName());
        }
    }
}
```

Given the following `HelloWorldService` with a `final` method

```java
package biz.deinum.blog.autowiring.aop;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.Writer;

@Service
public class HelloWorldService {

    @Autowired
    private Writer writer;

    public final void sayHello(String name) throws IOException {
        writer.write("Hello " + name);
    }
}
```

The configuration needs to enable AspectJ proxy creation

```java
package biz.deinum.blog.autowiring.aop;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

import java.io.PrintWriter;
import java.io.Writer;

@Configuration
@ComponentScan
@EnableAspectJAutoProxy
public class HelloWorldConfig {

    @Bean
    public Writer consoleWriter() {
        return new PrintWriter(System.out);
    }

}
```

Finally to bootstrap all this

```java
package biz.deinum.blog.autowiring.aop;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class HelloWorldAop {

    public static void main(String[] args) throws Exception {

        var ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class);
        var service = ctx.getBean(HelloWorldService.class);
        service.sayHello("World"); 
    }
}
```

Again this will through a `NullPointerException` due to being unable to proxy the `final` method. 

The solution in this case is to remove the `final` modifier and it will run. Sometimes that might not be an option, introducing an interface and enable interface based proxies is another solution. 

The `private` and default access modifier is sometimes seen in `@Controller`/`@RestController` classes and looks something like this. 

```java
package biz.deinum.blog.autowiring.aop;

@RequestMapping
private String index() {
    return "index";
}
```

Currently that doesn't have to be an issue (it is with our `LoggingAspect`) but starts to be an issue when introducing things like Spring Security and the use of `@PreAuthorize`. Using `@PreAuthorize` also leads to a proxy being created for the given class, but the `private` method is now being invoked on the proxy instead of the actual object (just as with the `final` method mentioned earlier).

<a href="#xml"></a>
## Using XML and not enable annotation processing
This one is an oldie but still appears at times. Historically Spring uses XML files for its configuration and as of Spring 2.0  annotation based configuration was possible. However as XML was still the leading one the annotation processing had to be enabled. To enable annotation processing you need to add `<context:annotation-config />` or use `<context:component-scan />`. Without one of these annotations wouldn't be processed. 

```java
package biz.deinum.blog.autowiring.xml;

import org.springframework.beans.factory.annotation.Autowired;

import java.io.IOException;
import java.io.Writer;

public class HelloWorldService {

    @Autowired
    private Writer writer;

    public final void sayHello(String name) throws IOException {
        writer.write("Hello " + name);
    }
}
```

The service above has an `@Autowired` annotation but is not a component. Using an XML based application context one would need to create the bean and the needed dependencies.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:util="http://www.springframework.org/schema/util"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="helloWorldService" class="biz.deinum.blog.autowiring.xml.HelloWorldService" />

    <bean id="writer" class="java.io.PrintWriter">
        <constructor-arg>
            <util:constant static-field="java.lang.System.out" />
        </constructor-arg>
    </bean>

</beans>
```

This would create the 2 beans **but** not apply annotation based injection due to the absence of `<context:annotation-config />`. Loading this XML file and obtaining the service to execute the `sayHello` method would result, yet again, in a `NullPointerException`. 

```
package biz.deinum.blog.autowiring.xml;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class HelloWorldXml {

    public static void main(String[] args) throws Exception {

        var ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        var service = ctx.getBean(HelloWorldService.class);
        service.sayHello("World");
    }
}
```

# Summary

As a rule of thumb an autowired field in Spring cannot be `null`. If it looks like it is `null` the error is generally on the user side of things and can be tracked down to one of the following issues:
1. [Using `@Autowired` on a `static` field](#static)
2. [Omitted `@Autowired` on a field](#omitted)
3. [Instance of bean not visible to Spring](#instance)
4. [Using AOP and invoking a `final`, `private` or default access method](#aop)
5. [Using XML and haven't enabled annotation processing](#xml)

The code can be found on [GitHub](https://github.com/mdeinum/blog-autowiring-null.git).