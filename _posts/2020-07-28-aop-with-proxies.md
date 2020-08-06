---
layout: post
title: Aspect Oriented Programming using JDK Dynamic Proxies
date: 2020-07-28
categories:
- Java
tags:
- aop
- proxies
published: false
gh-repo: mdeinum/blog-aop-with-proxies
gh-badge: star	
---

## Aspect-Oriented Programming
First, define Aspect-Oriented Programming.  

aspect-oriented programming (AOP) aims to increase modularity by allowing the separation of cross-cutting concerns. It does so by adding behavior to existing code (an advice) without modifying the code itself, instead separately specifying which code is modified via a "pointcut" specification, such as "log all function calls when the function's name begins with 'set'". This allows behaviors that are not central to the business logic (such as logging) to be added to a program without cluttering the code, core to the functionality.

The functionality of an application which isn't core functionality belongs in separate classes. The functionality will be added at execution time. The functionality to add can be things like logging, transaction management etc. 

For frameworks as Spring and features as CDI, AOP is a core concept to achieve separation of concerns. To enable things like `@Transactional`, Spring and CDI use AOP. By default, they will use (dynamic) proxies to enable AOP. 

The JDK supports proxies by itself and needs no extra frameworks. But there is one caveat, it can only do interface-based proxies. This means only classes implementing an interface are useable. If one doesn't use interfaces one needs a framework like CgLib or ByteBuddy.

## Dynamic Proxies
A dynamic proxy is a class generated at runtime which implements the interfaces of the target. The dynamic proxy can add behaviour to the target before or after invoking the method. It could even prevent the invocation of the method on the target.

To create a proxy the JDK provides the aptly named class `Proxy`. Together with an `InvocationHandler` this is all the infrastructure one needs.  
