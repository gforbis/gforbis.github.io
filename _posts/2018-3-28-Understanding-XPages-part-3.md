---
layout: post
date: 2018-03-28 09:00
title: Understanding XPages Part 3
tags: xpages jsf
published: true
---
## Scopes

Before going too much further in our exploration of JSF, it is important to make sure there is a clear understanding of exactly what scopes are and how they work. Within XPages, there are 4 scopes, which are just `Map<String,Object>` objects to store values. Going from narrowest to broadest they are:
* Request
* View
* Session
* Application

<!-- more -->

### Request

The request map is cleared at the end of every RenderResponse lifecycle event and is local to a particular request. It is useful for storing non-serializable objects and caching the results of expensive lookups. As mentioned earlier, the getters and setters of a bean will often be called multiple times during the JSF lifecycle. So storing results in the request scope and only going to the database if there is no such object already stored will greatly increase performance.

### View

**Make sure you consider the effects of multiple windows/tabs before using the view scope**

The view scope is local to a particular view/page. This map is stored in the `ViewRoot` (unlike the other scope maps which are stored in the `ExternalContext`) which means a new one is created before the page is built - this makes it unsuitable for carrying values from one page to the next. It also will be cleared if a different page is opened in a new tab.

### Session

The session scope persists as long as the user session. This makes it ideal to contain anything specific to the user that only needs to be created once. It can also carry information from one view to the next, but be cautious about storing too much information here because very large session objects can create performance bottlenecks - as memory limitations are hit these objects will frequently be serialized and deserialzed.

### Application

The application scope is shared by all application users. It is created once and then persists for as long as the web server is up. This is ideal for caching configuration settings that require expensive lookups.

#### Considerations

While these are simple maps, JSF uses them internally. Don't issue haphazzard `clear()` commands to reset state.

[Part 1](/Understanding-XPages-part-1)
[Part 2](/Understanding-XPages-part-2)
[Part 3](/Understanding-XPages-part-3)