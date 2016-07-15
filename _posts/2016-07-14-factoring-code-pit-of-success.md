---
title:  "Factoring Code - Design for the 'Pit of Success'"
categories: Design
tags: FactoringYourCode

date: 2016-07-14
---

I intend for this series of posts to be short tidbits leading up to my presentation at the upcoming [Tampa Code Camp][1] called "Factoring Your Code".

Pit Of Success
----------------

At this point I'm not 100% certain where I first heard the term "Pit of Success", but it has stuck with me. Essentially it refers to designing APIs, languages, or programs in a way that leads the developer down the right path when they're using them. In short, **Design for your user** (which may be you). 

Code that guides us down the correct path follows a couple key principals. First, it codifies the assumptions which were made in its writing and keeps us from violating them. Second, it works to make the less safe or less performant path the harder one to follow. 

### Assumptions

I've come to believe that the assumptions we make are key to the "pit of success". If we can codify our assumptions, then our code is less surprising and more likely to lead to success. 

A few tips to get you started:

1. Avoid invalid objects.
2. Assert yourself.
3. Use types to catch bugs.

#### Avoid Invalid Objects

How many times have you been working on a project and seen something like this?

```csharp
var employee = new Employee();
// at this point Employee fields are mostly null, false, and 0.
employee.Name = "Fred";
employee.Email = "Fred@example.com";

// Checking if employee is a valid object.
if(employee2.Name != null && employee2.Email != null) {}
```

But you think to yourself, does it even make sense to have an employee without a name? What even is a nameless employee? Does this guy have a cubicle down the hall from you, but just never interacts with anyone else?  Is he a forgotten remnant that finance forgot to stop sending paychecks to?

These pieces of information about your object are called _invariants_ and they are part of what make the employee an employee in your system.  So, **enforce them!** Don't allow me to create invalid objects, because I don't know what's expected.

```csharp
var employee = new Employee("Fred", "Fred@example.com");

// No longer a need for consistency checks, employees are valid objects.
```

This is much better, as you've _codified_ the correct initialization of your object. Even better than this would be to run validations in your constructor that verify your object is correct. One excellent example of this that I'll steal from the programming language Rust is their String type which verifies the string is valid UTF-8 upon construction. This is great because anywhere else you use a string, you can just assume correctness. 

Further learning - [Crafting Wicked Domain Models][2] by Jimmy Bogard.

#### Assert Yourself

Looking at the example above you might think, what should I do if someone constructs an Employee with a _null_ name or email?  Well, this case is something called a _precondition_.  We've already said that _invariants_ should not be violated, and _null_ is a definite violation.  I say, slap that person on the hand, give them a piece of your mind, and blow up in their face. Not literally of course, but the code should run no further. This input has no logical next step, and all our other code is relying on the _invariants_ that we promised to them. 

```csharp
public class Employee {
    public string Name;
    public string Email;

    public Employee(string name, string email){
        Trace.Assert(name != null); // Boom!
        // OR if you have something against explosions...
        if(name == null) throw new ArgumentException("name");
        // but prefer to fail fast
    }
}
```

Now we can continue on our merry way, safe in the knowledge that _invariants_ were upheld.

**NOTE:** If you want to take these ideas to their logical conclusion in C# check out [Code Contracts][4] which checks these preconditions at compile time.  Or, look into languages like Rust which build some of this into the language itself.

#### Use Types To Catch Bugs

One of the most potent weapons we have as programmers to keep ourselves from making mistakes is typing. Typing as in types rather than the action of banging on a keyboard. I'm not going to debate the merits of static vs dynamic, weak vs strong, but frequently in my experience 100s of possible bugs and many more lines of test scripts can be replaced by a good compiler.  

Compare the below code samples using an enum rather than a string.

```csharp
class Foo{
    // Valid Types: Bar, Baz
    public string FooType { get; set; }
}

// Using Foo
var foo_instance = new Foo { FooType: "Bar" };

if(foo_instance.FooType == "bar"){
    // Misspelling, subtle bug that may not be caught for a while.
}
```

```csharp
enum FooType{
    Bar,
    Baz,
}

class Foo{
    public FooType FooType { get; set; }
}

var foo_instance = new Foo { FooType: FooType.Bar };

if(foo_instance == FooType.bar){
    // Misspelling of enum, this code won't compile.
}
```

As you can see, we've nearly eliminated the opportunity for a subtle bug to sneak in.

For a more complex example, we'll look at our hangman game from the previous post. 

```csharp
public class Game {
    private readonly string _wordToGuess;
    private readonly string _wordToGuessUppercase;
    private readonly StringBuilder _wordDisplay;
    private int _lives;
    private int _lettersRevealed;

    private readonly List<char> _correctGuesses = new List<char>();
    private readonly List<char> _incorrectGuesses = new List<char>();

    public bool Finished { get { return Won || _lives == 0; } }
    public bool Won { get { return _lettersRevealed == _wordToGuess.Length; } }

    public Game(string wordToGuess, int lives){
        // Constructor
    }

    public GuessResult MakeGuess(char guess){
        guess = Char.ToUpper(guess);

        // Not new-ing this struct to allow the 
        // compiler to catch unset properties.
        // If GuessType were not set in one of the 
        // branches it would not compile
        GuessResult result;

        if (_correctGuesses.Contains(guess))
        {
            result.GuessType = GuessType.PreviousCorrect;
        }
        else if (_incorrectGuesses.Contains(guess))
        {
            result.GuessType = GuessType.PreviousIncorrect;
        }
        else if (_wordToGuessUppercase.Contains(guess))
        {
            result.GuessType = GuessType.Correct;
            _correctGuesses.Add(guess);

            RevealLetter(guess);
        }
        else 
        {
            result.GuessType = GuessType.Incorrect;
            _incorrectGuesses.Add(guess);
            _lives--;
        }

        result.WordDisplay = _wordDisplay.ToString();
        return result;
    }

    private void RevealLetter(char letter){
        for (int i = 0; i < _wordToGuess.Length; i++) 
        {
            if (_wordToGuessUppercase[i] == letter)
            {
                _wordDisplay[i] = _wordToGuess[i];
                _lettersRevealed++;
            }
        }
    }
}

public enum GuessType{
    None,
    PreviousCorrect,
    PreviousIncorrect,
    Correct,
    Incorrect,
}

public struct GuessResult{
    public string WordDisplay { get; set; }
    public GuessType GuessType { get; set; }
}
```

If you were reading very closely you may have noticed that we never new-ed up our GuessResult in the MakeGuess method, and this was intentional. We are relying on the fact that structs are not allowed to be used or returned prior to initalizing all of their properties. This means that if we were to forget to set the GuessType in one of the branches of the method it would not compile! We've just made the compiler our correctness checker. 

Obviously we're just getting started, there are so many other ways to make your APIs easier to use, and harder to use incorrectly. I plan to come back around with more of these tips, maybe a semi-frequent post subject?

(Running short on time pre-CodeCamp so, this will be the last in this series...)

[1]: http://www.tampacodecamp.net
[2]: https://vimeo.com/43598193
[3]: https://spin.atomicobject.com/2014/12/09/typed-language-tdd-part1/
[4]: https://msdn.microsoft.com/en-us/library/dd264808(v=vs.110).aspx