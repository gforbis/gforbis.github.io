---
layout: page
title: Example faces-config.xml
permalink: /faces-config-1/
---
```xml
<faces-config>
  <managed-bean>
    <managed-bean-name>AppConfig</managed-bean-name>
    <managed-bean-class>com.example.ApplicationBean</managed-bean-class>
    <managed-bean-scope>application</managed-bean-scope>
  </managed-bean>
  <managed-bean>
    <managed-bean-name>User</managed-bean-name>
    <managed-bean-class>com.example.CurrentUser</managed-bean-class>
    <managed-bean-scope>session</managed-bean-scope>
  </managed-bean>
  <managed-bean>
    <managed-bean-name>ProjectController</managed-bean-name>
    <managed-bean-class>com.example.ProjectViewController</managed-bean-class>
    <managed-bean-scope>view</managed-bean-scope>
  </managed-bean>
  <managed-bean>
    <managed-bean-name>ProjectService</managed-bean-name>
    <managed-bean-class>com.example.ProjectService</managed-bean-class>
    <managed-bean-scope>request</managed-bean-scope>
  </managed-bean>
</faces-config>
```