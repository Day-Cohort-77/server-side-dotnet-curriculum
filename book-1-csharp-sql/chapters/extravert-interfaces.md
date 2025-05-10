# Interfaces and Abstraction

In this chapter, we'll explore interfaces in C#, which are a powerful tool for achieving abstraction and creating flexible, maintainable code. Interfaces define a contract that classes can implement, allowing for polymorphism and loose coupling.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the concept of interfaces in C#
- Define and implement interfaces
- Use interfaces to achieve polymorphism
- Implement multiple interfaces in a single class
- Understand default interface methods

## Understanding Interfaces

An interface is a contract that defines a set of methods, properties, events, or indexers that a class must implement. Interfaces allow you to define what a class can do without specifying how it does it.

> 💡 Interfaces are different from inheriting from a parent class in that a parent class has default implementations that the child automatically gets. An interface simply requires that a class **must** implement it.

Key characteristics of interfaces:
- They can contain method signatures, properties, events, and indexers, but not fields
- They cannot contain implementation code
- A class can implement multiple interfaces
- Interfaces can inherit from other interfaces

## Creating an Interface

Let's create a simple interface for animals that can make sounds:

```csharp
// Models/ISoundMaker.cs
namespace ExtraVert.Models
{
    public interface ISoundMaker
    {
        // Method signature
        void MakeSound();

        // Property signature
        string SoundDescription { get; }
    }
}
```

This interface defines a contract that any class implementing `ISoundMaker` must provide:
1. A `MakeSound()` method that performs the action of making a sound
2. A `SoundDescription` property that describes the sound

## Implementing an Interface

Now, let's create a simple `Dog` class that implements the `ISoundMaker` interface:

```csharp
// Models/Dog.cs
using System;

namespace ExtraVert.Models
{
    public class Dog : ISoundMaker
    {
        public string Name { get; set; }
        public string Breed { get; set; }

        // Property from ISoundMaker interface
        public string SoundDescription { get; private set; }

        // Constructor
        public Dog(string name, string breed)
        {
            Name = name;
            Breed = breed;
            SoundDescription = "Woof! Woof!";
        }

        // Implement ISoundMaker.MakeSound method
        public void MakeSound()
        {
            Console.WriteLine($"{Name} says: {SoundDescription}");
        }

        // Dog-specific method
        public void Fetch()
        {
            Console.WriteLine($"{Name} is fetching the ball!");
        }
    }
}
```

Let's also create a `Cat` class that implements the same interface:

```csharp
// Models/Cat.cs
using System;

namespace ExtraVert.Models
{
    public class Cat : ISoundMaker
    {
        public string Name { get; set; }
        public string Color { get; set; }

        // Property from ISoundMaker interface
        public string SoundDescription { get; private set; }

        // Constructor
        public Cat(string name, string color)
        {
            Name = name;
            Color = color;
            SoundDescription = "Meow! Meow!";
        }

        // Implement ISoundMaker.MakeSound method
        public void MakeSound()
        {
            Console.WriteLine($"{Name} says: {SoundDescription}");
        }

        // Cat-specific method
        public void Climb()
        {
            Console.WriteLine($"{Name} is climbing the tree!");
        }
    }
}
```

Lastly, also create a `Snake` class that **doesn't** implement the interface.

```csharp
// Models/Snake.cs
using System;

namespace ExtraVert.Models
{
    public class Snake
    {
        public string Name { get; set; }
        public string Color { get; set; }

        // Constructor
        public Snake(string name, string color)
        {
            Name = name;
            Color = color;
        }

        // Snake-specific method
        public void Slither()
        {
            Console.WriteLine($"{Name} is slithering through the brush!");
        }
    }
}
```

## Using Interfaces for Polymorphism

One of the main benefits of interfaces is that they enable polymorphism - the ability to group objects that share the same interface into collections in a sensible way. Let's create a simple service that works with any `ISoundMaker`:

```csharp
// Services/SoundService.cs
using System;
using System.Collections.Generic;
using ExtraVert.Models;

namespace ExtraVert.Services
{
    public class SoundService
    {
        // This method works with any object that implements ISoundMaker
        public void MakeSoundTwice(ISoundMaker soundMaker)
        {
            Console.WriteLine("Making sound twice:");
            soundMaker.MakeSound();
            soundMaker.MakeSound();
        }

        // This method works with a list of any objects that implement ISoundMaker
        public void MakeAllSounds(List<ISoundMaker> soundMakers)
        {
            Console.WriteLine("All sounds:");

            foreach (var soundMaker in soundMakers)
            {
                soundMaker.MakeSound();
            }
        }
    }
}
```

## Using the Interface in a Program

Here's how we might use these classes and interfaces in a program:

```csharp
// Program.cs
using System;
using System.Collections.Generic;
using ExtraVert.Models;
using ExtraVert.Services;

namespace ExtraVert
{
    class Program
    {
        static void Main(string[] args)
        {
            // Create some animals
            Dog fido = new Dog("Fido", "Golden Retriever");
            Cat whiskers = new Cat("Whiskers", "Tabby");
            Snake skinny = new Snake("Skinny", "Copperhead");

            // Use dog-specific method
            fido.Fetch();

            // Use cat-specific method
            whiskers.Climb();

            // Use snake-specific method
            skinny.Slither();

            // Create a list of only animals that make sounds
            List<ISoundMaker> soundMakers = new List<ISoundMaker>
            {
                fido,    // Dog implements ISoundMaker
                whiskers // Cat implements ISoundMaker
            };

            // Create the sound service
            SoundService soundService = new SoundService();

            // Make all sounds
            soundService.MakeAllSounds(soundMakers);

            // Make Fido's sound twice
            soundService.MakeSoundTwice(fido);

            Console.ReadLine();
        }
    }
}
```

The output would be:
```
Fido is fetching the ball!
Whiskers is climbing the tree!
All sounds:
Fido says: Woof! Woof!
Whiskers says: Meow! Meow!
Making sound twice:
Fido says: Woof! Woof!
Fido says: Woof! Woof!
```

## Multiple Interfaces

A class can implement multiple interfaces. Let's create another interface for animals that can move:

```csharp
// Models/IMovable.cs
namespace ExtraVert.Models
{
    public interface IMovable
    {
        void Move();
        int Speed { get; }
    }
}
```

Now, let's update the `Dog` class to implement both interfaces:

```csharp
// Models/Dog.cs
using System;

namespace ExtraVert.Models
{
    public class Dog : ISoundMaker, IMovable
    {
        public string Name { get; set; }
        public string Breed { get; set; }

        // Property from ISoundMaker interface
        public string SoundDescription { get; private set; }

        // Property from IMovable interface
        public int Speed { get; private set; }

        // Constructor
        public Dog(string name, string breed)
        {
            Name = name;
            Breed = breed;
            SoundDescription = "Woof! Woof!";
            Speed = 15; // mph
        }

        // Implement ISoundMaker.MakeSound method
        public void MakeSound()
        {
            Console.WriteLine($"{Name} says: {SoundDescription}");
        }

        // Implement IMovable.Move method
        public void Move()
        {
            Console.WriteLine($"{Name} is running at {Speed} mph!");
        }

        // Dog-specific method
        public void Fetch()
        {
            Console.WriteLine($"{Name} is fetching the ball!");
        }
    }
}
```

🧨 Your job is to have all 3 animal class implement the `IMovable` interface since cats and snakes can also move.

## Interface Segregation Principle

The Interface Segregation Principle (ISP) is a fundamental principle of object-oriented design. It states that _"Clients should not be forced to depend upon interfaces that they do not use."_

In our example, we've applied the ISP by creating separate interfaces for different behaviors:
- `ISoundMaker` for animals that can make sounds
- `IMovable` for animals that can move

This allows classes to implement only the interfaces that are relevant to them. For example, a `Plant` class might not implement either interface, while a `Dog` class might implement both.

## Next Steps

In this chapter, we've explored interfaces and how they can be used to create more flexible, maintainable code. We've implemented interfaces for our animal classes and created a service class that works with interfaces.

In the next chapters, we'll explore SQL and database connectivity, which will allow us to persist our data beyond the lifetime of our application.

Before moving on, make sure you're comfortable with:
- Defining and implementing interfaces
- Using interfaces to achieve polymorphism
- Implementing multiple interfaces in a single class
- Understanding default interface methods

## Practice Exercise

Enhance the ExtraVert application by:
1. Creating a new interface called `ITrainable` with methods like `Train()` and properties like `TrainingLevel`
2. Implementing the `ITrainable` interface in the only in the `Dog` class
3. Creating a `TrainingService` class that works with `ITrainable` objects
4. Updating the program to use the new interface and service
5. Running the application and verifying that the training functionality works correctly