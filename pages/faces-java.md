---
layout: page
title: Faces.java
permalink: /facesutil/
---
```java
public class Faces {
  private Faces(){} // Prevent instantiation
  
  public static <T> findBean(String name, Class<T> type) {
    FacesContext ctx = FacesContext.getCurrentInstance();
    Object obj = ctx.getApplication().getVariableResolver().resolve(ctx, name);
    if (null == obj){
      return null;
    }
    else if (type.isAssignableFrom(obj.getClass()) {
      return (T) obj;
    } else {
      throw new IllegalArgumentException("Unexpected type: " + obj.getClass().getName());
    }
  }
}
```