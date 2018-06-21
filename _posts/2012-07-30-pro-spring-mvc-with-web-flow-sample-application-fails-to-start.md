---
author: mdeinum
comments: true
date: 2012-07-30 18:24:30+00:00
layout: post
link: https://mdeinum.wordpress.com/2012/07/30/pro-spring-mvc-with-web-flow-sample-application-fails-to-start/
slug: pro-spring-mvc-with-web-flow-sample-application-fails-to-start
title: 'Pro Spring MVC: with Web Flow - Sample Application Fails To Start'
wordpress_id: 65
categories:
- Java
- Spring
- Spring Web Flow
tags:
- pro-spring-mvc
- sample
- stack trace
- web flow
---

**Update (1-8-2012): **Please download the most recent version of the sources from either the [GitHub repository](https://github.com/mdeinum/pro-spring-mvc-code) or from the [Apress website](http://www.apress.com/9781430241553). This version fixes the problem with the project dependencies.

You arrived at this page because of 2 reasons. First you bought Pro Spring MVC: with Web Flow for which I would like to thank you and I really hope it is a valuable addition to your book shelve. Secondly you are trying to run the sample application which ships with the book but the application fails to start/deploy (and sometimes also has some compilation errors), for this we are really sorry and we apologize for the inconvenience that has given you. We also read books and we know that there is nothing so frustrating as a book with source code and the sources don't compile or the sample doesn't work.

[![STS Startup Error](http://mdeinum.files.wordpress.com/2012/07/sts-startup-error.png?w=300)](http://mdeinum.files.wordpress.com/2012/07/sts-startup-error.png)


## The Problem


For some reason the Gradle tooling we choose sometimes fails to correctly resolve and reference projects in the same workspace, this is either due to some error in our build file (despite all the testing we did) or it is an error/undocumented feature of the Gradle Tooling. However as a reader it doesn't really help you. (<del>See [GRADLE-2409](http://issues.gradle.org/browse/GRADLE-2409)</del>).

When one now deploys the application you get a nice stack trace as shown on the left (click for larger image). This is, believe us, not the experience we wanted you to have. Following are a couple of workarounds which can help you get it to work, meanwhile we will also try to fix the build script so that the tooling resolves correctly. You can always get the most recent sources from our [GitHub repository](https://github.com/mdeinum/pro-spring-mvc-code) this you can always fork or download as a ZIP file.


## <!-- more -->Workarounds


Before we describe 2 workarounds which where reported to work however depending on the combination of your Operating System, Java Version and STS version [YMMV](http://en.wiktionary.org/wiki/your_mileage_may_vary). The one that always works is the manual workaround however this requires some work on your part and you need to modify all the projects that don't deploy correctly. The quickest way it to use the Gradlew command from the command line to regenerate the Eclipse artifacts.


### Manual Modification of Project Properties





	
  1. Right-click on the project that doesn't deploy and select Properties from the menu. [[screenshot](http://mdeinum.files.wordpress.com/2012/07/manual-step1.png)]

	
  2. In the 'Properties for' select 'Deployment Assembly'. You will notice that there are 2 errors indicating that the entries cannot be found. Scroll down to find the 2 broken projects (bookstore-shared and bookstore-web-resources) and remove them (first select the entries and press the '**Remove**' button) [[screenshot](http://mdeinum.files.wordpress.com/2012/07/manual-step2.png)]

	
  3. After the removal press the '**Add**' button, in this screen select 'Project' and press '**Next >**' [[screenshot](http://mdeinum.files.wordpress.com/2012/07/manual-step3.png)]

	
  4. Now select the projects you want to add to the deployment assembly, select both the _bookstore-shared_ and _bookstore-web-resources_ project and press '**Finish**'. Now the projects will be added correctly to the classpath. [[screenshot](http://mdeinum.files.wordpress.com/2012/07/manual-step4.png)]

	
  5. Now remove the project from the server and redeploy now you should be able to use the application

	
  6. Repeat these steps for all the projects that fail.




### Use Gradle Eclipse plugin to generate .project files


From the command line run the gradlew command and use it to generate the artifacts needed for eclipse, after this import the projects as existing Eclipse projects. With this you will loose the ability to have the Eclipse Gradle Tooling take care of the classpath but at least it might give you a working workspace. You can do this on already imported projects.



	
  1. From the command line run './gradlew clean cleanEclipse eclipse' this will let Gradle regenerate the Eclipse artifacts

	
  2. In Eclipse select all the projects, right-click and select Refresh. Sit back and wait, when Eclipse has finished refreshing try to redeploy the application (first remove it from the server).


