---
layout: post
date: 2018-03-29 16:00
title: Understanding XPages Part 4
tags: xpages jsf
published: false
---
## Expression Language

Now that we have established a baseline familiarity with Managed Beans, let's dig into how we use and interact with them. Expression Language generally takes the form of `#|${BeanName.property|method}` or it can be expressed as `#|${BeanName[index|'key']}`. The first character indicates whether the value binding is dynamic (`#` - value is computed every time the expression is evaluaged) or static (`$` - value is computed once at page load and never changes. Some properties must be one or the other, but many accept either. BeanName is self-explanatory. Property is either used as a key for `Map` and `DataObject` objects, or it is converted to a getter. `property` becomes `getProperty()` or `isProperty()` (if `boolean`).

<!-- more -->

These statements are known as value bindings. If a value is submitted and there is a setter, the bean is updated with the new value. When the value is parsed as output (either during render or some other computation) it calls the getter and returns the result. If the bean itself does not exist, it is created and then the getter/setter is invoked. Here are a few quick examples.

### Examples

```xml
<xp:text value="#{User.firstName}" />
```
We can use two in the same statement:

```xml
<xp:text value="#{User.firstName} #{User.lastName}" />
```
It can be a two-way binding:

```xml
<xp:inputText value="#{Project.subject}" />
```
It can call a method:

```xml
<xp:eventHandler
	event="onclick"
	submit="true"
	action="#{User.startProject}">
</xp:eventHandler>
```

Server-side Javascript (SSJS) *is* expression language (but the way it works behind the scenes is vastly more complicated):

```xml
<xp:text value="#{javascript:return User.getLastName() + ', ' + User.getFirstName();" />
```

I'll dive further into SSJS on another post, but for now just be aware that non-SSJS EL is preferred to SSJS *when practical*.

You can bind a value directly to a scope bean:

```xml
<xp:link text="User Profile" value="#{sessionScope.profileUrl}" />
```

### Other things you can and can't do with EL

Expression language can be chained. e.g. `#{User.cases.openCount}` will call the `getOpenCount()` method on the object returned by `getCases()` (presuming it has such a method, otherwise an error will be thrown).

Expression language can perform limited conditional statements such at `styleClass="#{Project.status eq 'overdue' ? 'red' : 'green'}"`. The full list of operators can be [found here](https://docs.oracle.com/javaee/6/tutorial/doc/bnaik.html). You cannot use `<` and `>` because those have a specific meaning in xml, but I believe you can substitute `&amp;lt;` and `&amp;gt;`. Importantly, you cannot do string concatenation within an EL statement, but you can concatenate two separate statements.

Expression language - at least the version used in XPages - does not support passing parameters. There is a relatively easy hack around this, though (which I will get into in another post).