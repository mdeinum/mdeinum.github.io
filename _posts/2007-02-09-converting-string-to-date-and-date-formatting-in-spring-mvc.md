---
author: mdeinum
comments: true
date: 2007-02-09 09:24:04+00:00
layout: post
link: https://mdeinum.wordpress.com/2007/02/09/converting-string-to-date-and-date-formatting-in-spring-mvc/
slug: converting-string-to-date-and-date-formatting-in-spring-mvc
title: Converting String to Date and Date-formatting in Spring-MVC
wordpress_id: 16
categories:
- Java
- Spring
tags:
- spring mvc
---

This week alone I answered the question about date formatting and how to bind Strings to Objects multiple times. If I would get an euro/dollar for everytime I gave the same answer. So I figured maybe it is time to create simple example to show how it is done in Spring. (The sources used are included in the [propertyeditors.zip](http://www.deinum.biz/wp-content/uploads/2007/02/propertyeditors.zip) attached to this post, rename to .war to deploy it in tomcat or jetty (tested in Jetty 6.0.2))
<!-- more -->
For this project we will only use a WebApplicationContext so the only thing we need to do is configure the Spring DispatcherServlet.

```xml
Step-By-Step
sbs
org.springframework.web.servlet.DispatcherServlet
1

sbs
*.do

index.jsp

```

As you see we mapped all the *.do to the dispatcher servlet. Problem with the servlet spec is that the welcome page has to be a physical file. So the index.jsp only does a redirect to index.do. The Servlet is named **sbs**. As with the Spring convention we then need to create **sbs-servlet.xml** to get the context loaded. For this example I use the default BeanNameUrlHandlerMapping, which resolve an URI to a Controller.

We also need something to resolve our views (a ViewResolver) so I configured an InternalResourceViewResolver, which looks up jsp in the WEB-INF/jsp directory.

[sourcode language='xml']
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.0.xsd">

[/code]Well that is it, configuration wise. We have a controller, with a form and a validator, we have 2 views (entry and success).

The controller isn't that big of a deal, it extends simpleformcontroller and only implements the initBinder method. For binding to other objects then Strings or primitives we need a PropertyEditor. Luckily for us Spring comes shipped with some PropertyEditors. For this example we will use the CustomDatePropertyEditor. The PropertyEditor will convert a String to a Date and vice-versa.

```java
public class DateEntryController extends SimpleFormController {
  protected void initBinder(HttpServletRequest request, ServletRequestDataBinder binder) throws Exception {
    SimpleDateFormat dateFormat = new SimpleDateFormat("dd-MM-yyyy");
    CustomDateEditor editor = new CustomDateEditor(dateFormat, true);
    binder.registerCustomEditor(Date.class, editor);
  }
}
```

The [initBinder](http://www.springframework.org/docs/api/org/springframework/web/servlet/mvc/BaseCommandController.html#initBinder(javax.servlet.http.HttpServletRequest,%20org.springframework.web.bind.ServletRequestDataBinder)) method as mentioned here, registers a [CustomDateEditor](http://www.springframework.org/docs/api/org/springframework/beans/propertyeditors/CustomDateEditor.html) which takes a text in the format (dd-MM-yyyy) and transforms that into a valid date.

the line '_binder.registerCustomEditor(Date.class, editor);_' tells the binder to register this propertyeditor for all the Date properties on the command object. If there is a need for multiple property editors for the same type you can also specify the name of the property on the command object.

The Validator is a simple one (just to generate some errors and show that valid dates are translated back into strings).

```java
public class DateEntryValidator implements Validator {

  public boolean supports(Class clazz) {
    return DateEntryFormObject.class.isAssignableFrom(clazz);
  }

  public void validate(Object target, Errors errors) {
    DateEntryFormObject commandObject = (DateEntryFormObject) target;
    Date startDate = commandObject.getStartDate();
    Date endDate = commandObject.getEndDate();

    if (startDate == null) {
      errors.rejectValue("startDate", "required", "Starting Date is required.");
    }

    if (startDate != null && endDate != null && endDate.before(startDate)) {
      errors.rejectValue("endDate", "notbefore.startdate", "End date cannot be before start date.");
    }
  }
}
```

Well that's all there is. The only remaining things are the 2 jsps and the command object.

```java
public class DateEntryFormObject implements Serializable {

  private Date startDate;
  private Date endDate;

  public Date getEndDate() {
    return endDate;
  }

  public void setEndDate(Date endDate) {
    this.endDate = endDate;
  }

  public Date getStartDate() {
    return startDate;
  }

  public void setStartDate(Date startDate) {
    this.startDate = startDate;
  }
}
```

entry.jsp
```xml
<%@taglib prefix="form" uri="http://www.springframework.org/tags/form">
<table >
  <tr >

<td colspan="3" >Enter dates in format (dd-MM-yyyy)
</td>
  </tr>
  <tr >

<td >Start Date:
</td>

<td >
</td>

<td >
</td>
  </tr>
  <tr >

<td >End Date:
</td>

<td >
</td>

<td >
</td>
  </tr>
  <tr >

<td colspan="3" >
</td>
  </tr>
</table>
```
```xml
<%@taglib prefix="form" uri="http://www.springframework.org/tags/form">
<table >
  <tr >

<td colspan="2" >

## Dates entered

</td>
  </tr>
  <tr >

<td >Start Date:
</td>

<td >
</td>
  </tr>
  <tr >

<td >End Date:
</td>

<td >
</td>
  </tr>
</table>
```
There is just one important thing to remember PropertyEditors work only for the command object, not for reference data, or objects stored somewhere else. For those to format correctly you will need the good ol' fmt tags.
