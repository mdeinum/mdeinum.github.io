---
author: mdeinum
comments: true
date: 2007-01-05 14:45:13+00:00
layout: post
link: https://mdeinum.wordpress.com/2007/01/05/one-application-per-client-database/
slug: one-application-per-client-database
title: One application, per client database
wordpress_id: 15
categories:
- Java
- Spring
tags:
- aop
- spring aop
---

In one of my recent projects we came a cross an application model which had 1 codebase but for every client they had (around 40) they deployed one application. Sometimes they had to redeploy several times because they had memory and performance issues. We soon realized that we needed to do something about this way of deploying. The only two things which where different per client where the database connection and the front-end.
<!-- more -->

**Front-end**
Well the front-end is simple enough there are enough templating engines around which you can use. We decided on using [sitemesh](http://www.opensymphony.com/sitemesh/) this in combination with JSTL gave us all the power we needed.

**Database connection**
But what about the database connections. We had 40 of them configured in our tomcat context file. We figured we needed something like the [`HotSwappableTargetSource`](http://static.springframework.org/spring/docs/2.0.x/api/org/springframework/aop/target/HotSwappableTargetSource.html). The challenge was how do we decide for which client we need to process something. The application has urls like http://www.ourcomp.com/client1 and http://www.ourcomp.com/client2 etc. We created a filter which extracted the client1 part from the URL and put that in a `ContextHolder` (which is a `ThreadLocal`).

```java
public abstract class ContextHolder {

  private static final ThreadLocal holder = new ThreadLocal();

  public static void setContext(String context) {
    LoggerFactory.getLogger(ContextHolder.class).debug("context set '{}'", context);
    holder.set(context);
  }

  public static String getContext() {
    return (String) holder.get();
  }
}
```

Now we had the context to use in a property we could retrieve anywhere. So next we took the idea of the `HotSwappableTargetSource` and adapted it for our situation. We created a `ContextSwappableTargetSource`, we need to create it with the `targetClass` is provides (in our case a `javax.sql.DataSource`) and with a map of `DataSource`s to use. The keys in the map correspond to the context (or clientnames) used in the url.

```java
/**
* TargetSource which returns the correct target based on the current context set in the {@link biz.deinum.springframework.core.ContextHolder}.
* If no context is found a {@link TargetLookupFailureException} is thrown or the <code>defaultTarget</code> is returned
* , depending on the setting of the alwaysReturnTarget property (default is false);
*
* @author M. Deinum
* @version 1.0
* @see ContextHolder
* @see TargetSource
*/
public class ContextSwappableTargetSource implements TargetSource, InitializingBean {
  private final Logger logger = LoggerFactory.getLogger(ContextSwappableTargetSource.class);
  private Map targets = Collections.synchronizedMap(new HashMap());
  private Class targetClass;
  private boolean alwaysReturnTarget = false;
  private Object defaultTarget;

  /**
  * Constructor for the {@link ContextSwappableTargetSource} class. It takes a
  * Class as a parameter.
  *
  * @param targetClass The Class which this TargetSource represents.
  */
  public ContextSwappableTargetSource(Class targetClass) {
    super();
    this.targetClass=targetClass;
  }

  /**
  * Locate and return the sessionfactory for the current context.
  *
  * First we lookup the context name from the {@link ContextHolder}
  * this context name is used to lookup the desired target. When none
  * is found we return the default target.
  *
  * If the targetClass is of a invalid type we throw a {@link BeanNotOfRequiredTypeException}
  *
  * @see ContextHolder
  */
  public Object getTarget() throws Exception {
    // Determine the current context name from theclass that holds the
    // context name for the current thread.
    String contextName = ContextHolder.getContext();
    logger.debug("Current context: '{}'", contextName);

    Object target = targets.get(contextName);
    if (target == null && alwaysReturnTarget) {
      logger.debug("Return default target for context '{}'", contextName);
      target = defaultTarget;
    } else if (target == null && !alwaysReturnTarget){
      logger.error("Cannot locate a target of type '{}' for context '{}'", targetClass.getName(), contextName);
      throw new TargetLookupFailureException("Cannot locate a target for context '"+contextName+"'");
    }

    if (!targetClass.isAssignableFrom(target.getClass())) {
      throw new TargetLookupFailureException("The target for '"+contextName+"' is not of the required type." + "Expected '"+targetClass.getName()+"' and got '"+target.getClass().getName()+"'");
    }
    return target;

  }

  public final Class getTargetClass() {
    return targetClass;
  }

  public final boolean isStatic() {
    return false;
  }

  public void releaseTarget(Object arg0) throws Exception {}

  public final void afterPropertiesSet() throws Exception {
    Assert.notNull(targetClass, "TargetClass property must be set!");

    if (alwaysReturnTarget && defaultTarget == null) {
      throw new IllegalStateException("The defaultTarget property is null, while alwaysReturnTarget is set to true. " + "When alwaysReturnTarget is set to true a defaultTarget must be set!");
    }
  }

  public final void setAlwaysReturnTarget(final boolean alwaysReturnTarget) {
    this.alwaysReturnTarget=alwaysReturnTarget;
  }

  public final void setDefaultTarget(final Object defaultTarget) {
    this.defaultTarget=defaultTarget;
  }

  public final void setTargets(final Map targets) {
    this.targets.clear();
    this.targets.putAll(targets);
  }
}
```

All the classes are in place, now we only needed to wire things up in our application context and we should be good to go. First we configure the datasources.

Next we need to setup the `ContextSwappableTargetSource`, the key in the map is the value which is going to be set in the `ContextHolder`. In a web application this value could be set by a ServletFilter on each request. The `TargetSource` is wrapped in a `ProxyFactoryBean` so a `Proxy` will be created for the `ContextSwappableTargetSource`.

And that is it. Now inject the `datasourceTargetSource` into a `JdbcTemplate`s datasource property and you are good to go.

At every request the context is set in the `ContextHolder`. Then at every action on the `JdbcTemplate` the `getTarget` method on the `ContextSwappableTargetSource` is called returning the real and correct datasource instance for that request.

In this example we used it to dynamically replace `DataSource` instances but this can work with in theory every object. We have also succesfully used it with multiple hibernate `SessionFactory` instances.

<del>The source code (as available on posttime) can be found [here](http://www.deinum.biz/2007/01/05/one-application-per-client-database/contextswappabletargetsourcezip/). Or point yuor favorite SVN client to [http://bespring.googlecode.com/svn/trunk/](http://bespring.googlecode.com/svn/trunk/)</del>. I submitted this to the Spring framework in [JIRA issue SPR-3014](http://opensource.atlassian.com/projects/spring/browse/SPR-3014)

In the meantime the code has moved to [GitHub](https://github.com/mdeinum/spring-utils), feel free to fork/use it.
