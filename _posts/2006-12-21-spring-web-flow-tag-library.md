---
author: mdeinum
comments: true
date: 2006-12-21 09:58:27+00:00
layout: post
link: https://mdeinum.wordpress.com/2006/12/21/spring-web-flow-tag-library/
slug: spring-web-flow-tag-library
title: Spring Web Flow Tag Library
wordpress_id: 17
categories:
- Java
- Spring Web Flow
tags:
- Spring
- tag library
- tags
- web flow
---

A while ago we started to use [Spring Web Flow](http://opensource.atlassian.com/confluence/spring/display/WEBFLOW/Home). We needed to convert our old WizardForms and Multi `SimpleFormController` screens to the Spring Web Flow ones. After converting about three jsp's I got fed up with the hidden fields, the submit buttons with the specified name. Generating urls with a `flowExecutionKey` and `eventId` was even worse. Also after making al those typos in `flowExecutionKey` I decided to create a taglibrary which can write different HTML tags needed in Spring Web Flow.
<!-- more -->
So after experimenting a bit I wrote three tags:

* `FlowExecutionKeyTag`: Writes hidden field with the `flowExecutionKey`
* `UrlTag`: Writes a url containing the `flowExecutionKey` and `eventId`
* `SubmitTag`: Writes a submit button and optional a hidden `flowExecutionKey` field

**`FlowExecutionKeyTag`**
```xml
<swf:flowexecutionkey/>
```
will generate


```xml
<input type="hidden" name="_flowExecutionKey" value="<current_flowexecutionkey>">
```

**UrlTag**
```xml
<swf:url url="somelink.flow" eventId="read" paramName="someId" value="${someObject.id}">Link</swf:url></pre>
```
will generate



```xml
<a href="somelink.flow?_flowExectionKey=<current_flowexecutionkey>&_eventId=read&someId=123">Link</a>
```

**SubmitTag**
```xml
<swf:submit eventId="save" value="Save"/>
```
will generate

```xml
<input type="submit" value="Save" name="_eventId_save">
```

There are 2 JIRA issues concerning Web Flow Tags, I attached this code to one of them.

* [SWF-87](http://opensource.atlassian.com/projects/spring/browse/SWF-87)
* [SWF-116](http://opensource.atlassian.com/projects/spring/browse/SWF-116)

The sources can also be downloaded from [here](http://www.deinum.biz/wp-content/uploads/2006/12/webflowtags.zip) or point your favorite subversion client to the repository at [http://bespring.googlecode.com/svn/.](http://bespring.googlecode.com/svn/)
