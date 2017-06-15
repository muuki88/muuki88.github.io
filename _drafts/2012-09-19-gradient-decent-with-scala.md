---
author: wp_admin
comments: false
date: 2012-09-19 12:52:36+00:00
layout: post
link: http://mukis.de/pages/gradient-decent-with-scala/
slug: gradient-decent-with-scala
title: Gradient Decent with Scala
wordpress_id: 245
categories:
- Allgemein
- Tutorials
---

Currently I'm watching a [Scala](http://www.scala-lang.org/) and a [Maschine Learning](http://en.wikipedia.org/wiki/Machine_learning) course on [coursera.org](https://www.coursera.org/) and
wanted to try some simple stuff for myself. I choose [Gradient Decent](http://en.wikipedia.org/wiki/Gradient_descent) would be a
perfect start to try some functional programming.

The code

    
    import scala.math._
    
    object GradientDecent extends App {
    
      val alpha = 0.1 //size of steps taken in gradient decent
      val samples = List((Vector(0.0, 0.0), 2.0), (Vector(3.0, 1.0), 12.0), (Vector(2.0, 2.0), 18.0))
    
      var tetas = Vector(0.0, 0.0, 0.0)
      for (i 
            teta - (alpha / samples.size) * samples.foldLeft(0.0) {
              case (sum, (x, y)) => decentTerm(sum, 1, x, y, tetas)
            }
          case (teta, i) =>
            teta - (alpha / samples.size) * samples.foldLeft(0.0) {
              case (sum, (x, y)) => decentTerm(sum, x(i - 1), x, y, tetas)
            }
        }
      }
    
      def decentTerm(sum: Double, x_j: Double, x: Vector[Double], y: Double, tetas: Vector[Double]) = {
        sum + x_j * (h(x, tetas) - y)
      }
    
      def h(x: Vector[Double], teta: Vector[Double]): Double = {
        teta(0) + {
          for (i  sum + x)
      }
    
    }


And thats pretty much everything. This is just a first version and I'm sure somebody would find ways
to optimize it. However even this _hacked_ version is very short and handsome :)

**Update
**The code snippet here is a gradient decent for performing [linear regression](http://en.wikipedia.org/wiki/Linear_regression).


#### 
