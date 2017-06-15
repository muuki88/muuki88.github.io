---
author: wp_admin
comments: false
date: 2012-10-25 08:52:30+00:00
layout: post
link: http://mukis.de/pages/pde-target-platform-cache-path/
slug: pde-target-platform-cache-path
title: PDE target platform cache path
wordpress_id: 257
categories:
- Allgemein
- News
tags:
- bundle path
- cache
- clean
- directory
- Eclipse
- eclipse pde
- folder
- OSGi
- pde
- reset
- target platform
- workspace
---

Sometimes you build a broken bundle and publish it on a local [update site](http://www.vogella.com/articles/EclipseTycho/article.html) for some of your colleges. However you're colleges have already [set up their target platform](http://www.vogella.com/articles/EclipseTargetPlatform/article.html) and [Eclipse PDE](http://www.eclipse.org/pde/) cached the bundles. PDE seems to be really smart when it comes to use cached bundles. Deleting and resetting the target platform didn't work for me. So I want to replace the bundle in the cache, but where is the **folder**?


#### Short answer:


**{workspace}/.metadata/.plugins/org.eclipse.pde.core/.bundle_pool/plugins/**


#### How I found it:





	
  1. Go to your workspace

	
  2. **find -name 'bundle.name*' **which results in the caching directory and the place of your file

	
  3. Replace the incorrect bundle in your cache




#### Note:


This is a short hack in development environments. If a release is broken you should realize this before you publish the site. And if it happens, then update the version of your broken bundle an republish, so the newer version is fetched.
