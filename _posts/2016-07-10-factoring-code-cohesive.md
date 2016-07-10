---
title:  "Factoring Code - Keep it Cohesive"
categories: Design
tags: FactoringYourCode

date: 2016-07-10
---

I intend for this series of posts to be short tidbits leading up to my presentation at the upcoming [Tampa Code Camp][1] called "Factoring Your Code".

Cohesion
----------

Cohesion is the "degree to which the elements of a module belong together".  

At this point, the previous articles start to come together to paint a richer picture.  We're shooting for simplicity and keeping dependencies explicit, which will make cohesion (or lack of it) more obvious.

First an example of low cohesion using a simple hangman game:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace Hangman{
    public static class Program {
        public static void Main(string[] args){
            var random = new Random();

            string[] wordBank = { "Blue", "Black", "Yellow", "Orange", "Green", "Purple" };

            string wordToGuess = wordBank[random.Next(0, wordBank.Length)];
            string wordToGuessUppercase = wordToGuess.ToUpper();

            var displayToPlayer = new StringBuilder(wordToGuess.Length);
            for (int i = 0; i < wordToGuess.Length; i++)
                displayToPlayer.Append('_');

            var correctGuesses = new List<char>();
            var incorrectGuesses = new List<char>();

            int lives = 5;
            bool won = false;
            int lettersRevealed = 0;

            string input;
            char guess;

            while (!won && lives > 0) 
            {
                Console.Write("Guess a letter: ");

                input = Console.ReadLine().ToUpper();
                guess = input[0];

                if (correctGuesses.Contains(guess)) 
                {
                    Console.WriteLine("You've already tried '{0}', and it was correct!", guess);
                    continue;
                }
                else if (incorrectGuesses.Contains(guess)) 
                {
                    Console.WriteLine("You've already tried '{0}', and it was wrong!", guess);
                    continue;
                }

                if (wordToGuessUppercase.Contains(guess)) 
                {
                    correctGuesses.Add(guess);

                    for (int i = 0; i < wordToGuess.Length; i++) 
                    {
                        if (wordToGuessUppercase[i] == guess)
                        {
                            displayToPlayer[i] = wordToGuess[i];
                            lettersRevealed++;
                        }
                    }

                    if (lettersRevealed == wordToGuess.Length)
                    {
                        won = true;
                    }
                }
                else 
                {
                    incorrectGuesses.Add(guess);

                    Console.WriteLine("Nope, there's no '{0}' in it!", guess);
                    lives--;
                }

                Console.WriteLine(displayToPlayer);
            }

            if (won)
                Console.WriteLine("You won!");
            else
                Console.WriteLine("You lost! It was '{0}'", wordToGuess);

            Console.Write("Press ENTER to exit...");
            Console.ReadLine();
        }
    }
}
```

In this simple Hangman game we can see that the code to pick a random word, handle input, evaluate guesses, update the display, and determine if we've won or lost are all mixed together. Untangling these responsibilities will make our code more cohesive by trimming out the pieces that don't belong together. 

We'll tackle the challenge of untangling the game logic from that of picking the word to be guessed and handling input by creating a Game class which tracks the game's state. The class only needs a constructor (taking the word to guess and the number of incorrect guesses allowed) and a MakeGuess method which updates the game's state based on the guess. The resulting code is displayed below:

```csharp
namespace Hangman{
    public class Game {
        private readonly string _wordToGuess;
        private readonly string _wordToGuessUppercase;
        private readonly StringBuilder _wordDisplay;
        private int _lives;
        private int _lettersRevealed;

        private readonly List<char> _correctGuesses = new List<char>();
        private readonly List<char> _incorrectGuesses = new List<char>();

        public bool Finished { get; set; }
        public bool Won { get; set; }

        public Game(string wordToGuess, int lives){
            _wordToGuess = wordToGuess;
            _wordToGuessUppercase = wordToGuess.ToUpper();
            _lives = lives;

            _wordDisplay = new StringBuilder(new String('_', _wordToGuess.Length));
        }

        public string MakeGuess(char guess){
            // Determine if guess was previously guessed, right, or wrong
            // Return a string representing the state of the game.
            // Set Finished and Won based on guess.
        }
    }

    public static class Program {
        public static void Main(string[] args){
            var random = new Random();

            string[] wordBank = { "Blue", "Black", "Yellow", "Orange", "Green", "Purple" };

            string wordToGuess = wordBank[random.Next(0, wordBank.Length)];

            var game = new Game(wordToGuess, 5);

            string input;
            char guess;

            while (!game.Finished) 
            {
                Console.Write("Guess a letter: ");

                input = Console.ReadLine();
                guess = input[0];

                var guessResult = game.MakeGuess(guess);

                Console.WriteLine(guessResult);
            }

            if (game.Won)
                Console.WriteLine("You won!");
            else
                Console.WriteLine("You lost! It was '{0}'", wordToGuess);

            Console.Write("Press ENTER to exit...");
            Console.ReadLine();
        }
    }
}
```

This pares down our main method quite a bit, and both the main method and the Game class have become more streamlined as a result. One thing that still bothers me about this code is that the game logic has to construct the strings used in the display.  We can fix this by refactoring a bit more to remove most of the display logic from the Game class.

We'll add an enum of GuessType which tags the guesses as previously guessed, correct or incorrect. This lets the UI decide how it should display the information.

```csharp
namespace Hangman{
    public class Game {
        private readonly string _wordToGuess;
        private readonly string _wordToGuessUppercase;
        private readonly StringBuilder _wordDisplay;
        private int _lives;
        private int _lettersRevealed;

        private readonly List<char> _correctGuesses = new List<char>();
        private readonly List<char> _incorrectGuesses = new List<char>();

        public bool Finished { get; set; }
        public bool Won { get; set; }

        public Game(string wordToGuess, int lives){
            _wordToGuess = wordToGuess;
            _wordToGuessUppercase = wordToGuess.ToUpper();
            _lives = lives;

            _wordDisplay = new StringBuilder(new String('_', _wordToGuess.Length));
        }

        public GuessResult MakeGuess(char guess){
            // Determine if guess was previously guessed, right, or wrong
            // Return an object representing the state of the game.
            // Set Finished and Won based on guess.
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

    public static class Program {
        public static void Main(string[] args){
            var random = new Random();

            string[] wordBank = { "Blue", "Black", "Yellow", "Orange", "Green", "Purple" };

            string wordToGuess = wordBank[random.Next(0, wordBank.Length)];

            var game = new Game(wordToGuess, 5);

            string input;
            char guess;

            while (!game.Finished) 
            {
                Console.Write("Guess a letter: ");

                input = Console.ReadLine();
                guess = input[0];

                var guessResult = game.MakeGuess(guess);

                switch(guessResult.GuessType){
                    case GuessType.PreviousCorrect:
                        Console.WriteLine("You've already tried '{0}', and it was correct!", guess);
                        break;
                    case GuessType.PreviousIncorrect:
                        Console.WriteLine("You've already tried '{0}', and it was wrong!", guess);
                        break;
                    case GuessType.Correct:
                        // Just display the updated Word
                        break;
                    case GuessType.Incorrect:
                        Console.WriteLine("Nope, there's no '{0}' in it!", guess);
                        break;
                    default:
                        Trace.Fail("Unexpected GuessType returned");
                        break;
                }

                Console.WriteLine(guessResult.WordDisplay);
            }

            if (game.Won)
                Console.WriteLine("You won!");
            else
                Console.WriteLine("You lost! It was '{0}'", wordToGuess);

            Console.Write("Press ENTER to exit...");
            Console.ReadLine();
        }
    }
}
```

At this point, our Game class simply tracks the state of Hangman on a word and evaluates guesses.  We have completely separated the responsibilies of input handling and display resulting in a much more cohesive set of responsibilies.

Hopefully I've demonstrated with this expanded example the thought process and mechanics of refactoring toward cohesiveness. Take the opportunity to practice breaking down your own code into it's cohesive concepts and rebuilding from those pieces.  I think you'll find it's easier than it may appear and you'll be surprised the amount of clarity it can bring. 

(Next time we examine the "Pit of Success"...)

[1]: http://www.tampacodecamp.net