---
title:  "Factoring Code - Keep It Simple"
categories: Design
tags: FactoringYourCode

date: 2016-06-26
---

I intend for this series of posts to be short tidbits leading up to my presentation at the upcoming [Tampa Code Camp][1] called "Factoring Your Code".

KISS, YAGNI
-------------

I bet everyone reading has a few small utility programs that they've written to solve a very specific issue.  Think of how one of these utilities is written.  This utility probably has very few options, possibly none at all.  You hard-coded file names and magic values.  There's no database, no abstractions, no testing, and few classes.  Data flows in, data flows out.  You wouldn't submit this code to an interviewer, or even call it "good", but it does its job and you know exactly how to fix it when it goes wrong.

**This utility is simple.**

Imagine scaling that code style up to a few hundred thousand lines of code and it becomes a nightmare.  Our system has become a mine field where any edit is a potential disaster with unforseen breakages.  This can't be right, we just talked about the virtues of the simple utility... It seems that there's a limit to how far simplicity can take us.  And I've certainly seen those limits bent and broken. 






There's a reason that we all feel the most productive when working on a green-field project, and that reason is simplicity.  At the beginning of a project, we can fit the entire project in our heads, pivot when things are going poorly, and make changes confidently.




[1]: http://www.tampacodecamp.net