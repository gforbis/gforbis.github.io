---
layout: post
date: 2018-03-27 09:00
title: Understanding XPages Part 2
tags: xpages jsf
published: true
---
## Managed Beans

If you aren't familiar with beans, then you need to know there are three things that define a bean compared to a plain old Java object (POJO).

1. They have a zero-argument constructor
2. They are serializable
3. They use accessor methods (getters and setters)

The question is, why are these things important to us in JSF?
<!-- more -->
### Zero-argument constructor

JSF automatically creates objects for you when you need them by using the `Bean.class.newInstance()` method. This is really useful because it means we don't have to worry about it. But it also means we can't pass arguments to the constructor. Use dependency injection and lazy instantiation whenever possible. Be extremely wary of accessing other beans in your constructor through static methods. It can lead to circular dependencies and other difficult-to-troubleshoot bugs.

### Serializable

What does serializable mean? It means an object in memory can be turned into a representation of the current state, stored or transmitted, and then used to create a new object that is identical to the original. Here is an example:

```java
public class Person {
  private String firstName;
  private String lastName;
  private Date birthday;
}
```

This can be serialized into a JSON string like so:

```javascript
{
  "firstName":"John",
  "lastName":"Smith",
  "birthday":"1990/04/28"
}
```

And given that JSON string, we could rebuild the Person object a year later with 100% fidelity. Now consider an object that contains an open File handle or OutputStream - those are objects which cannot be serialized. We indicate an object is serializable through the `Serializable` interface. This adds no code to our object and we can declare it like so: `public class Person implements Serializable`. If you don't do this for a bean, you will get intermittent `NotSerializableException` errors from JSF. Intermittent because JSF handles serialization automatically for you and you only get the error if it needs to serialize an object and cannot.

### Accessor methods

This is a "soft" requirement because Expression Language (EL - which I will get into in more depth in another post) also recognizes the `Map` and `List` interfaces (and XPages recognizes `DataObject`). The reason this is important is because this is how the template (XPage) uses EL to talk to beans. In your XML you can place a component and write `value="#{Bean.value}"` and this is translated to the following sequence of events:

1. Look for an existing bean.
2. If it doesn't exist, create it using `newInstance()` and store it in the scope indicated in faces-config.xml.
3. If that object is a supported interface such as `Map`, use that interface to get the value (e.g. `get("value")`).
4. If that object is *not* a supported interface, convert to camelCase and use the accessor (e.g. `getValue()`).

#### Considerations

1. If a bean has a getter, but no setter, it will be treated like a read-only field.
2. Boolean values will be retrieved via `isValue()` rather than `getValue()`.
3. Values returned can be beans, maps, etc and chained. (e.g. `#{Bean.owner.name}` will be turned into `Bean.getOwner().getName()`).
4. Array notation can also be used (e.g. `#{Bean['value']}`).

{% include undrstndxpgnav.html %}