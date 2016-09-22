AKA: A C#er's Guide to Structs, Enums, and Null
---------------------------------------------------
My aim in this post is to compare and contrast the some data structures in C# and Rust that a coworker and I were discussing the other day.  I'm not sure I expressed myself clearly, so hopefully this helps.

Wikipedia defines an [Algebraic Data Type][wiki-adt] as: 

> A kind of composite type, i.e., a type formed by combining other types. Two common classes of algebraic types are product types (i.e., tuples and records) and sum types, also called tagged or disjoint unions or variant types.

In C# terms, product types look like classes and sum types look like enums (kind of).  We'll start on common ground.

### Product Types (classes/structs)
The simplest class you could build looks something like this:

```csharp
public class Employee {
    public int ID;
    public string Name;
    public int Age;
}
```

equivalently in Rust:

```rust
pub struct Employee {
    pub id: i32,
    pub name: String,
    pub age: i32,
}
```

These both represent an Employee as the product of 2 ints and a string which we can now refer to by name.  C# `var emp = new Employee{ ID=123, Name="Bob", Age=30 };` or Rust `let emp = Employee{id:123, name:"Bob".to_string(), age:30};`.  This lets us increase our level of abstraction from groups of primitive types to objects representing a thing or concept.  

### Sum Types (enums)



### Null



[wiki-adt]: https://en.wikipedia.org/wiki/Algebraic_data_type

[enum-classes]: https://lostechies.com/jimmybogard/2008/08/12/enumeration-classes/
[type-safe-enum]: http://blog.falafel.com/introducing-type-safe-enum-pattern/
[enum-alternatives]: http://ardalis.com/enum-alternatives-in-c
