---
author: mdeinum
comments: true
date: 2009-10-02 17:10:45+00:00
layout: post
link: https://mdeinum.wordpress.com/2009/10/02/migrate-classic-j2ee-to-spring-step-1-revised/
slug: migrate-classic-j2ee-to-spring-step-1-revised
title: Migrate classic J2EE to Spring (Step 1) (revised)
wordpress_id: 51
categories:
- J2EE
- Spring
tags:
- Conversion
- Java
- Migration
---

A couple of months ago I wrote a [step 1 on migrating a classis J(2)EE application to a spring based application](http://mdeinum.wordpress.com/2009/05/06/migrate-classic-j2ee-to-spring/). Recently I had some time again on my hands and after studying the application and also after some discussions I had I decided to structure the project a little different. 



#### "External" applications


The Adventure Builder applications uses 4 other applications to deliver its services. However in a normal real situation those 4 applications are outside of our control. We cannot change them nor redeploy them. So I decided to just include the ear files for those 4 projects instead of rebuilding them each time.



#### Generated stubs/skeletons


The application makes use of generated java and xml files. Also in a real situation you should generate those once and after that reuse. You should only (re)generate those files if the external interface changes (best would be to not generate at all). I decided to use the generated java files and xml files and include them in the project. They can be found in the src/generated directory.



#### Deployment


Deployment is now maven based, maven can start and create a glassfish domain for you. It will create all jms/jdbc/mail resources and deploy the 4 external applications. After that setup you can choose to start glassfish with the normal startServ command or reuse maven to start it. Simply run the setup.bat/setup.sh file from the setup directory. To deploy our own 2 applications simply go to the apps directory and type mvn glassfish:deploy.



#### Testing


The initial project as is has some test classes available so that we can make sure that the application still behaves as it should behave after we are going to change it to use Spring and Hibernate.

Hopefully Step 2 will follow shortly and that we can have a look at the different steps and easier code.


