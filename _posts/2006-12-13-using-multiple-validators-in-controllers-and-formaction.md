---
author: mdeinum
comments: true
date: 2006-12-13 13:38:56+00:00
layout: post
link: https://mdeinum.wordpress.com/2006/12/13/using-multiple-validators-in-controllers-and-formaction/
slug: using-multiple-validators-in-controllers-and-formaction
title: Using multiple validators in Controllers and FormAction
wordpress_id: 8
categories:
- Java
- Spring
tags:
- spring mvc
- web flow
---

On a job I did recently we did a lot of refactoring the old (web) application, they used an abundance of (Web) Frameworks, we reduced it to 1 (well actually 2 if you count Spring Web Flow :) ).

They already had a lot of controllers and validators build and also somewhere some custom validation logic in the desired classes. One thing I noticed is that they had a few command objects which had an `emailaddress` or `telephone` number. For each of those objects they also wrote a suitable `Validator`. Copying and pasting all the logic concering emailaddress validation each time, or even worse reinvented the logic.
<!-- more -->
What I decided to do is to put the logic concerning the emailaddress validation into 1 `Validator`. Problem is you can only have 1 `Validator` on a `Controller` of `FormAction`. A bit problematic, however I couldn't imagine that we where the first ones to run into this. So off to the [Spring Forum](http://forum.springframework.org) after some search we found the [`CompositeValidator`](http://forum.springframework.org/showpost.php?p=26308&postcount=3) which we copied (Thanks to Colin Yates) for this.

```java
public final class CompoundValidator implements Validator {

  private final Validator[] validators;

  public CompoundValidator(final Validator[] validators) {
    super();
    this.validators=validators;
  }

  /**
  * Will return true if this class is in the specified map.
  */
  public boolean supports(final Class clazz) {
    for (Validator v : validators) {
      if (v.supports(clazz)) {
        return true;
      }
    }
    return false;
  }

  /**
  * Validate the specified object using the validator registered for the object's class.
  */
  public void validate(final Object obj, final Errors errors) {
    for (Validator v: validators) {
      if (v.supports(obj.getClass())) {
        v.validate(obj, errors);
      }
    }
  }
}
```
Now we have the solution for registering multiple validators for a `Controller` or `FormAction` you can create a `CompositeValidator` which wraps other validators.

Still we had all the emailaddress validation cluttered around, another problem to solve was in each command object the field containing the `emailaddress` was named differently i.e. `email`, `emailadres`, `emailaddress` etc. So we need to design something which could map classes to the field name needed. I couldn't find something on the excellent Spring Forum or with Google so it was back to the coding board :) . I came up with the `AbstractClassMappingValidator`, this allows you to specify a Class and the desired field to check for that Class.

```java
public abstract class AbstractClassMappingValidator implements Validator {    /** Prefix Strings which are added to the field names */

  protected static final String REQUIRED = "required";
  protected static final String INVALID = "invalid";

  private Map mappings = new HashMap();

  protected boolean allowEmpty = false;

  public final boolean supports(Class clazz) {
    for (Class targetClazz: mappings.keySet()) {
      if (ClassUtils.isAssignable(targetClazz, clazz)) {
        return true;
      }
    }
    return false;
  }

  /**
   * Gets the fieldname from the configured map.
   * @param target
   * @return
   */
  protected final String getFieldName(Object target) {
    for (Class targetClazz: mappings.keySet()) {
      if (ClassUtils.isAssignable(targetClazz, target.getClass())) {
        return mappings.get(targetClazz);
      }
    }
    throw new IllegalStateException("Cannot find fieldname for class " + target.getClass().getName() + ". Class is not compatible with declared types. ["+mappings+"]");
  }

  public final void setMappings(final Map mappings) {
    this.mappings = mappings;
  }

  public final void setAllowEmpty(final boolean allowEmpty) {
    this.allowEmpty = allowEmpty;
  }
}
```
You can configure this validator by setting its `mappings` property. If we have two objects(`BusinessObjectOne` & `BusinessObjectTwo`) with the fields `email` and `emailaddress` we can use the following snippet to configure the validator.
```xml
<bean id="id" class="SomeClassExtendingAbstractClassMappingValidator">
  <property name="mappings">
    <map key-type="java.lang.Class" value-type="java.lang.String">
      <entry key="com.mycompany.objects.BusinessObjectOne" value="email"/>
      <entry key="com.mycompany.objects.BusinessObjectTwo" value="emailAddressNew"/>
      <entry key="com.mycompany.objects.BusinessObjectThree" value="emailAddress"/>
    </map>
  </property>
</bean>
```
Now we have a `CompositeValidator`, we can map classes to fields to validate but we still need to check the emailaddresses against a defined regular expression. So we created a `RegExpValidator` it takes a regular expression and matches the value of the field against this regexp. If a field is required can also be configured.
```java
/**
 * Validator which matches a value against a defined regular expression.
 *
 * First it checks if a given value is not empty, if empty values are allowed
 * this can be switched off. Next the value (if not empty) is checked against
 * the configured regular expression. If it doesn't match the value is rejected.
 *
 * When empty values aren't allowed and the string is empty the errorcode is
 * generated as **required.fieldname**. If the format is invalid the errorcode
 * is generated as **invalid.fieldname**.
 *
 * @author Marten Deinum
 *
 */

public final class RegExpValidator extends AbstractClassMappingValidator {    private String              regexp     = null;

  public void validate(Object target, Errors errors) {
    String field = getFieldName(target);
    String value = (String) errors.getFieldValue(field);
    boolean isEmpty = !StringUtils.hasText(value);

    if (!allowEmpty && isEmpty) {
      ValidationUtils.rejectIfEmptyOrWhitespace(errors, field, REQUIRED + "." + field);
    }

    if (!isEmpty) {
      if (!value.matches(regexp)) {
         errors.rejectValue(field, INVALID + "." + field, new Object[] {field, value}, "{1} is not a valid value for {0}.");
      }
    }
  }

  public void setRegExp(final String regexp) {
    this.regexp = regexp;
  }
}
```
Now we have all the components in place to make the magic work. First we configure the emailaddress validator. When we have done that we can setup the `CompositeValidator` with multiple validators and inject the configured one into a `Controller` or `FormAction`.

Feel free to comment and to use the code, if you make any improvements please let me know everything is welcome :).The code can be found on [code.google.com](http://code.google.com/p/bespring) connect to the subversion repository to get the sources of the validators(projects core and validation).
