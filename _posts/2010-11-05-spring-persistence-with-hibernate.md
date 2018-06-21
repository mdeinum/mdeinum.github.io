---
author: mdeinum
comments: true
date: 2010-11-05 10:25:25+00:00
layout: post
link: https://mdeinum.wordpress.com/2010/11/05/spring-persistence-with-hibernate/
slug: spring-persistence-with-hibernate
title: Spring Persistence with Hibernate
wordpress_id: 61
categories:
- Java
- Reviews
- Spring
---

[This book](http://www.amazon.com/Spring-Persistence-Hibernate-Ahmad-Seddighi/dp/1849510563/ref=sr_1_2?ie=UTF8&s=books&qid=1289125358&sr=1-2) sets out to explain the usage of hibernate with the spring framework. This is basically done in the first chapters. It explains how to configure hibernate from within a spring ApplicationContext and it explains how to write a dao. To bad that, for the dao, they still use the old technique with HibernateTemplate/HibernateDaoSupport which isn’t recommended anymore since the release of spring 2.0, it is still in the framework for backwards compatibility. The book would have been better if they would explain this.

The remainder of the book is more or less an introduction to hibernate and explains how to write and execute HQL based queries, use the (Detached)Criteria API. Next to that it tries to explain spring, dependency injection and Spring MVC. In short everything is touched upon, but all just to little. 

If you need a kickstart into configuring hibernate and spring and don’t have an hibernate knowledge this book could be starting guide. If you already have some knowledge on how to do those things, I would suggest JPA persistency with Hibernate (although a bit dated) and the spring reference guide. For the other parts there are some great books out there.

