---
layout: post
title: "Use @PropertySource with YAML files"
date: 2018-07-04
categories:
- Spring
tags:
- configuration
published: true
---

By default the [`@PropertySource`][1] annotation isn't usable with YAML files. This is also the standard [answer][6] given when asked on, for instance StackOverflow (even by me!).

But as of [Spring 4.3][7] it's possible to make it work. Spring 4.3 introduced the [`PropertySourceFactory`][3] interface. The `PropertySourceFactory` is a factory for a `PropertySource`. The default implementation used is the [`DefaultPropertySourceFactory`][4], which creates `ResourcePropertySource` instances.

Writing a custom implementation requires implementing a single method, createPropertySource. The custom implementation needs to do 2 things:

* Load the given resource into a java.util.Properties object
* Create a PropertySource wrapping the loaded properties

To load a YAML file Spring provides the [`YamlPropertiesFactoryBean`][5]. This class will load 1 or more files and convert it into a `java.util.Properties` object. Spring provides a [`PropertiesPropertySource`] which wraps a `java.util.Properties` object. Finally the name of the PropertySource is either given or derived. The derived name is the resource description, as mentioned in [the contract][8].  

```java
package biz.deinum.blog.yaml;

import java.io.FileNotFoundException;
import java.io.IOException;
import java.util.Properties;

import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.core.env.PropertiesPropertySource;
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.support.EncodedResource;
import org.springframework.core.io.support.PropertySourceFactory;
import org.springframework.lang.Nullable;

public class YamlPropertySourceFactory implements PropertySourceFactory {

    @Override
    public PropertySource<?> createPropertySource(@Nullable String name, EncodedResource resource) throws IOException {
        Properties propertiesFromYaml = loadYamlIntoProperties(resource);
        String sourceName = name != null ? name : resource.getResource().getFilename();
        return new PropertiesPropertySource(sourceName, propertiesFromYaml);
    }

    private Properties loadYamlIntoProperties(EncodedResource resource) throws FileNotFoundException {
        try {
            YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
            factory.setResources(resource.getResource());
            factory.afterPropertiesSet();
            return factory.getObject();
        } catch (IllegalStateException e) {
            // for ignoreResourceNotFound
            Throwable cause = e.getCause();
            if (cause instanceof FileNotFoundException)
                throw (FileNotFoundException) e.getCause();
            throw e;
        }
    }
}
```

**NOTE:** To load YAML it is required that SnakeYAML 1.18 or higher is on the classpath!

Next the YAML file, `blog.yaml` which we want to load:

```yaml
foo:
  bar: baz
```

The `@PropertySource` annotation has a `factory` attribute. This is the attribute used to specify which `PropertySourceFactory` to use. Here we give it the value `YamlPropertySourceFactory.class`. The value attribute contains the name of the YAML resource to load.

```java
package biz.deinum.blog.yaml;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.ConfigurableEnvironment;

@SpringBootApplication
@PropertySource(factory = YamlPropertySourceFactory.class, value = "classpath:blog.yaml")
public class YamlPropertysourceApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext ctx =
                SpringApplication.run(YamlPropertysourceApplication.class, args);

        ConfigurableEnvironment env = ctx.getEnvironment();
        env.getPropertySources()
                .forEach(ps -> System.out.println(ps.getName() + " : " + ps.getClass()));

        System.out.println("Value of `foo.bar` = " + env.getProperty("foo.bar"));
    }
}
```
**NOTE:** Although this sample uses Spring Boot this _isn't required_ it works with plain Spring (version 4.3 or up) as well.

When running this application it will
* Print the name and type of all available `PropertySource`s
* Get the value from `foo.bar` which comes from the `blog.yaml` file

The output should be something like the image below:

![Output of using YamlPropertySourceFactory](/img/yaml1.png){:class="img-responsive"}

[1]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/PropertySource.html
[2]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/PropertySource.html
[3]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/PropertySourceFactory.html
[4]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/DefaultPropertySourceFactory.html
[5]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/YamlPropertiesFactoryBean.html
[6]: https://stackoverflow.com/questions/43020491/spring-boot-external-configuration-of-property-file
[7]: https://jira.spring.io/browse/SPR-8963
[8]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/PropertySource.html#name--
