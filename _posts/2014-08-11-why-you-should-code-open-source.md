---
date: 2014-08-11 17:40:07+00:00
layout: post
title: Why you should code open source
categories:
- Allgemein
- Thoughts
tags:
- benefits
- open developement
- opensource
---

There are a lot of articles about using open source in your company, what are
the benefits, what should you do, etc. etc. This blog post isn't. It's about your personal
and very egoistic benefits of coding open source.

Note that I believe very strongly in open source, but sometimes you get people
to do the "right" thing for the "wrong" reasons. I don't think the following reasons
are "wrong", but are not my personal top motivations for doing open source.

<!-- more -->


## Learn how to read code


In my personal experience I spent only 10-20% percent of my time actually writing
code. The rest I use for reading code for code reviews, learning, understanding or
finding bugs. Given this distribution of my time, a way to actually get my work
done faster is to read code faster.

Contributing to an open source project improves your code reading skills
every time as have to


### Understand a code base


You learn how to understand a subsystem of a code base, e.g. if you want
to extend it for your purpose. You'll get experienced in drawing the architecture
of the project in front of your inner eye and which systems you have to
understand in detail and which ones not. Developers which are hired
directly after finishing university often feel the need to understand the
whole architecture completely to change something. This should only
be true for bad, tightly coupled, unmodularized spaghetti architectures.

This ability is so basic and yet overlooked. You will code with more
confidence and joy if you are able to focus on the problem itself
and not the "what does this actually do".


### Separate style from content


There are a lot of different codes styles, often revealing the "native"
language of the code author. Coding in open source projects will
improve your skill to read those "accents". Even more you may learn
some new ways of doing things.

And sometimes you see really awful formatted code and
you need to remove all the clutter in your head to see the
shiny core.

```java
public boolean check()
{
   if(x.equals(y))
   {
      if(y.equals(z))
      {
        return true;
      }
      else
      {
        return false;
      }
   }
   else
   {
       return false;
   }
}
```


This is nice, when you write it on a block to identify the core problem,
but not to translate it literally to code. This is nothing but

```java
public boolean check() {
   return x.equals(y) && y.equals(z);
}
```

### Understand and apply patterns


Sometimes design patterns are obvious, e.g. encoded in the
class or method name. However this is not always the case
and you learn to spot patterns in code when they are not so
obvious, maybe because the author used it by accident or
a naming scheme would distract from the actual code.


### Implement isolated fixes/features


Nobody likes to review more lines of code than absolutely necessary.
Contributing means you keep your changes to a minimum if you want
your pull request being accepted. You will be asked to provide tests
and docs. And this will get more and more routine, as you will get
positive feedback each time you provide a well formed pull request
and will keep this practice in your daily job.


## Build a reputation


In the end we as developers want to be recognised for our code
quality and our abilities to write good software. This is an illusion as
HR Managers are most of the time HR guys and not developers.
However being an activate part of an open source community
build a reputation as other developers know you which may work
at the company you apply. And those people can value your work
and quality and may be able to influence the HR office. Or they
even offer you a job by themselves because you already have showed
your skills.


## Get Inspired


Most none devs think that coding is an uncreative activity. Well, you know
that couldn't be more far from the truth. Like artist we have to do routine jobs
to get started. But like painting a picture, solving a problem needs a lot of
creativity. In my personal opinion to be as creative as possible you need
an environment as diverse as possible ( this is my main motivation to get
more woman in IT. We have only 50% of the possible diversity if we only
have men in IT ).

Open Source projects are a great pool for diversity. People from all
around the world are working at one peace of code. You only need
an open mind and the will to learn and teach. "Beginners mind" is
one of the principles in this great article "10 rules being a Zen programmer".
You will get a lot of inspiration to solve problems, which will help you
for all of your future coding problems.

Contributing to great libraries means working with great programmers.
Image you work together with Albert Einstein on one of his papers,
just because you can.
