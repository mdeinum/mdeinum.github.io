---
layout: post
title: Delay startup of your Spring Boot application until your DB is up.
date: 2020-06-30
categories:
- Java
- Spring
tags:
- configuration
- micro-services
published: true
---

When using [Spring Boot](https://spring.io/projects/spring-boot) or just plain [Spring Framework](https://spring.io/projects/spring-framework) it might be that you want to delay the startup of your application until a proper connection to the database can be made. This might be even more the case when using container technologies, like [Docker](https://www.docker.com/).

I recently came across the [DatabaseStartupValidator](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/support/DatabaseStartupValidator.html) which has been part of the Spring Framework since 2003!. This little gem will delay the further startup of your application until a connection to the database can be made. It, by default, will try every second to connect to the DB and simply catches the exception. It will try for 60 seconds and after that will fail if no connection can be made (all of these properties are configurable).

To use you simply need to declare a bean and inject the datasource (see [Listing 1](#listing1)). You define a validation query (as of Spring 5.3 it will use the JDBC 4, `isValid` method by default!). Spring Boot comes with a handy [enum](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/DatabaseDriver.java) that already contains default validation queries for a range of supported database (used by the health check in Spring Boot Actuator).

<a id="listing1"></a>
```java
@SpringBootApplication
public class DatabaseUpApplication {

    public static void main(String[] args) {
        SpringApplication.run(DatabaseUpApplication.class, args);
    }

    @Bean
    public DatabaseStartupValidator databaseStartupValidator(DataSource dataSource) {
        var dsv = new DatabaseStartupValidator();
        dsv.setDataSource(dataSource);
        dsv.setValidationQuery(DatabaseDriver.POSTGRESQL.getValidationQuery());
        return dsv;
    }
}
```
#### Listing 1: Configuration of `DatabaseStartupValidator`

However just defining this bean might not be enough, a little more configuration is needed. Beans that depend on the `DataSource` like an `EntityManagerFactory`, `Flyway` or `JdbcTemplate` need to depend on the `DatabaseStartupValidator` as well. Now you could manually define all the beans and add an `@DependsOn` on those `@Bean` methods (you will loose some of the auto-configuration from Spring Boot that way) **or** you could use a [`BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html) to modify the beans. 

```java
@Bean
public static BeanFactoryPostProcessor dependsOnPostProcessor() {
    return bf -> {
        // Let beans that need the database depend on the DatabaseStartupValidator
        // like the JPA EntityManagerFactory or Flyway
        String[] flyway = bf.getBeanNamesForType(Flyway.class);
        Stream.of(flyway)
                .map(bf::getBeanDefinition)
                .forEach(it -> it.setDependsOn("databaseStartupValidator"));

        String[] jpa = bf.getBeanNamesForType(EntityManagerFactory.class);
        Stream.of(jpa)
                .map(bf::getBeanDefinition)
                .forEach(it -> it.setDependsOn("databaseStartupValidator"));
    };
}
```
#### Listing 2: `BeanFactoryPostProcessor` to declare depending beans

In this sample we add the `dependsOn` for the `Flyway` and `EntityManagerFactory` bean definitions (the recipe of how to create a bean not the actual bean itself). This will delay the execution of `Flyway` and the `EntityManagerFactory` until the `DatabaseStartupValidator` is fully initialized. 

Now when starting the code and delaying the start of the database the startup of the application will stall until it can reach the database. One can simulate this by starting the application and run the database in a Docker container and delay the start. The output would show something like the following:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.1.RELEASE)

2020-06-29 09:42:54.081  INFO 32104 --- [           main] b.d.databaseup.DatabaseUpApplication     : Starting DatabaseUpApplication on iMac-van-Marten.local with PID 32104 (/Users/marten/Repositories/database-up/target/classes started by marten in /Users/marten/Repositories/database-up)
2020-06-29 09:42:54.084  INFO 32104 --- [           main] b.d.databaseup.DatabaseUpApplication     : No active profile set, falling back to default profiles: default
2020-06-29 09:42:54.840  INFO 32104 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data JPA repositories in DEFERRED mode.
2020-06-29 09:42:54.887  INFO 32104 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 40ms. Found 1 JPA repository interfaces.
2020-06-29 09:42:55.412  INFO 32104 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-06-29 09:42:55.418  INFO 32104 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-06-29 09:42:55.419  INFO 32104 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.36]
2020-06-29 09:42:55.505  INFO 32104 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-06-29 09:42:55.505  INFO 32104 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1385 ms
2020-06-29 09:42:56.590  INFO 32104 --- [           main] o.s.j.support.DatabaseStartupValidator   : Database has not started up yet - retrying in 1 seconds (timeout in 58.979 seconds)
2020-06-29 09:42:58.602  INFO 32104 --- [           main] o.s.j.support.DatabaseStartupValidator   : Database has not started up yet - retrying in 1 seconds (timeout in 56.967 seconds)
2020-06-29 09:42:59.671  INFO 32104 --- [           main] o.s.j.support.DatabaseStartupValidator   : Database startup detected after 4.102 seconds
2020-06-29 09:42:59.723  INFO 32104 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-06-29 09:42:59.759  INFO 32104 --- [         task-1] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2020-06-29 09:42:59.791  INFO 32104 --- [         task-1] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.4.17.Final
2020-06-29 09:42:59.895  INFO 32104 --- [         task-1] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.0.Final}
2020-06-29 09:42:59.958  INFO 32104 --- [         task-1] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.PostgreSQL10Dialect
2020-06-29 09:43:00.067  INFO 32104 --- [           main] o.f.c.internal.license.VersionPrinter    : Flyway Community Edition 6.4.4 by Redgate
2020-06-29 09:43:00.076  INFO 32104 --- [           main] o.f.c.internal.database.DatabaseFactory  : Database: jdbc:postgresql://localhost:5432/sample (PostgreSQL 12.3)
2020-06-29 09:43:00.125  INFO 32104 --- [           main] o.f.core.internal.command.DbValidate     : Successfully validated 1 migration (execution time 00:00.022s)
2020-06-29 09:43:00.140  INFO 32104 --- [           main] o.f.core.internal.command.DbMigrate      : Current version of schema "public": 1
2020-06-29 09:43:00.142  INFO 32104 --- [           main] o.f.core.internal.command.DbMigrate      : Schema "public" is up to date. No migration necessary.
2020-06-29 09:43:00.213  INFO 32104 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-06-29 09:43:00.214  INFO 32104 --- [           main] DeferredRepositoryInitializationListener : Triggering deferred initialization of Spring Data repositories…
2020-06-29 09:43:00.335  INFO 32104 --- [         task-1] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2020-06-29 09:43:00.343  INFO 32104 --- [         task-1] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2020-06-29 09:43:00.464  INFO 32104 --- [           main] DeferredRepositoryInitializationListener : Spring Data repositories initialized!
2020-06-29 09:43:00.473  INFO 32104 --- [           main] b.d.databaseup.DatabaseUpApplication     : Started DatabaseUpApplication in 6.672 seconds (JVM running for 7.269)
```

The source code for this blog can be found on [GitHub](https://github.com/mdeinum/blog-database-up).