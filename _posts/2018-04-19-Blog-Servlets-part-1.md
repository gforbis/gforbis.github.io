---
layout: post
date: 2018-04-19 17:00
title: Servlets in XPages
tags: java
published: false
---
## Servlets in XPages

During the build process, everything in XPages is compiled to Java classes - specifically to a servlet. When you /MyApp.nsf/Example.xsp, your request is answered by a servlet and /MyApp.nsf/Example2.xsp is answered by a different servlet. So the next logical question is, how can we ever skip the XPage part and just write a servlet... and should we?

To answer the last question first, if you have ever written an XAgent (a non-rendering XPage that performs some action on the server) the answer is a resounding _yes_. Answering the first one requires some boilerplate, but once that is created, writing a servlet is _easy_.

### Setup

First, you'll need to add the following to your buildPath (in the Project Explorer view, right-click your project and select Build Path -> Configure Build Path -> Add Library): Notes/framework/shared/eclipse/plugins/com.ibm.domino.xsp.adapter_[version]/lwpd.domino.adapter.jar

Next, we'll need to create a Java class to listen for requests and provide a servlet of our choosing. First, we create some boilerplate that can be used everywhere:

```java
package servlets;

import java.util.Map;
import java.util.HashMap;
import javax.servlet.Servlet;
import javax.servlet.ServletException;
import com.ibm.designer.runtime.domino.adapter.ComponentModule;
import com.ibm.designer.runtime.domino.adapter.IServletFactory;
import com.ibm.designer.runtime.domino.adapter.ServletMatch;
import com.ibm.domino.xsp.module.nsf.NSFComponentModule;

public abstract class ServletFactory implements IServletFactory {
	private static final String PATTERN = "/xsp/((%1$s)(?:$|/)((?:\\w*)(?:\\b)(?:$|/)(?:.*)))";
	private NSFComponentModule module = null;
	private final Map<String, Class<? extends CustomFacesServlet>> servletClasses = new HashMap<String, Class<? extends CustomFacesServlet>>();
	private final Map<String, Pattern> servletPatterns = new HashMap<String, Pattern>();
	
	final public void init(ComponentModule module) {
		this.module = (NSFComponentModule) module;
		registerServlets();
	}
	
	final protected NSFComponentModule getModule() {
		return module;
	}
	
	private final Servlet getServlet(final String key, final String name) throws ServletException {
		return this.module.createServlet(servletClasses.get(key), name, null);
	}
	
	final protected void registerServlet(String name, Class<? extends CustomFacesServlet> servletClass){
		servletClasses.put(name, servletClass);
		servletPatterns.put(name, Pattern.compile(String.format(PATTERN, name))); 
	}
	
	abstract protected void registerServlets();
	
	final public ServletMatch getServletMatch(String contextPath, String path) throws ServletException {
		for (Map.Entry<String, Pattern> entry : servletPatterns.entrySet()) {
			ServletMatcher matcher = entry.getValue().matcher(path);
			if (matcher.matches()) {
				Servlet servlet = null;
				try {
					servlet = getServlet(entry.getKey(), matcher.group(2));
				} catch (Exception e) {
					System.err.println("Error creating servlet: " + e);
					e.printStackTrace();
				}
				if (null != servlet) {
					return new ServletMatch(servlet, matcher.group(2), matcher.group(3));
				}
			}
		}
		return null;
	}
}
```
This is a repository of the servlet classes we want to provide. They will be provided at /MyApp.nsf/xsp/myServletPath. That ugly regex pattern just ensures we are matching the URI to the correct servlet. .group(1) is everything after the /xsp/. .group(2) is the servlet path. .group(3) is the remainer of the URI and could include URL parameters or further directives. 

Next we extend it:

```java
package app.servlets;

import servlets.ServletFactory;

public class MyServletFactory extends ServletFactorn {
	@Override
	protected void registerServlets() {
		registerServlet("test", TestServlet.class);
	}
}
```
Second, right-click on your source folder (probably Code/Java or WebContent/WEB-INF/src) and add a new folder named META-INF/services. Next, right-click in this folder and create a new file named com.ibm.xsp.adapter.servletFactory.