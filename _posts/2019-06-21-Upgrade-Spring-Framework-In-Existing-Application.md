---
layout: post
title: "Upgrade Spring Framework in an existing application"
date: 2019-06-21
categories:
- Java
- Spring
tags:
- configuration
published: true
---

When working on an existing application using an older version of the Spring Framework (aka Spring) it might be time to upgrade. Although in most cases upgrading to a newer version of Spring is a matter of changing the version. When doing a large upgrade, like from 2.5 to 5.1, you might have challenges.

In all those versions several things happened with Spring
1. Packing of Spring changed from one large jar to modules (one spring.jar to different modules)
2. The names of the modules used changed (spring-mock -> spring-test, spring-dao -> spring-orm)
3. Framework support might have dropped (like Struts 1, Castor or Velocity)
4. Unsupported framework versions (like Jackson 1 and Hibernate 3)

When upgrading to a newer version of Spring I follow a couple of rules:
1. Start with a working application!
2. Take small steps
3. Upgrade one library / version at a time
4. Change one thing at a time
5. Write tests when not already in place
6. If something breaks, revert

Start with a working application
===
You always want to start with an application that works. This might sound like a silly rule but more then once I have been part of a team in charge of fixing an application and on the side also upgrade frameworks. It is hard to keep track of what breaks what, is it the upgrade or the attempt to fix other issues in the application.

Take small steps
===
Taking small steps is important to limit the scope of a change. When trying to upgrade with a span of 8 major versions the amount of changes will be overwhelming. It requires changes to the web parts of your application, the persistence part, and several libraries that are in use. Then when things start to fail it is hard to determine what is the cause.

I generally first upgrade to the fix version of Spring for the version we are at. So if we are at 3.2.4 I would generally first upgrade to 3.2.18. Then upgrade one version at a time 4.0 -> 4.1 etc. This way you only have to grok the changes from one version to the next, else you would need to incorporate all the changes from 3.2 to 5.1 in one go.

Upgrade one library / version at a time
===

This is related to the fact that one should take small steps. When trying to upgrade multiple library versions in one go chances are things will break. Controlling that with one library upgrade at a time will help in reducing the risk. As well as keeping a clean view on what lead to the failure.

Unsupported Library Version
====

When upgrading to a version of a Spring that dropped support for a certain framework/library you want to first upgrade the libraries in use. For instance Spring 5 dropped support for Hibernate 3 and 4. When your application is using Hibernate 4 you first want upgrade to Hibernate 5 before upgrading Spring. First use a Spring version that supports both versions and upgrade Hibernate. After that you can continue to upgrade Spring (or repeat for another library).


**NOTE:** Sometimes library support is dropped, you can then refactor and ditch the library. But that isn’t always feasible. Another option is to copy the sources from Spring (it is open-source) and maintain the removed parts yourself. We did this once on a Struts1 based project. But always consider this to be a migration strategy rather than a long term strategy.

**NOTE 2:** When a library gets removed, upgraded etc. you need a test harness to make sure that the code still works in the same way as it did before. This is especially important if you expose an external API, you don't want your clients to suffer.

Testing
===
As always when developing software tests are your friends. Especially when it comes to verifying contracts and the overal state of the application.

If you have REST or SOAP based services exposed to external parties write tests for them. The tests should ensure that the contract of those services did't break between different framework/library versions. For instance upgrading from Jackson 1 to Jackson 2 could possible change the JSON being send back to the clients. Now if you also control the client that doesn't have to be a problem. But if you have this is a public API you want to ensure correctness.

Reverting is always an option
===
When something breaks, you can always revert the last change. This the advantage of doing one upgrade/thing at a time. After the revert figure out what to do, change libraries, hold the upgrade, upgrade an other library etc. 
