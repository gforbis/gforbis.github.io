---
layout: post
date: 2018-04-02 13:00
title: Understanding XPages Part 5
tags: xpages jsf
published: true
---
## Faces-Config.xml and Dependency Injection
Hopefully this isn't a new term for you, but here is a brief explanation in case it is. Beans often rely on one another. Your `MyStuff` bean needs your `User` bean; your `User` bean needs your `UserService` bean. Outside of JSF, we would pass these dependencies through the constructor, but that isn't an option due to the zero-argument requirement. So JSF provides another mechanism for initializing beans in faces-config.xml - the managed property. (Side note: newer versions of JSF move this out of faces-config and into class annotations.)

<!-- more -->

[Here is an example faces-config](/faces-config-example-1/). The `User` bean holds information about the current user - their name, permissions, etc. It relies on the `UserService` to provide this information from a database. The `UserService` in turn relies on `AppConfig` to provide the database paths to each database that contributes information about the `User`. So the first questions is how does `UserService` talk to `AppConfig`?

#### Dependency Injection

By adding a managed property to a bean configuration, we can enforce that *immediately after the bean is instantiated* and *before it is provided* these managed properties will be set. These managed properties can include other beans and, if they do, then *that* bean will be provided in the same way.

```xml
<managed-bean>
  <managed-bean-name>UserService</managed-bean-name>
  <managed-bean-class>com.example.UserService</managed-bean-class>
  <managed-bean-scope>request</managed-bean-scope>
  <managed-property>
    <property-name>appConfig</property-name>
    <value>#{AppConfig}</value>
  </managed-property>
</managed-bean>
```

We need to provide a setter and private getter within the `UserService`:

```java
public class UserService {
  private ApplicationBean appConfig = null;
  
  public void setAppConfig(ApplicationBean appConfig) {
    this.appConfig = appConfig;
  }
  
  private ApplicationBean getAppConfig() {
    return Objects.requireNonNull(appConfig, "appConfig not set");
  }
  [...]
}
```
(See [Objects class here]({{ /objects.java/ }}))

Note that the property name is lower case due to the convention that a property named appConfig will be accessed through `getAppConfig()` and `setAppConfig(...)`. This is the preferred way to managed dependencies, but it doesn't work in every case.


#### VariableResolver

The `User` bean needs the `UserService` but there is a problem. If we inject the `UserService` using the above technique, it is only valid for the first request. The `UserService` bean is request scope, so it is destroyed after each request. If the `User` needs to call upon it any time after the first request, an error would be thrown. So JSF helpfully disallows you from setting a request-scoped bean as a managed property in any bean with a larger scope. This is where we need to make user of the `VariableResolver`. I created a [utility class](/facesutil/) which has a `findBean(...)` function that makes use of this technique.

Using this function I can grab `UserService` at any time by calling `Faces.findBean("UserService", UserService.class)`. This can be improved slightly by making it a static getter on the class itself:

```java
public class UserService {
  [...]
  public static UserService get() {
  	return Faces.findBean("UserService", UserService.class);
  }
  [...]
}
```

When using this technique, we want to resolve dependencies lazily - not in the constructor. This avoids circular dependency issues where two beans depend on one another and neither can be instantiated without the other already existing.

#### Why use one over the other ?

Dependency Injection 
* If an object is expensive to create or pass around, dependency injection means it only needs to be done once.
* It prevents a whole class of possible bugs by identifying circular dependencies before they become a troubleshooting nightmare.
* Provides for better separation of concerns

Resolver
* If a smaller-scope object needs to be used by a larger-scope object
* Keeps `faces-config.xml` size down

{% include undrstndxpgnav.html %}