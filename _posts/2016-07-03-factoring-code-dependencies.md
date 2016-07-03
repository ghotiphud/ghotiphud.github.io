---
title:  "Factoring Code - Dependencies"
categories: Design
tags: FactoringYourCode

date: 2016-07-03
---

I intend for this series of posts to be short tidbits leading up to my presentation at the upcoming [Tampa Code Camp][1] called "Factoring Your Code".

Dependencies
--------------

Just to make sure we're all on the same page, when I talk about dependencies I mean things like:

* Databases
* Remote Services
* Internal Services
* Files
* Data

Basically, any call in your application that deals with a resource that isn't under your control or is modified in many places.

Explicit
----------

> “Programs must be written for people to read, and only incidentally for machines to execute.”  
> - Harold Abelson, Structure and Interpretation of Computer Programs

Implicit code feels like "magic" but confuses those unfamiliar with the trick.  It often allows you to accomplish things more quickly by reaching across layers and grabbing what you need, but anyone who's run into the limit of the magic will know that pain lies beneath it.

The hard part about dependencies is that they sneak up on you. If you're not vigilant you can end up with code like this:

```csharp
// Somewhere deep in your system
public string GetCurrentUserName(){
    return HttpContext.Current.User.Identity.Name;
}

// Usage
public void TheFunc(){
    var username = GetCurrentUserName();
                // ^^^^ UserName comes from the ether.  Magic!
}
```

Haha! If you're not familiar with C# and ASP.Net, you now are tied to a large framework with many intricacies of its own. Oh, and every other class that calls this method is also entangled! Want to test those classes/methods? It's possible... (google "Mock HttpContext") But there has to be a better way...

```csharp
public string GetUserName(IIdentity user){
    return user.Name;
}

// Usage
public void TheFunc(IIdentity user){
    var username = GetUserName(user);
                            // ^^^^ Dependencies revealed
}
```

Simply passing the dependency into the method has clarified what's happening here.  

Dependency Pain
-----------------

You're probably thinking to yourself, if I'm passing all dependencies into methods, won't I end up with bloated parameter lists?  Yes, but people respond to pain, and this pain will push you toward fewer dependencies and better designs.  

Now that we've revealed the pain, how should we go about easing it? 

* What if we pass more basic data?

```csharp
public void TheFunc(string username){
    // No need to call GetUserName anymore.
}
```

* What if we gather up a few of those functions with common dependencies and wrap them up nicely together?

```csharp
public class Foo {
    private IIdentity _user;

    public Foo(IIdentity user){
        _user = user;
    }

    public void TheFunc() { /* uses IIdentity */}
    public int Function2() { /* uses IIdentity */}
    public double Function3() { /* uses IIdentity */}
}
```

There are probably 1000 other ways to factor the code for even an example as simple as this, but how do we choose a good factoring?

(Next we tackle cohesiveness...)

[1]: http://www.tampacodecamp.net