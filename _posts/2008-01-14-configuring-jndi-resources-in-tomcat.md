---
author: mdeinum
comments: true
date: 2008-01-14 13:40:54+00:00
layout: post
link: https://mdeinum.wordpress.com/2008/01/14/configuring-jndi-resources-in-tomcat/
slug: configuring-jndi-resources-in-tomcat
title: Configuring JNDI Resources in Tomcat
wordpress_id: 19
categories:
- Java
tags:
- jndi
- tomcat
---

It seems quite hard to configure a [JNDI Resource](http://tomcat.apache.org/tomcat-6.0-doc/jndi-resources-howto.html) in [Tomcat](http://tomcat.apache.org). Especially when it comes to configuring XA capable resources. The key lies in understanding how Resources work/need to be configured.
<!-- more -->
A [Resource](http://tomcat.apache.org/tomcat-6.0-doc/config/context.html#Resource%20Definitions) has a few properties which are used by Tomcat, those are the _auth_, _name_, _description_, _scope_ and _type_, from those **name** and **type** are required. Type is the actual type (fully qualified classname) it needs to create and the name is the jndi name relative to `java:comp/env`.

All the other properties specified on the Resource element are either used by the factory or the type created by the factory. So if you specify a property named `username``'` in your `Resource` then there must be a get/set pair for that property on the type (i.e. `setUsername`/`getUsername`).

If we look at the mysql sample from the Tomcat documentation.

```xml
<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
          maxActive="100" maxIdle="30" maxWait="10000"
          username="javauser" password="javadude"
          driverClassName="com.mysql.jdbc.Driver"
          url="jdbc:mysql://localhost:3306/javatest?autoReconnect=true"/>
```

We register a `javax.sql.DataSource` with the name `jdbc/TestDB` and the authentication is done by the Container. Now if we look further we see a few properties.

* `maxActive`
* `maxIdle`
* `maxWait`
* `username`
* `password`
* `driverClassName`
* `url`



If we don't specify an explicit type and/or a factory Tomcat uses its default implementation which is [Apache Commons DBCP](http://commons.apache.org/dbcp/). For the default `javax.sql.DataSource` it uses also the default/basic implementation of that library, it uses [BasicDataSource](http://commons.apache.org/dbcp/api-1.2.2/org/apache/commons/dbcp/BasicDataSource.html). If you study the javadoc of that class you will see a getter/setter pair for each of the properties named above.

Now lets put into practice all what we have learned here and try to configure a Mysql XA Capable datasource. We first need to specify the specific factory to use else the default factory will be used. The factory to use is the [`MysqlDataSourceFactory`](http://www.docjar.com/docs/api/com/mysql/jdbc/jdbc2/optional/MysqlDataSourceFactory.html)

Next we need to specify the type the factory needs to instantiate, we want a MySQL XA DataSource and thus we need to instantiate `com.mysql.jdbc.jdbc2.optional.MysqlXADataSource`. A list of the configurable properties can be found in the MySQL [reference documentation](http://dev.mysql.com/doc/refman/5.1/en/connector-j-reference-configuration-properties.html).

```xml
<Resource name="jdbc/TestDB" auth="Container"
          type="com.mysql.jdbc.jdbc2.optional.MysqlXADataSource"
          factory="com.mysql.jdbc.jdbc2.optional.MysqlDataSourceFactory"          
          user="javauser" password="javadude" explicitUrl="true"
          url="jdbc:mysql://localhost:3306/javatest?autoReconnect=true"/>
```

And there we have it a configured XA Capable Mysql datasource. I think the `MysqlDataSourceFactory` can be improved a lot the same goes for the javadocs for the connector. However with a look at the sourcecode and a good understanding on how JNDI Resources can be configured in Tomcat it is quite easy to figure out how Resources can be configured.
