---
layout: post
date: 2018-03-26 11:30
title: Understanding XPages Part 1
tags: xpages blog
---

## How is XPages related to JSF and why is that important?

JSF is a standard Java EE tool for dynamically generating websites using servlets. XPages extends JSF with UI Components and Java libraries that allow the presentation of No-SQL (Domino) data to the browser. It can be very useful to understand JSF in order to understand how to leverage XPages rather than fighting against it. There is a lot of complexity to JSF, but much of it can be ignored if you are using it the right way.

## Basic JSF Components

* The page template
  * Written in XML
  * Parsed during the build and turned into Java classes
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
  
Each of these topics can be explored in great depth and complexity, but some basic understanding of each will prevent a lot of trouble.