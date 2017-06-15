---
author: wp_admin
comments: false
date: 2012-05-03 12:58:42+00:00
layout: post
link: http://mukis.de/pages/software-development-at-universities/
slug: software-development-at-universities
title: Software Development at Universities
wordpress_id: 172
categories:
- Thoughts
tags:
- code
- software
- university
---

A lot of frameworks and applications are developed at universities. Some have practical reasons, some are developed to support researching activities or are the main goal of the research process and weren't even planned at the beginning.

Developing is good as students need to gain practices, because without [practice](http://www.gamedev.net/blog/355/entry-2250592-become-a-good-programmer-in-six-really-hard-steps/) you never get a good software developer, engineer or architect. However there are some points I'm missing too often at university.


### Collaboration


I regularly meet people which are in the 4,5 or 6 semester and ask me what a version control system is. Most of them manage their code on only one machine. With the upcoming cloud stores like Dropbox, or Google Drive, people start synchronizing with this repository (which I highly recommend against). In my university there's one course where you have to code together with others. However techniques like code reviews, issue tracking, mailing list and other vcs as SVN aren't mentioned. At the beginning it's hard to learn but pays off in the long run.


### Presentation


I don't want to write documentation, because that's just a little part. As far as I got to know projects from universities the one with the most appealing website, nice and small tutorials, little sample applications have the highest impact. Information science departments can be compared with a foundation like [Eclipse](http://eclipse.org/). The have some projects developed in their name and organize employees, external coworkers, code and public relations. Like the Eclipse Foundation the departments should provide an infrastructure for students and PhDs to present their work on the website, including wiki, issue tracker, code repository, documentation, forum, mailing list and website. If the department is not able to provide such an Infrastructure it should consider renting space for example at [GitHub](https://github.com/), which provides all these features.


### Realworld


University is for research and trying things out. A lot of programmers however love to do fancy stuff without any purpose. Just because "it's cool and it works". If you really aim to achieve something with your project you must have some real world applications that make use of your research results.


### Google before you code


There's almost always someone who tried to code something you want to do. Search for it! Google (Code), GitHub, Bitbucket, Sourceforge or other code repositories. This is especially helpful for small task which aren't really part of your work. The are some rules of thumb I use to pick projects I want to use in my projects



	
  1. **Open Source**
Nothing is more frustrating, when you search for errors, memory leaks or implementation details and can't look inside the source code.

	
  2. **When was the last commit**
I try to avoid projects which are inactive for more than 6 months. 

	
  3. **Does it implement some specification which his well known or widely used**
I think it's highly recommend to choose projects which implement some specification (e.g. OSGi, JPA, JAX-RS,..), because you can easily switch implementation providers. 

	
  4. **Is the licence compatible with my project**
This is necessary, if you like it or not.   


A list of companies which provide a rich set of libraries, mostly for Java, are

	
  * Google (Guava, Commons, Protobuf,...)

	
  * [Typesafe](https://github.com/typesafehub) (Scala, Akka, Config, ...)

	
  * Apache (Commons, Axis2, Camel, Hadoop,...)

	
  * [Twitter](https://github.com/twitter/) (Bootstrap, scala util,...)

	
  * Eclipse (EMF, Sapphire, EclipseLink,...)




There are plenty more of course, this is just what I came up first to my mind. Using libraries makes your code cleaner, easier to maintain and you can focus on the problems your want to solve. 





All this stuff could be summarized as "working more professional". Software development is just too complex to lose time with inefficient work. Even universities are more theoretical, they should motivated students more to form groups and start small software projects on their on to experience software development.
