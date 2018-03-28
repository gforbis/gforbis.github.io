---
layout: post
date: 2018-03-26 11:30
title: Understanding XPages Part 1
tags: xpages jsf
published: true
---
<nav markdown=1>
[Part 1](/Understanding-XPages-part-1)
[Part 2](/Understanding-XPages-part-2)
[Part 3](/Understanding-XPages-part-3)
</nav>
## How is XPages related to JSF and why is that important?

JSF is a standard Java EE tool for dynamically generating websites using servlets. XPages extends JSF with UI Components and Java libraries that allow the presentation of No-SQL (Domino) data to the browser. It can be very useful to understand JSF in order to understand how to leverage XPages rather than fighting against it. There is a lot of complexity to JSF, but you can get a lot out of it without plumbing its full depth.
<!-- more -->
## Basic JSF Components

* The page template
  * Written in XML
  * Parsed during the build and turned into Java servlets
* faces-config.xml
  * Managed bean and component registration
  * Dependency injection
* JSF Lifecycle
* Managed beans
  * Serializable
  * Zero-argument constructor
  * Uses getters/setters or conforms to a supported interface
* Scopes
  * Application
  * Session
  * View
  * Request
  
A basic understanding of each of these will prevent a lot of confusion when working in XPages.

### JSF Lifecycle

When a browser hits a JSF application, the following series of events are triggered:

1. **Restore view** restores an existing ViewHandler object (if any) or creates a new one on navigation to a new page.
  * A ViewHandler is the thing that puts the models together that will eventually become the web page
  * Before page load and after page load events are triggered only when the ViewHandler is created - not when it is restored.
2. **Apply requests** goes through the browser request and extracts things like URL parameters and Form POST data and constructs submitted value objects.
  * Converters are applied to convert the text values from the request into numbers and dates
  * If a converter fails to understand the submitted value, processing aborts and the browser response is completed and sent.
3. **Process validations** takes these submitted value objects and applies any server-side validation rules.  
  * Unless you have complex validation rules, you shouldn't need to write any code for this phase.
  * If you are validating field A based on the value of field B, make sure you examine the submitted value and not the actual value - which will cause confusing and inconsistent results because sometimes field B is unchanged and the validator works correctly, and other times it is changed, and the validator appears to lag behind.
  * Only examine submitted values for validation. Any other use is probably creating more complex problems than it solves.
  * If any rules are violated, processing aborts and the response is completed and sent.
4. **Update model** is where these submitted value objects have their converted and validated data placed into the actual data models.  
  * Before this process the submitted value objects exist and the value of the actual object is unchanged.
  * After this, submitted value objects are null and the values stored in the actual object are the new values.
  * Again, interjecting your own code into this process is likely to create more problems than it solves.
5. **Process application** applies business logic to these updated models. This is where most of the actual code you write runs.
6. **Render response** takes these updated models and constructs the HTML (usually) to send back to the browser. Once the response is rendered, it is sent and the lifetime of any request-scoped beans is complete.

#### Considerations

Bean values can (and usually will) be retrieved several times during this process. It is important to only do expensive computations once and cache the value in the request scope for subsequent retrieval. You don't want to do a full-text search 7 times per page load. That advice notwithstanding, be mindful of premature optimization.
