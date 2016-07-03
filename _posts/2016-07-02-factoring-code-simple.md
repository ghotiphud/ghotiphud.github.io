---
title:  "Factoring Code - Keep It Simple"
categories: Design
tags: FactoringYourCode

date: 2016-07-02
---

I intend for this series of posts to be short tidbits leading up to my presentation at the upcoming [Tampa Code Camp][1] called "Factoring Your Code".

Contraptions
--------------

Rube Goldberg was an American cartoonist best known for his series of cartoons depicting complicated gadgets that perform simple tasks in indirect, convoluted ways.  His legacy lives on in contraptions like this one:

<iframe width="560" height="315" src="https://www.youtube.com/embed/TUbjzvu1DqE?start=65&end=93" frameborder="0" allowfullscreen></iframe>

Now obviously no one would think that building a large contraption just to erase a whiteboard is a worthwhile task, but that doesn't stop software engineers... Somehow we've perfected the Rube Goldberg machine. We've got factories and strategies and frameworks and abstractions on top of abstractions. 

KISS, YAGNI
-------------

I once worked on a project where the web client built a JSON object with a dozen or so properties, send that to the server where it was deserialized, passed through 10 more layers, each one adding more properties (including at one point an entire copy of the original), and building ever larger objects.  Once it reached the bottom of the stack (if it made it there before multiple errors were thrown), it proceeded to set a key-value pair... After weeks of deciphering what was actually happening, I deleted over 100,000 lines of code and replaced the maze of function calls with "Get Value" and "Set Value". 

This clarity into what was actually happening had cascading effects on the surrounding code, and we ended up with a much cleaner product. 

Often times the best solution will be the simplest one, but many times that solution is extremely difficult to see "the forest for the trees".  Each layer of this stack made **some** sense, the abstractions were poor, but at the time they were built, the project was in the "too little information" stage. So take heed and revisit the assumptions you made early in a project.

Whence Complexity Comes
-------------------------

Not all complexity is avoidable, and I think that's apparent because if it were, all of our requirement documents would just say "make the computer do the thing".  I mean, sometimes we even get something other than "as a user, I would like to log in".  

Now, unless you're just banging out CRUD apps all day long that collect data that isn't used elsewhere, you're going to have to put some thought into the question "is this complexity due to the problem or due to the implementation?". The difference here is "inherent complexity" (essential) vs "incidental complexity" (oops). There are no easy answers here, but the process can be extremely rewarding (deleting code is the best!). 

So to top things off, I'll leave you with an example of mechanized whiteboarding with inherent complexity: 

<iframe width="560" height="315" src="https://www.youtube.com/embed/4QgeQAiSmM8" frameborder="0" allowfullscreen></iframe>

(Please ignore the fact that a clock would be simpler...)

[1]: http://www.tampacodecamp.net