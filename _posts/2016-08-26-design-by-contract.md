---
title:  "How I \"Invented\" Design By Contract That One Time..."
categories: Design

date: 2016-08-26
---

How I "Invented" Design By Contract That One Time...
------------------------------------------------------
Or how there's nothing new under the sun.

Story Time
============

So a bit of background about me: I grew up working with computers.  My parents got a used IBM 386 compatible PC from a garage sale for a few bucks when I was 3 years old.  I quickly learned how to boot from the boot floppy, swap out the frogger floppy, and load it up.  I had a small collection of basic games, and I loved that PC.  I'm pretty sure the only reason I got that computer was so I'd stop bugging my mom.  She was an accountant who did the books for a small company and for that reason had a computer to run Lotus123.  I was intrigued.  I was soon showing my mom new things in Lotus and got my first taste of what computers could really do.  Manipulating numbers in a spreadsheet, evaluating formulas, building forms in Visual Basic...  Fast forward many years and experiments later and I had a good understanding of programming, networking, and the other basics.  I ran circles around my classmates in High School programming mostly due to my headstart with computers.  Junior year I enrolled in a Java class at the local college, and took home top scores.  If this was all that programming had to offer, then I imagined I'd soon get bored of it, it felt like a hobby activity or a tool you'd use to accomplish something more than something to study in it's own right.  So, I decided that since programming was just a tool that I'd got a decent handle on, I'd go into Mechanical Engineering instead.  At UF (Go Gators!) I still took as many digital logic, microprocessor, and programming electives as I could, but ended up with a BS in Mech E.  

That's how I found myself at SRS ([Savannah River Site](https://en.wikipedia.org/wiki/Savannah_River_Site)).  This government facility is one where plutonium, uranium, and tritium were produced and refined during the Cold War era.  Due to this, they have a large inventory of nuclear waste that must be handled and tracked carefully.  I found myself in the "Transfer Group" whose job it is to ensure that transfering waste from one large tank to another won't cause a nuclear meltdown.  Part of this job was writing "transfer procedures".

Transfer Procedures
=====================

Transfer Procedures are a long series of steps such as "open valve A-12" and "turn on pump C-16" written so that the operations crew could execute them successfully by reading straight through them.  Humans make mistakes, so there were several safeguards in place to ensure that what was written was what would happen.  Every transfer proceeded in a similar way:

1. Engineer was tasked with writing the procedure (ex. Tank 13 - Tank 48)
2. Engineer would find similar previous procedures (ex. Tank 12 - Tank 48)
3. Engineer would review and make changes
4. Engineering managers and team would review the procedure
5. Transfer happens

I'm actually leaving out many other details of what makes a good procedure, but suffice it to say, it was reviewed from numerous different angles.

Now, if you haven't drawn the parallel yet, we were writing programs that were executed by human beings.  And due to the configuration of piping, valves, and tanks many pieces of the procedures ended up copied and pasted from procedure to procedure sometimes word for word, sometimes slightly modified.  As a developer, I balked at this.  I envisioned a series of subroutines to contain this common logic.  Surely reusing these pieces of one procedure would greatly simplify the task of creating the next procedure.  Additionally, the hours/days of review could be reduced by starting from a known good quantity and improvements to the subroutines would flow to all procedures using them.  Now, there were a few issues with this logic: 

* How do you define a subroutine, what are it's inputs and outputs?
* When do you need to change that subroutine?
* When is it safe to assume it's good?

Contracts
===========

So, thinking through these issues I started to form a few ideas.  

* **Assumptions**
	* All valves closed, all pumps off to start (default state)
	* Single entry and exit point per subroutine (linear flow)
	* Transfers must end with all state reset to initial conditions
* Common global state of valves and pumps
* If state well-known before, then we could know the state after
* As long as there were no piping changes between approval date and execution, the procedure is valid

Going from these assumptions, if we listed the state of the facility prior to execution and after, that could serve as our contract.  This gets us our wonderful world of reuse that we had hoped for in a safe way and saves us many hours reviewing the same thing over and over.  Additionally, if we have two subroutines, as long as their after and before lists match up, they would be compatible with one another.  Later I learned about "Design By Contract"...

Design By Contract
====================

[Design By Contract](https://en.wikipedia.org/wiki/Design_by_contract) defines the notion of a subroutine's contract:

* Acceptable and unacceptable input values or types, and their meanings
* Return values or types, and their meanings
* Error and exception condition values or types that can occur, and their meanings
* Side effects*
* Preconditions*
* Postconditions*
* Invariants*

Now due to the nature of the problem, the first three don't really apply. I'll cover the starred ones:

* **Side Effects**
	* Transfer - Moving waste
	* Subroutine - Some reconfiguration of valves and pumps
* **Preconditions**
	* Transfer - Everything off and closed
	* Subroutine - Specified configuration of valves and pumps
* **Postconditions**
	* Transfer - Everything off and closed
	* Subroutine - Specified configuration of valves and pumps
* **Invariants** - Both require that the facility has not changed since the routine was approved

Now to bring it all back to code in rust syntax...

Each procedure could be defined as `fn Transfer(&mut state: Facility)` with a side-effect of waste movement.  Our subroutines would look something like `fn Subroutine(&mut state: Facility)`.  So a complete procedure would be a series of subroutines:

```rust
const facility_configuration_at_approval = ...;
const default_state = ...;

fn Transfer(&mut state: Facility) {
	// ensure facility has not changed
	assert!(facility_configuration_is_same(facility_configuration_at_approval));
	
	// safety check for all closed/off
	assert!(state_matches(default_state, state));
	
	// ...
	// State changes to match 'A' preconditions
	// ...
	
	SubroutineA(&mut state);
	SubroutineB(&mut state);
	SubroutineC(&mut state);
	
	// ...
	// Cleanup/close valves etc.
	// ...
	
	assert!(state_matches(default_state, state));
}

const a_preconditions = ...;
const a_postconditions = ...;

fn SubroutineA(&mut state: Facility) {
	assert!(state_matches(a_preconditions, state));
	
	// ...
	// Some valves changed here.
	// ...
	
	assert!(state_matches(a_postconditions, state));
}

// ... Code omitted
```

In conclusion, cross functional training and thinking provide great advantages and opportunities to see untapped efficiencies.  Also, although you probably haven't thought up something new, applying old ideas in new areas can be just as good.  Keep your mind open to see the parallels.

PS. Until I get some kind of comments section set up here, let me know what you think on twitter [@ycason](https://twitter.com/ycason).
