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

By default the [`@PropertySource`][1] annotation cannot be used with YAML files, and this is also the standard [answer][6] given when asked on for instance StackOverflow (even by me). However as of Spring 4.3 it is possible to specify a [`PropertySourceFactory`][3] which, as the name implies is a factory for a `PropertySource` the default implementation is the [`DefaultPropertySourceFactory`][4].

```java
package biz.deinum.blog.yaml;

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

    private Properties loadYamlIntoProperties(EncodedResource resource) {
        YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
        factory.setResources(resource.getResource());
        factory.afterPropertiesSet();
        return factory.getObject();
    }
}
```

**NOTE:** To load YAML it is required that SnakeYAML 1.18 or higher is on the classpath!

The `PropertySourceFactory` interface has a single method, `createPropertySource` that needs to be implemented. The `YamlPropertySourceFactory` shown above will load a YAML file into a `java.util.Properties` object and wrap that in a `PropertiesPropertySource` (one of the `PropertySource` implementations). To load the YAML file the [`YamlPropertiesFactoryBean`][5] can be used. A `PropertySource` is required to have a name and according to the contract it can be either given by the `name` attribute on the `@PropertySource` annotation or it will be derived from the `Resource.getDescription` method.

Next the YAML file, `blog.yaml` which we want to load:

```yaml
foo:
  bar: baz
```

Now that there is `PropertySourceFactory` that can load YAML an `@PropertySource` annotation that will use this `PropertySourceFactory` can be added to the configuration.

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
. Print the name and type of all the `PropertySource` instances created for this `ApplicationContext`
. Get the value from `foo.bar` which comes from the `blog.yaml` file

The output should be something like the image below:

![Output of using YamlPropertySourceFactory](/img/yaml1.png){:class="img-responsive"}

[1]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/PropertySource.html
[2]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/PropertySource.html
[3]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/PropertySourceFactory.html
[4]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/DefaultPropertySourceFactory.html
[5]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/YamlPropertiesFactoryBean.html
[6]: https://stackoverflow.com/questions/43020491/spring-boot-external-configuration-of-property-file
