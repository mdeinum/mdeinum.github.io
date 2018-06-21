---
author: mdeinum
comments: true
date: 2009-05-06 11:10:45+00:00
layout: post
link: https://mdeinum.wordpress.com/2009/05/06/migrate-classic-j2ee-to-spring/
slug: migrate-classic-j2ee-to-spring
title: Migrate classic J2EE to Spring (Step 1)
wordpress_id: 21
categories:
- J2EE
- Java
- Spring
tags:
- Conversion
- Migration
---

A few months ago [SpringSource ](http://www.springsource.com)released [a white paper](http://www.springsource.com/files/MigratingAppsToSpring.pdf) describing the migration from J2EE to a Spring framework based application. Even before they wrote that white paper I already had the idea of writing something about how to migrate from J2EE to a Spring based application. However Colin Sampaleanu (at al.) beat me to writing the white paper :). 

However I also had the idea of providing some practical information like a sample. The white paper gives some thought on what to do and where but it doesn't give a clear sample. So I started writing a possible migration path (multi step sample) from a J2EE application to a Spring based application. 
<!-- more -->
The code is available in a [repository hosted on googlecode](http://springstore.googlecode.com). Please feel free to checkout the code (for each step there will be a branch/tag). If you have suggestions/issues with the code also please feel free to register an issue or supply a patch. 

For this sampe I took the SUN blueprint application (I focused on J2EE 1.4 because that is widely used and adopted and there is still a lot of 'legacy' out there build with the 1.4 spec) called the '[Adventure Builder](http://java.sun.com/developer/releases/adventure)'. To make it a easier to build and migrate I started migrating the application to a maven build (instead of the asant build). 

In the next few post I'll explain the migration strategy, the different steps and here and there some best practices which later on will ease the migration path. I'm not going to migrate all the applications (the Adventure Builder sample consists of several applications), I'm focussing on the 'Order Processing Module' and later on the 'Consumer Website' application all of this without changing the other applications (or with minor changes).

Development for this project is done in the [SpringSource Toolsuite](http://www.springsource.com/products/suite/sts). Deployment is done in the '[Glassfish](http://www.glassfish.org)' application server. To make it easier to build I first migrated the blueprint application to a [maven](http://maven.apache.org) based build.

To build, install and deploy the application follow the following steps (after checking out the code from svn).



	
  1. goto the parent project inside the springstore application

	
  2. `mvn clean -Dskip.resources=true` (skip.resources disables deletion of the jms/jdbc resources in glassfish)

	
  3. `mvn glassfish:start-domain` (start glassfish)

	
  4. `mvn install` (will build the ear files, and generate the jms/jdbc resources and fills the test database)

	
  5. `mvn glassfish:deploy`



Afterwards the applications is available on your [localhost](http://localhost:8080/ab) just play around with the application and have a look at the code. (If you want you can compare it to the original Adventure Builder application it should be more or less the same. 

If you want to undeploy the applications simply type `glassfish:undeploy`

I still have [a few issues with the maven based build](http://code.google.com/p/springstore/issues/list?can=2&q=label%3AComponent-Scripts), so if someone can help/advice on that please let [me](mailto:mdeinum@gmail.com) know.
