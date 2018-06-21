---
author: mdeinum
comments: true
date: 2009-05-25 06:59:27+00:00
layout: post
link: https://mdeinum.wordpress.com/2009/05/25/spring-web-flow-2-web-development/
slug: spring-web-flow-2-web-development
title: Spring Web Flow 2 Web Development
wordpress_id: 44
categories:
- Reviews
---

A few weeks ago I was contacted by [packt publishing](http://www.packtpub.com) to review one of their books. They asked me if I wanted to review [Spring Web Flow 2 Web Development](http://www.packtpub.com/develop-powerful-web-applications-with-spring-web-flow-2/book) (sample [chapter](http://www.packtpub.com/files/spring-web-flow-2-web-development-sample-chapter-4-spring-faces.pdf)).

In short the book consists of 254 pages in all those pages they try to explain Spring Web Flow, Maven, Ant+Ivy, Spring Security and some basics (and also not all basics) about the Spring Framework it self. Ofcourse not to mention all the normal stuff like title pages and indexes. All the explaining of the added frameworks takes away from the actual goal and that is to cover and explain Spring Web Flow. I also found that there is no real layering/build up in the book, they directly start with FlowExecutionListeners even before they covered the basics. Another draw back is the samples used, they don't really use one sample (application) but different snippets which make it harded for a new Spring Web Flow user to get an overall feel. So the short verdict I wouldn't say the book is bad but it could be a lot better, most of the information to be found in the book but it can be quite a search at times. If you are a Spring Web Flow beginner I would try to find other books covering the basics of Spring Web Flow.
<!-- more -->
After the short version here is the somewhat longer version a chapter by chapter brief coverage of the book. The opinions here are my own and are quite influenced by the fact that I have written some initial code for Spring Web Flow and Spring Security integration and that I was part of the team that wrote [the training on Spring Web Flow](http://www.springsource.com/training/rwa001).

The book starts with a short and quick introduction to the 3 pillars/corners stones and briefly discusses the changes between Spring Web Flow 1 and 2. The chapter is a bit confusing due to the fact that they use flow and conversation interchangeably. There are some small parts that could be improved here but all in all not a bad chapter.

The second chapter discusses the setup to run the examples which come with the book. They explain it using maven, ant+ivy and how to do it with various IDEs. They also give a 'basic' sample of a flow, however they right away start with FlowExecutionListeners, all the different states that there are and try to explain it. However the explanation is to brief (allthough everything is covered in the book). I would have sticked with 1 build solution and 1 IDE. That would also make it easier to read and understand, now with each sample the book covers both maven and ant+ivy. Also showing a sample with everything included and calling it 'a basic sample' isn't the best thing to do. The sample flow also uses depracted states (action-state is basically deprecated) and doesn't use/follow best practices. Overall everything is in the chapter but it is either to much or to little.

The third chapter explains the basics of Spring Web Flow. The chapter start with the explanation of how xml is composed and after that starts with explaining the persistence context and FlowExecutionListeners. Both not really 'basics of Spring WebFlow'. Halfway the chapter they start explaining the basics, the different states, the different scopes, how the variable resolution mechanism works etc. Basically they try to explain everything about Spring Web Flow with jsp in 1 chapter and they don't really explain it in the right order. Some parts could have been in more detail others are in to much detail. Also parts that would be interesting, like how to influence binding and how to do type conversion, aren't covered. In my opinion this chapter lacks what it needs.

The fourth chapter is about integration JSF with Spring Web Flow. They again briefly explain how to do it and they try to explain how to integrate 2 different JSF frameworks with Spring Web Flow. However the latter removes us from the real objective which is the Spring Faces module and how to use it. Some of the code and things they explain aren't really true (like the custom ResourceServlet you would need!), next to that they are also trying to explain how the Spring Namespace support works which doesn't really apply nor does it add something here.

The fifth chapter covers AJAX. They brush on element decoration and progressive enhancement, however they don't really explain why one should use it. The book covers the configuration and usage of Spring Javascript and next delves into partial page rendering with the use of Apache Tiles. The beginning of the chapter is pretty good however they end with explaining the whole Spring Web Flow configuration xsd. This is better left to the [reference guide](http://static.springframework.org/spring-webflow/docs/2.0.x/reference/html/index.html) or in an appendix in the book. The chapter starts off good however seems/feels likes to be rushed to an end with explaining the xsd which also feels oddly out of place here.

Chapter six covers testing and the testing of subflows. One of the best chapters in the book, it covers testing pretty good and also touches on the subject of mocking your objects and perform tests. Confusing in this chapter arises due to the use of 2 different versions of the testing framework, stick with 1 version I would say. 

Chapter seven dives into Spring Security and using that to secure your flows. Again they try to explain Spring Security and the mix with Spring Web Flow in 1 chapter, this doesn't really work. They explain to little and in places don't do justice to what Spring Security can do. Not all to bad but it again coud have been better then what it is.

A lot of stuff gets covered in this book but it doesn't read pretty easy. The beginning of the book directly covers FlowExecutionListeners even before explaining the real basics of Spring Web Flow. If you need something it probably is in there but not in the most likely location. They also use different samples through out the book, if they would have used just one sample application it would have been easier to understand and to give everything a place. The code samples are at times also confusing and use different styles (jsp 2 and jsp 1 syntax, use of deprecated code, incomplete/incorrect code samples etc) and sometimes use different versions of the same framework. Things that would have been good to explain aren't explained (binding, object conversion).  

All of this leaves me no choice then to not recommend this book especially if you are a Spring Web Flow beginner. Try to find another book there are probably better ones out there.  




