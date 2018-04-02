---
layout: page
title: Objects.java
permalink: /objectsutil/
---
```java
public class Objects {
  private Objects(){} // Prevent instantiation
  
  public static <T> T requireNonNull(T obj, String message) {
    if (null == obj) {
      throw new NullPointerException(message);
    }
    return obj;
  }
}
```