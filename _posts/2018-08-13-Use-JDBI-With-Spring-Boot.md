---
layout: post
title: "Use JDBI with Spring Boot"
date: 2018-08-13
categories:
- Spring
tags:
- configuration
published: true
---
As a developer I'm always interested in using different data access technologies instead of plain JDBC. Especially combined with Spring Boot. Recently I came across [JDBI](http://jdbi.org) and was intested in trying it out with Spring Boot as an experiment.

I created a simple order system to try it. First the `Order` and the `OrderRepository`

```java
package biz.deinum.orders;

import java.math.BigDecimal;
import java.util.Objects;

public class Order {

    private final String id;
    private final BigDecimal amount;

    public Order(String id, BigDecimal amount) {
        this.id = id;
        this.amount = amount;
    }

    public String getId() {
        return id;
    }

    public BigDecimal getAmount() {
        return amount;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        Order order = (Order) o;
        return Objects.equals(id, order.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }

    @Override
    public String toString() {
        return String.format("Order (id=%s, amount=%s)", this.id, this.amount);
    }
}
```
The `Order` is a simple object holding 2 attributes the `id` and `amount`. The `OrderRepository` declares a few methods to store and retrieve `Order`s from the database.

```java
package biz.deinum.orders;

import java.util.List;
import java.util.Optional;

interface OrderRepository {

    List<Order> findAll();
    Optional<Order> findById(String id);

    Order save(Order order);
    List<Order> saveAll(List<Order> orders);
}
```

The main object in JDBI is [`JDBI`][1] (nicely consistent). To execute a query one requires a [`Handle`][2]. This can be obtained using the `withHandle` (with a return value) or `useHandle` (for no return value) methods on `JBDI`. With that knowledge lets implement a JDBI based repository.

```java
package biz.deinum.orders;

import java.util.List;
import java.util.Optional;

import org.jdbi.v3.core.Jdbi;
import org.jdbi.v3.core.statement.PreparedBatch;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

@Repository
@Transactional
class JdbiOrderRepository implements OrderRepository {

    private static final String INSERT_ORDER_QUERY = "INSERT INTO orders(id, amount) VALUES (:id, :amount);";
    private static final String SELECT_ORDERS_QUERY = "SELECT id, amount FROM orders";
    private static final String SELECT_ORDER_QUERY = "SELECT id, amount FROM orders WHERE id=:id";
    private final Jdbi jdbi;

    JdbiOrderRepository(Jdbi jdbi) {
        this.jdbi = jdbi;
    }

    @Override
    public List<Order> findAll() {
        return jdbi.withHandle(handle ->
                handle.createQuery(SELECT_ORDERS_QUERY)
                        .mapTo(Order.class)).list();
    }

    @Override
    public Optional<Order> findById(String id) {
        return jdbi.withHandle(handle ->
                handle.createQuery(SELECT_ORDER_QUERY)
                        .bind("id", id)
                        .mapTo(Order.class)).findFirst();
    }

    @Override
    public Order save(Order order) {
        jdbi.useHandle(handle ->
                handle.createUpdate(INSERT_ORDER_QUERY)
                        .bind("id", order.getId())
                        .bind("amount", order.getAmount()).execute());
        return order;
    }

    @Override
    public List<Order> saveAll(List<Order> orders) {
        jdbi.useHandle(handle -> {
            PreparedBatch preparedBatch =
              handle.prepareBatch(INSERT_ORDER_QUERY);
            orders.forEach(order -> preparedBatch
                .bind("id", order.getId())
                .bind("amount", order.getAmount()).add());
            preparedBatch.execute();
        });
        return orders;
    }
}
```

The `findAll` method executes the `SELECT_ORDERS_QUERY` and the result will be mapped to an `Order`. The `findById` uses the `SELECT_ORDER_QUERY` and binds the `id` to the `:id` in the query (just as with `setParameter` in JPA!). The find methods do return a value and hence those obtain a `Handle` through the use of `withHandle`.

The `save` method stores a single `Order` and executes the `INSERT_ORDER_QUERY` and binds the `id` and `amount` to the query. The `saveAll` uses a batched update and binds for each `Order` the `id` and `amount`.

Finally the configuration of JDBI. There is [a section][3] on using JDBI with Spring in the guide. This is in XML and actually doesn't work. (See [JDBI-1211][4]). There is an easy solution. Wrap the Spring Boot configured `DataSource` in a [`TransactionAwareDataSourceProxy`][5].

```java
@Bean
public Jdbi jdbi(DataSource dataSource) {
    // JDBI wants to control the Connection wrap the datasource in a proxy
    // That is aware of the Spring managed transaction
    TransactionAwareDataSourceProxy dataSourceProxy = new TransactionAwareDataSourceProxy(dataSource);
    Jdbi jdbi = Jdbi.create(dataSourceProxy);
    jdbi.installPlugins();
    jdbi.installPlugin(new H2DatabasePlugin());

    jdbi.registerRowMapper(Order.class, ConstructorMapper.of(Order.class));

    return jdbi;
}
```

We configure JDBI with
- the `H2DatabasePlugin` for H2 support and
- detect other plugins using `installPlugins`.
- A mapper for the `Order` using a constructor.

**NOTE:** There is a `JdbiFactoryBean` but that isn't as flexible as one would like. It also has a flaw in potentially creating multiple JDBI instances.

Finally we would need a Spring Boot application class and some code to insert and read using the `OrderRepository`.

```java
package biz.deinum.orders;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import java.util.UUID;
import java.util.concurrent.ThreadLocalRandom;

import javax.sql.DataSource;

import org.jdbi.v3.core.Jdbi;
import org.jdbi.v3.core.h2.H2DatabasePlugin;
import org.jdbi.v3.core.mapper.reflect.ConstructorMapper;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy;

@SpringBootApplication
public class OrderJdbiApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderJdbiApplication.class, args);
    }

    @Bean
    public ApplicationRunner ordering(OrderRepository orderRepo) {
        return args -> {

            List<Order> allOrders = orderRepo.findAll();
            System.out.println("# Orders: " + allOrders.size());
            System.out.println("Orders: " + allOrders);

            Order order1 = createRandomOrder();
            orderRepo.save(order1);

            allOrders = orderRepo.findAll();
            System.out.println("# Orders: " + allOrders.size());
            System.out.println("Orders: " + allOrders);


            List<Order> orders = new ArrayList<>(50);
            for (int i = 0; i < 50; i++) {
                orders.add(createRandomOrder());
            }
            orderRepo.saveAll(orders);
            allOrders = orderRepo.findAll();
            System.out.println("# Orders: " + allOrders.size());
            System.out.println("Orders: " + allOrders);

            Optional<Order> order2FromDb = orderRepo.findById(orders.get(ThreadLocalRandom.current().nextInt(50)).getId());
            System.out.println(order2FromDb);
            Optional<Order> noOrderFromDb = orderRepo.findById("noop");
            System.out.println(noOrderFromDb);
        };
    }

    private static Order createRandomOrder() {
        double amount = ThreadLocalRandom.current().nextDouble(1000.00);
        String id = UUID.randomUUID().toString();
        return new Order(id, BigDecimal.valueOf(amount));
    }

    @Bean
    public Jdbi jdbi(DataSource dataSource) {
        // JDBI wants to control the Connection wrap the datasource in a proxy
        // That is aware of the Spring managed transaction
        TransactionAwareDataSourceProxy dataSourceProxy =
            new TransactionAwareDataSourceProxy(dataSource);
        Jdbi jdbi = Jdbi.create(dataSourceProxy);
        jdbi.installPlugins();
        jdbi.installPlugin(new H2DatabasePlugin());

        jdbi.registerRowMapper(Order.class, ConstructorMapper.of(Order.class));

        return jdbi;
    }
}

```
The `@SpringBootApplication` annotated class doesn't do much out of the Spring Boot defaults. There is an `ApplicationRunner` that does some querying and insertion of random created orders.

We would also need to have a schema. Spring Boot has support for schema creation out of the box. With the following `schema.sql` this can be created.

```sql
CREATE TABLE orders (
  id     CHAR(36) PRIMARY KEY,
  amount DECIMAL NOT NULL
);
```

Now when starting the application it should retrieve and save some orders and output it to the console.

![JDBI Output](../img/jdbi-output.png)

Thats it to run JDBI in a Spring Boot application.

Observations
===

As mentioned the `JdbiFactoryBean` isn't as flexible as wanted and contains a flaw. The flaw might result in multiple `Jdbi` instances being created instead of a single one. This is mostly an issue when using Java based configuration and directly referencing the `JdbiFactoryBean`.

When mapping to a bean with a constructor the constructor has to be `public`. I like to have default scoped (or `protected`) constructors so that only classes inside the package can construct instances. When using Spring with a `BeanPropertyRowMapper` this isn't an issue.

Jdbi mappers appear only to work with `public` types. Using a non-public class results in an error.

Jdbi wants to have full control over the `javax.sql.Connection`, which might lead to issues in an environment with managed transactions (like Spring or maybe even a JEE Container).

[1]: http://jdbi.org/apidocs/org/jdbi/v3/core/Jdbi.html
[2]: http://jdbi.org/apidocs/org/jdbi/v3/core/Handle.html
[3]: http://jdbi.org/#_spring
[4]: https://github.com/jdbi/jdbi/issues/1211
[5]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/datasource/TransactionAwareDataSourceProxy.html
