---
title:  "Factoring Code - What and When"
categories: Design
tags: FactoringYourCode

date: 2016-06-26
---

I intend for this series of posts to be short tidbits leading up to my presentation at the upcoming [Tampa Code Camp][1] called "Factoring Your Code".

Factoring
---------------

First a definition (made up as I type right now, seems legit):

`Representing something as a composite of it's parts.`

If we all think back to our days in school, there's a good chance that you've covered this topic in algebra. Most likely as a way to find the roots of an equation.  For example:

```
f(x) = x^2 + 2x - 3

f(x) = (x + 3) * (x - 1)

so for f(x) = 0, 
x = -3 or 1
```

Applying this notion is very powerful and a bit abstract.  We are saying that those two equations are equivalent.  How do we know?  They produce the same answers.  So, the equation or problem is unchanging, but how we chose to represent it is up to us.  Well, if it's up to me, I'm going to choose whichever representation makes solving my problem easiest.  Just as our equation became easier to understand, and easier to work with, so will our code when properly factored.

When to Factor
----------------------

There have been many attempts to define rules of thumb for "good code", and I'm sure you're familiar with at least a few.

* SOLID
* GRASP
* Design Patterns
* DRY
* Etc.

All of these are helpful, somewhat rigorous, and I recommend that you read up on them as I won't have time to go in depth here.  Learning these things will help to hone your senses to detect "code smells" and give you a common vocabulary to communicate your ideas.

But when we get down to it, there are a handful of principles that are easy to follow, and seem to do a decent job.

* Keep things simple
* Keep dependencies explicit
* Keep things cohesive
* Design for the "pit of success"
* Experiment! (Don't be afraid to fail)

(Next up, I expound on these further...)

[1]: http://www.tampacodecamp.net