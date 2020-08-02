---
date: 2014-06-23 21:23:04+00:00
title: Open Source Projects - Between accepting and rejecting pull request
excerpt: Being a commiter comes with a lot of responsibilities. You are responsible for the code quality, supporting your community, encouraging people to contribute to your project and of course providing an awesome open source product
categories:
- open source
- oss
---

Lately I have done a lot work for the [sbt-native-packager](https://github.com/sbt/sbt-native-packager) project. Being a
commiter comes with a lot of responsibilities. You are responsible for the code quality, supporting your community,
encouraging people to contribute to your project and of course providing an awesome open
source product.

Most of the open source commiters will probably start out as a contributor by providing pull
requests fixing bugs or adding new features. From this side it looks rather simple, the
project maintainer probably knows his/her domain and the code well enough to make
a good judgement. Right?

This is not always the case. The bigger the projects get, the smaller the chance gets
one contributor alone can merge your pull requests. However there's a lot you can do
to make things easier! I'm really glad a lot of contributors already do a lot of these things,
but I wanted to write down my experience.


## Provide tests


This is obvious, right? However tests are so much more then just _proving it works_ or
_proving it's fixed_. Tests are like documenation for the maintainers. They can see _how
_the new features works or _what_ caused the bug. Furthermore it gives the maintainer
confidence to work on this feature/bug fix himself as there's already a test which
checks his work.


## Provide documentation


If you add a new feature then add a minimal documentation. A few sentence _what does this_,
_how can I use it_ and _why should I use it_ are enough. It makes life a lot easier for maintainers
judging your pull request, because they can try it out very easily themselves without going
through all of your code at first.


## Be ready for changes


To maintain a healthy code base with a lot of contributors is a challenge. So if you decide
to contribute to an open source project try to stick to the style which is already applied in
the repository. This applies to the high abstraction level to the deep bottom of low level code.
And if you don't then be prepared to change your code as the maintainers have to make sure
the code can be easily understood by everybody else. Sometimes it's hard not to take this
personally and we try to be very polite. However sometimes corrections are necessary.

There's an easy way to avoid all of this...


## Small commits, early pull requests


Start small and ask early. Write comments in your code, use the awesome tooling most of
the code hosting sites provide like discussions or in-code-comments. Providing a base for
discussions is IMHO the best way to get things done. You can discuss what's good and
bad, if the approach is correct or not. You avoid a lot work, which might  not be useful
or out of scope and the maintainers don't have to feel bad about rejecting a lot of work.


## Tell us more!


A lot of open source projects where created for a specific need, but the nature of an
open source project leads sometimes to an extension of this specific need and you
add more features. Tell us what you do with it! The maintainers (hopefully) love there
project and are amazed by the things you can do with it. Write blog posts, tweets
or stackoverflow discussions to show your case.
