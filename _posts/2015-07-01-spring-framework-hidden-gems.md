---
author: mdeinum
comments: true
date: 2015-07-01 13:11:16+00:00
layout: post
link: https://mdeinum.wordpress.com/2015/07/01/spring-framework-hidden-gems/
slug: spring-framework-hidden-gems
title: 'Spring Framework: "Hidden" Gems'
wordpress_id: 155
categories:
- Java
- Spring
tags:
- aop
---

The [Spring Framework](http://projects.spring.io/spring-framework/) (and its portfolio projects) contain a lot of functionality already by themselves. However there are also some nice hidden gems inside the framework, in this blog I will (un)cover a couple of them. The code for the sample(s) can be found on [Github](https://github.com/mdeinum/samples/tree/master/spring-gems).


### <!-- more -->




### Log incoming Requests


A lot of projects I worked on wanted, for debugging purpose, log the incoming requests. Sadly I also saw a lot of homegrown solutions for this (although not necessarily a bad thing). The Spring Framework has a nice servlet filter which already provides this functionality out-of-the-box. The [AbstractRequestLoggingFilter](http://docs.spring.io/autorepo/docs/spring-framework/current/javadoc-api/org/springframework/web/filter/AbstractRequestLoggingFilter.html) provides 3 concrete implementations, one for Commons Logging, one for Log4J and 1 that uses the servlet context for logging.

Using Spring Boot the following bean definition will enable request logging, it will include the information from the client (always the IP Address and optional the session id and username), the query string and the posted payload (if any is available).

    
    @Bean
    public CommonsRequestLoggingFilter requestLoggingFilter() {
        CommonsRequestLoggingFilter crlf = new CommonsRequestLoggingFilter();
        crlf.setIncludeClientInfo(true);
        crlf.setIncludeQueryString(true);
        crlf.setIncludePayload(true);
        return crlf;
    }


To enable logging the log level for `org.springframework.web.filter.CommonsRequestLoggingFilter` must be set to `DEBUG`, this can be done using the `application.properties` by adding:

    
    logging.level.org.springframework.web.filter.CommonsRequestLoggingFilter=DEBUG


When a request comes in the following lines will be available in the logging.

    
    2015-06-30 08:52:44.581 DEBUG 66103 --- [nio-8080-exec-3] o.s.w.f.CommonsRequestLoggingFilter : Before request [uri=/hello-world?name=World;client=0:0:0:0:0:0:0:1]
    2015-06-30 08:52:44.584 DEBUG 66103 --- [nio-8080-exec-3] o.s.w.f.CommonsRequestLoggingFilter : After request [uri=/hello-world?name=World;client=0:0:0:0:0:0:0:1]
    




### Logging method invocations


At times you might want to log the different method invocation, including method arguments and/or return values or errors. Spring has a very customizable interceptor for this, which is conveniently named [`CustomizableTraceInterceptor`](http://docs.spring.io/autorepo/docs/spring/current/javadoc-api/org/springframework/aop/interceptor/CustomizableTraceInterceptor.html). As the interceptor is pretty old, it exists since Spring 1.2, you need to do some additional configuration to get it to work.

First configure the interceptor

    
    @Bean
    public CustomizableTraceInterceptor customizableTraceInterceptor() {
        CustomizableTraceInterceptor cti = new CustomizableTraceInterceptor();
        return cti;
    }


Next configure an `Advisor` so that it can be applied when enabling AOP.

    
    @Bean
    public Advisor traceAdvisor() {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(public * biz.deinum..*.*(..))");
        return new DefaultPointcutAdvisor(pointcut, customizableTraceInterceptor());
    }


This advisor basically says intercept all public methods to classes in the `biz.deinum` package and all sub-packages. We also need to enable class based proxies, as we also want to intercept calls to the `HelloWorldController` which doesn't implement an interface. To enable this add the following to the `application.properties`.

    
    spring.aop.proxy-target-class=true
    


One last step is to enable TRACE logging for the interceptor else nothing will be logged, again this is as easy as adding an entry to the `application.properties` again.

    
    logging.level.org.springframework.aop.interceptor.CustomizableTraceInterceptor=TRACE


Now when restarting the application and issuing a request the following lines are added to the logging.

    
    2015-06-30 09:13:33.577 TRACE 66322 --- [nio-8080-exec-1] o.s.a.i.CustomizableTraceInterceptor     : Entering method 'sayHello' of class [biz.deinum.gems.web.HelloWorldController]
    2015-06-30 09:13:33.583 TRACE 66322 --- [nio-8080-exec-1] o.s.a.i.CustomizableTraceInterceptor     : Entering method 'greet' of class [biz.deinum.gems.SimpleHelloWorldService]
    2015-06-30 09:13:33.588 TRACE 66322 --- [nio-8080-exec-1] o.s.a.i.CustomizableTraceInterceptor     : Exiting method 'greet' of class [biz.deinum.gems.SimpleHelloWorldService]
    2015-06-30 09:13:33.589 TRACE 66322 --- [nio-8080-exec-1] o.s.a.i.CustomizableTraceInterceptor     : Exiting method 'sayHello' of class [biz.deinum.gems.web.HelloWorldController]
    


Although this tells you when a method is entered and left it doesn't include the method arguments or performance metrics. For this we need to configure the enter and exit message for the `CustomizableTraceInterceptor`.

    
    @Bean
    public CustomizableTraceInterceptor customizableTraceInterceptor() {
        CustomizableTraceInterceptor cti = new CustomizableTraceInterceptor();
        cti.setEnterMessage("Entering method '" + PLACEHOLDER_METHOD_NAME + "("+ PLACEHOLDER_ARGUMENTS+")' of class [" + PLACEHOLDER_TARGET_CLASS_NAME + "]");
        cti.setExitMessage("Exiting method '" + PLACEHOLDER_METHOD_NAME + "' of class [" + PLACEHOLDER_TARGET_CLASS_NAME + "] took " + PLACEHOLDER_INVOCATION_TIME+"ms.");
        return cti;
    }


Now when restarted the value of the method arguments is included as well as the time it took to execute the method.

    
    2015-06-30 09:21:55.595 TRACE 66368 --- [nio-8080-exec-1] o.s.a.i.CustomizableTraceInterceptor     : Entering method 'sayHello(World)' of class [biz.deinum.gems.web.HelloWorldController]
    2015-06-30 09:21:55.602 TRACE 66368 --- [nio-8080-exec-1] o.s.a.i.CustomizableTraceInterceptor     : Entering method 'greet(World)' of class [biz.deinum.gems.SimpleHelloWorldService]
    2015-06-30 09:21:55.608 TRACE 66368 --- [nio-8080-exec-1] o.s.a.i.CustomizableTraceInterceptor     : Exiting method 'greet' of class [biz.deinum.gems.SimpleHelloWorldService] took 6ms.
    2015-06-30 09:21:55.608 TRACE 66368 --- [nio-8080-exec-1] o.s.a.i.CustomizableTraceInterceptor     : Exiting method 'sayHello' of class [biz.deinum.gems.web.HelloWorldController] took 13ms.
    




### Performance Logging


Although you can use the `CustomizableTraceInterceptor` to log performance you might want to have a bit more information regarding your performance logging. There are various solution out there one of them is [JAMon](http://jamonapi.sourceforge.net) and Spring provides the convenient [`JamonPerformanceMonitorInterceptor`](http://docs.spring.io/autorepo/docs/spring/current/javadoc-api/org/springframework/aop/interceptor/JamonPerformanceMonitorInterceptor.html) to intregrate with this.

First add the `JamonPerformanceMonitorInterceptor` as a bean

    
    @Bean
    public JamonPerformanceMonitorInterceptor jamonPerformanceMonitorInterceptor() {
        return new JamonPerformanceMonitorInterceptor();
    }


Next add the point cut expression to assure that the interceptor is executed when a method is executed.

    
    @Bean
    public Advisor performanceAdvisor() {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(public * biz.deinum..*.*(..))");
        return new DefaultPointcutAdvisor(pointcut, jamonPerformanceMonitorInterceptor());
    }


The final step is to enable the logger by setting the log level for this class to TRACE.

    
    logging.level.org.springframework.aop.interceptor.JamonPerformanceMonitorInterceptor=TRACE


Now when the application is restart again the following lines will appear in the logs.

    
    2015-06-30 09:43:01.748 TRACE 66496 --- [nio-8080-exec-7] s.a.i.JamonPerformanceMonitorInterceptor : JAMon performance statistics for method [biz.deinum.gems.web.HelloWorldController.sayHello]:
    JAMon Label=biz.deinum.gems.web.HelloWorldController.sayHello, Units=ms.: (LastValue=0.0, Hits=4.0, Avg=3.5, Total=14.0, Min=0.0, Max=13.0, Active=0.0, Avg Active=1.0, Max Active=1.0, First Access=Tue Jun 30 09:42:52 CEST 2015, Last Access=Tue Jun 30 09:43:01 CEST 2015)
    


**Note:** JAMon itself has more in-depth support for performance logging (also for web requests, JDBC etc.) you might want to check out.


### Expose Hibernate SessionFactory when using JPA


When in the proces of migrating a classic [Hibernate](http://www.hibernate.org) based application to JPA (or simply needing a `SessionFactory`) you might need both an `EntityManagerFactory` and `SessionFactory` in the same application. You can configure both beans and try to get it to work with 2 different `PlatformTransactionManager`s but it can be done easier. Spring provides the `HibernateJpaSessionFactoryBean` to expose the underlying `SessionFactory` from the `EntityManagerFactory`.

To configure the bean simply add it to the configuration

    
    @Bean
    public HibernateJpaSessionFactoryBean sessionFactory(EntityManagerFactory emf) {
        HibernateJpaSessionFactoryBean jssfb = new HibernateJpaSessionFactoryBean();
        jssfb.setEntityManagerFactory(emf);
        return jssfb;
    }


And now you can use the underlying `SessionFactory` to get access to a plain `Session`.


### WebUtils and ServletRequestUtils


When writing controller classes or otherwise working with the `HttpServletRequest` or `HttpSession` can be cumbersome, especially when doing type conversion or `null` checks for the existence of the `HttpSession` for instance. The [`WebUtils`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/util/WebUtils.html) and `[ServletRequestUtils](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/ServletRequestUtils.html)` classes have some nice helper methods to ease working with the session and request attributes respectively.

When needing an attribute from the session you often do `request.getSession(false)` and check if the session is `null`. Code like this is not uncommon to see:

    
            HttpSession session = request.getSession(false);
            if (session != null) {
                String foo = session.getAttribute("foo");
            }
    


When using the `WebUtils` this can be reduced to a oneliner.

    
    String foo = WebUtils.getSessionAttribute(request, "foo");


When the property is required you can use the `getRequiredSessionAttribute` method, which in case of a missing attribute throws an exception.

When using a `(Http)ServletRequest` sometimes you need to get a parameter of the request and sometimes need to convert it to a different type. (Although I would recommend using `@RequestParam` in your method signature for that). You can use the `ServletRequestUtils` to make that easier.

    
            String name = request.getParameter("name");
            if (name == null) {
                name = "Stranger";
            }
    


When using the helper methods from `ServletRequestUtils` this becomes:

    
    String name = ServletRequestUtils.getStringParameter(request, "name", "Stranger");
    


If the parameter is required you can use the `getRequiredStringParameter`. Next to getting a text based parameter it can also help with converting to a different type.

    
            String val = request.getParameter("someId");
            long id = val == null ? -1L : Long.valueOf(val);
    


With the helper methods.

    
    long id=ServletRequestUtils.getLongParameter(request, "someId", -1L);
    


**Note:** Although it makes your code easier it also makes your code (more) dependent on Spring, you have to consider for yourself if this is a bad thing or not. The `WebUtils` and `ServletRequestUtils` will be mainly used in controllers which are generally already using Spring interfaces or annotations.
