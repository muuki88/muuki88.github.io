---
date: 2010-09-02 18:46:13+00:00
title: Using ISharedImages in plugin.xml
categories:
- Tutorials
tags:
- eclipse
---

Eclipse provides a lot of images for general purpose, e.g. _save, delete, edit, folders,_ etc. You can access these ImageDescriptors via


```java
PlatformUI.getWorkbench().getSharedImages().getImageDescriptor(SharedImages.IMAGE_ID);
```


However there is a more sophisticated way to create Entries in toolbars and/or menus. This is shown in a great tutorial by Lars Vogel [here](http://www.vogella.de/articles/EclipseCommands/article.html). You have now a command and a referencing menu extension. To use the Eclipse SharedImages you just put the ISharedImages constant in the _icon _field, like


```
**icon:** _IMG_ETOOL_SAVEALL_EDIT_
```

It works fine for me in Eclipse 3.6 Helios.  The article is based on [https://bugs.eclipse.org/bugs/show_bug.cgi?id=246224](https://bugs.eclipse.org/bugs/show_bug.cgi?id=246224)
