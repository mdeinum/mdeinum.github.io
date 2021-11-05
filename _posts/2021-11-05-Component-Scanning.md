---
layout: post
title: The power of component scanning in Spring
date: 2021-11-05
categories:
- Java
- Spring
tags:
- configuration
published: true
gh-repo: mdeinum/blog-component-scanning
gh-badge: star	
---

When you use Spring (Boot) you inevitably come around or will use the [`@ComponentScan`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html) annotation to automatically detect your classes (meta) annotated with [`@Component`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html). 

Lets take a look at a basic Spring Boot sample that will detect our `@Component` annotated classes. Lets write an application that uses a `Printer` to print out a message. 

```java
package biz.deinum.blog.blogcomponentscanning.printer;

public interface Printer {

    void print(String msg);
}
```

A basis class printing to `System.out` and the class is annotated with `@Component` so that component-scanning can pick this class up. 

```java
package biz.deinum.blog.blogcomponentscanning.printer;

import org.springframework.stereotype.Component;

@Component
class SystemOutPrinter implements Printer {

    @Override
    public void print(String msg) {
        System.out.printf("Message: %s%n", msg);
    }
}
```

Finally we will use Spring Boot to bootstrap our application and print out a message to all available `Printer` instances. We will use an [`ApplicationRunner`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationRunner.html) to print **Hello World!* to all configured `Printer` instances. For the time being this will only be a single one. 

```java
package biz.deinum.blog.blogcomponentscanning;

import java.util.List;

import biz.deinum.blog.blogcomponentscanning.printer.Printer;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan(basePackages = "biz.deinum.blog.blogcomponentscanning.printer")
public class BlogComponentScanningApplication {

    public static void main(String[] args) {
        SpringApplication.run(BlogComponentScanningApplication.class, args);
    }

    @Bean
    public ApplicationRunner printHelloWorl(List<Printer> ps) {
        return args -> ps.forEach(p -> p.print("Hello World!"));
    }
}
```

**NOTE:** While it isn't necessary to add the `@ComponentScan` annotation to this Spring Boot application, as this is a blog about component-scanning it was added for clarity and has a purpose in this blog!

When the application is run it will output *Message: Hello World!* to the console. Not the most daring application but it shows component-scanning in action. 

First lets take a bit of a closer look at the `@ComponentScan` annotation. It has several attributes that can be set, most commonly used are the `basePackages`, `basePackageClasses` and `useDefaultFilters` and maybe some of you used `lazyInit` and `scopedProxy` as well. However there are a few more and interestingly those are quite powerful, the annotation also takes `includeFilters` and `excludeFilters`. These attributes take a array of [`@Filter`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.Filter.html)

With this `@Filter` annotation you can control what is included (or excluded) from being considered as a component. By default a filter with the [`FilterType.ANNOTATION`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/FilterType.html) is used. However there are more modes to use for scanning for components. The `@FilterType` annotation allows for 5 different values. 

| Value | Description |
| ----- | ----------- |
| `ANNOTATION` | Filter candidates marked with a given annotation. |
| `ASPECTJ` | Filter candidates matching a given AspectJ type pattern expression. |
| `ASSIGNABLE_TYPE` | Filter candidates assignable to a given type. |
| `CUSTOM` | Filter candidates using a given custom [`TypeFilter`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/type/filter/TypeFilter.html) implementation. |
| `REGEX` | Filter candidates matching a given regex pattern. |

All of these can be used to detect classes to be considered a component and they don't have to be annotated with `@Component`. To illustrate the power of this lets add a new `Printer` instance to our application and **don't** annotate that with `@Component` but lets see how we can use the other modes for filtering to include them nonetheless. 

```java
package biz.deinum.blog.blogcomponentscanning.printer;

class SystemErrPrinter implements Printer {

    @Override
    public void print(String msg) {
        System.err.printf("Message: %s%n", msg);
    }
}
```

## Use `ASSIGNABLE_TYPE`

When you run the application, without modification, you will see the same output as before. Not lets modify the `@ComponentScan` annotation and lets use the `ASSIGNABLE_TYPE` filter to detect the implementations. 

```java
package biz.deinum.blog.blogcomponentscanning;

import java.util.List;

import biz.deinum.blog.blogcomponentscanning.printer.Printer;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;

@SpringBootApplication
@ComponentScan(
    basePackages = "biz.deinum.blog.blogcomponentscanning.printer",
    useDefaultFilters = false,
    includeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = Printer.class)
})
public class BlogComponentScanningApplication {

    public static void main(String[] args) {
        SpringApplication.run(BlogComponentScanningApplication.class, args);
    }

    @Bean
    public ApplicationRunner printHelloWorl(List<Printer> ps) {
        return args -> ps.forEach(p -> p.print("Hello World!"));
    }
}
```

The application will now detect classes that implement the `Printer` interface in the package `biz.deinum.blog.blogcomponentscanning.printer`. The `useDefaultFilters = false` has been added so that only the explicitly declared `includeFilters` are taking into account when detecting classes. This wasn't strictly necessary but was added to show the power of using `ASSIGNABLE_TYPE`. 

## Use `ASPECTJ`

With the `FilterType.ASPECTJ` one can write an AspectJ type pattern expression (the same used in the `@Pointcut` annotation when writing Aspects!) to match types. This pattern is different from a regular expression but very powerful to use. 

```java
package biz.deinum.blog.blogcomponentscanning;

import java.util.List;

import biz.deinum.blog.blogcomponentscanning.printer.Printer;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;

@SpringBootApplication
@ComponentScan(
    basePackages = "biz.deinum.blog.blogcomponentscanning.printer",
    useDefaultFilters = false,
    includeFilters = {
        @ComponentScan.Filter(type = FilterType.ASPECTJ, pattern = "biz.deinum.blog.blogcomponentscanning.printer.Printer+")
})
public class BlogComponentScanningApplication {

    public static void main(String[] args) {
        SpringApplication.run(BlogComponentScanningApplication.class, args);
    }

    @Bean
    public ApplicationRunner printHelloWorl(List<Printer> ps) {
        return args -> ps.forEach(p -> p.print("Hello World!"));
    }
}
```

The AspectJ expression must be given in the `pattern` attribute. The expression written here means basically detect all classes that implement the `biz.deinum.blog.blogcomponentscanning.printer.Printer` interface. You can make the expressions as complex as you want or as simple as you want. For instance `biz.deinum.blog.blogcomponentscanning.printer.*Printer` would detect all classes in that package having a name ending with `Printer`. In this case that would still work as our implementations all end with that suffix. 

One drawback with this approach is that one needs AspectJ on the classpath to parse the expressions, now in most Spring Boot applications this isn't an issue, however it might be something to think about. 

## Use `REGEX`
As with the previous expression based one one can also use a Regular Expression to detect classes. Now this will only work with matches based on name, not as with the AspectJ one with types. 

```java
package biz.deinum.blog.blogcomponentscanning;

import java.util.List;

import biz.deinum.blog.blogcomponentscanning.printer.Printer;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;

@SpringBootApplication
@ComponentScan(
    basePackages = "biz.deinum.blog.blogcomponentscanning.printer",
    useDefaultFilters = false,
    includeFilters = {
        @ComponentScan.Filter(type = FilterType.REGEX, pattern = ".*Printer")
    })
public class BlogComponentScanningApplication {

    public static void main(String[] args) {
        SpringApplication.run(BlogComponentScanningApplication.class, args);
    }

    @Bean
    public ApplicationRunner printHelloWorl(List<Printer> ps) {
        return args -> ps.forEach(p -> p.print("Hello World!"));
    }
}
```

The given expression to use has to be specified in the `pattern` attribute of the `@Filter` annotation. In this case the expression `.*Printer` indicates classname ending with `Printer` are to be included. You can make the expressions as complex or simple as one want to detect classes with a given name. 

## Use `CUSTOM`

You can also write your own extensions to these filters use the `FilterType.CUSTOM`. For this one needs to implement a `TypeFilter` to implement the actual logic for detecting the classes. In fact all other `FilterType` options are actually implementations of a `FilterType` as well, only they are predefined. 

```java
package biz.deinum.blog.blogcomponentscanning;

import biz.deinum.blog.blogcomponentscanning.printer.Printer;

import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;
import org.springframework.util.ClassUtils;

public class PrinterTypeFilter implements TypeFilter {

    @Override
    public boolean match(MetadataReader mr, MetadataReaderFactory mrf) {
        var cm = metadataReader.getClassMetadata();
        var cn = cm.getClassName();
        try {
            return Printer.class.isAssignableFrom(ClassUtils.forName(cn, PrinterTypeFilter.class.getClassLoader()));
        } catch (ClassNotFoundException cnfe) {
            throw new IllegalStateException(cnfe);
        }
    }
}
```
The `PrinterTypeFilter` obtains the name of the class that has been detected using the `MetadataReader` which contains the metadata of the current class that has been detected/read. The class is then loaded and checked if it is assignable to the `Printer` interface, if so it will return `true` else `false`. Last part is to instruct Spring to use this custom `TypeFilter` for detecting classes. 

```java
package biz.deinum.blog.blogcomponentscanning;

import java.util.List;

import biz.deinum.blog.blogcomponentscanning.printer.Printer;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;

@SpringBootApplication
@ComponentScan(
    basePackages = "biz.deinum.blog.blogcomponentscanning.printer",
    useDefaultFilters = false,
    includeFilters = {
        @ComponentScan.Filter(type = FilterType.CUSTOM, classes = { PrinterTypeFilter.class})
    })

public class BlogComponentScanningApplication {

    public static void main(String[] args) {
        SpringApplication.run(BlogComponentScanningApplication.class, args);
    }

    @Bean
    public ApplicationRunner printHelloWorl(List<Printer> ps) {
        return args -> ps.forEach(p -> p.print("Hello World!"));
    }
}
```

The `type` attribute has been set to `FilterType.CUSTOM`, the `classes` attribute points to the `PrinterTypeFilter` class (this attribute can include multiple type filters at once which are used in an **or** fashion so if one of the filters matches the class will be included.). When a class is detected its metadata will be passed on to the `PrinterTypeFilter` which will determine based on java logic if it should be included. 

## Use `ANNOTATION` with a custom annotation
Finally you can even use the `FilterType.ANNOTATION` to detect classes annotated with your own custom annotation (which doesn't have to be meta annotated with `@Component`). You could create, for instance, a `@PrintingComponent` annotation and have that automatically detected. 

```java
package biz.deinum.blog.blogcomponentscanning;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface PrintingComponent {}
```

This is a basic annotation, which can be put on a type and is retained on class files at runtime. This is needed so that Spring can detect the annotation. 

Now modify your `SystemOutPrinter` and `SystemErrPrinter` to include this annotation on the type and change the `@ComponentScan` to pickup classes with our own annotation. 
```java
package biz.deinum.blog.blogcomponentscanning;

import java.util.List;

import biz.deinum.blog.blogcomponentscanning.printer.Printer;

import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;

@SpringBootApplication
@ComponentScan(
    basePackages = "biz.deinum.blog.blogcomponentscanning.printer",
    useDefaultFilters = false,
    includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = { PrintingComponent.class})
    })
public class BlogComponentScanningApplication {

    public static void main(String[] args) {
        SpringApplication.run(BlogComponentScanningApplication.class, args);
    }

    @Bean
    public ApplicationRunner printHelloWorl(List<Printer> ps) {
        return args -> ps.forEach(p -> p.print("Hello World!"));
    }
}

```
The `type` attribute has been set to `FilterType.ANNOTATION` the `classes` attribute takes the annotations to detect (here it is only our `PrintingComponent` annotation created earlier). When running this there will still be a message printed to both the `System.out` and `System.err` indicating the components have been picked up using our own custom annotation. 

## Conclusion
We can conclude that the component-scanning infrastructure available in Spring is very powerful and far more flexible than one might consider. At first glance it might appear that it can only be used to detect classes (meta) annotated with `@Component`. However if you look closer at the documentation and the options mentioned in there you will see it is actually a very powerful mechanism for detecting classes based on multiple options. 

Use what you need regular expression, Aspect type expressions, assignable type (with either an interface or (base) class), write a custom annotation to detect or go all the way writing your own `FilterType`. A lot of options with each its own use-cases. Use them wisely and to your benefit.  
