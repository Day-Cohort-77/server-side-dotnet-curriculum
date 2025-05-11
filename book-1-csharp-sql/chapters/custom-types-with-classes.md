# Custom Types with Classes

In this chapter, we'll explore how to create custom types in C# using the `class` keyword. Classes are the foundation of object-oriented programming and allow you to define your own data types with properties and behaviors.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand what classes are and why they're useful
- Create custom types using the `class` keyword
- Define properties to store data in your classes
- Implement methods to add behavior to your classes
- Create instances of your custom types
- Store and work with collections of custom types

## Setting Up

Create a new console application to work with:

```bash
dotnet new console -n ClassExplorer
cd ClassExplorer
code .
```

## Introduction to Classes

A class is a blueprint for creating objects. It defines the data (properties) and behavior (methods) that objects of that type will have. Classes allow you to model real-world entities in your code, making your programs more organized and easier to understand.

### Why Use Classes?

Classes provide several benefits:

1. **Organization**: Group related data and functionality together
2. **Encapsulation**: Hide implementation details and expose only what's necessary
3. **Reusability**: Create multiple instances of the same type
4. **Abstraction**: Model complex systems at a higher level

## Creating a Simple Class

Let's create a simple `Person` class to represent a person with a name, age, and occupation:

```csharp
// Define the Person class
public class Person
{
    // Properties to store data
    public string Name { get; set; }
    public int Age { get; set; }
    public string Occupation { get; set; }

    // Method to display information about the person
    public void Introduce()
    {
        Console.WriteLine($"Hello, my name is {Name}. I am {Age} years old and work as a {Occupation}.");
    }
}
```

In this example:
- `public class Person { ... }` defines a new class called `Person`
- `public string Name { get; set; }` defines a property to store the person's name
- `public int Age { get; set; }` defines a property to store the person's age
- `public string Occupation { get; set; }` defines a property to store the person's occupation
- `public void Introduce() { ... }` defines a method that displays information about the person

## Creating Instances of a Class

Once you've defined a class, you can create instances (objects) of that class:

```csharp
// In your Program.cs Main method
Person person1 = new Person();
person1.Name = "Alice";
person1.Age = 30;
person1.Occupation = "Software Developer";

Person person2 = new Person();
person2.Name = "Bob";
person2.Age = 25;
person2.Occupation = "Data Analyst";

// Call the Introduce method on each person
person1.Introduce(); // Output: Hello, my name is Alice. I am 30 years old and work as a Software Developer.
person2.Introduce(); // Output: Hello, my name is Bob. I am 25 years old and work as a Data Analyst.
```

## Creating a Second Class

Let's create another class to represent a `Book`:

```csharp
public class Book
{
    // Properties
    public string Title { get; set; }
    public string Author { get; set; }
    public int PageCount { get; set; }

    // Method to display book information
    public void DisplayInfo()
    {
        Console.WriteLine($"'{Title}' by {Author} ({PageCount} pages)");
    }
}
```

Now we can create instances of the `Book` class:

```csharp
Book book1 = new Book();
book1.Title = "The Great Gatsby";
book1.Author = "F. Scott Fitzgerald";
book1.PageCount = 180;

Book book2 = new Book();
book2.Title = "To Kill a Mockingbird";
book2.Author = "Harper Lee";
book2.PageCount = 281;

book1.DisplayInfo(); // Output: 'The Great Gatsby' by F. Scott Fitzgerald (180 pages)
book2.DisplayInfo(); // Output: 'To Kill a Mockingbird' by Harper Lee (281 pages)
```

## Working with Collections of Custom Types

One of the powerful aspects of custom types is that you can create collections of them, just like you would with built-in types:

```csharp
// Create a list of Person objects
List<Person> people = new List<Person>();

// Add the people we created earlier
people.Add(person1);
people.Add(person2);

// Create and add a new person directly
people.Add(new Person { Name = "Charlie", Age = 35, Occupation = "Teacher" });

// Create a list of Book objects
List<Book> books = new List<Book>();

// Add the books we created earlier
books.Add(book1);
books.Add(book2);

// Create and add a new book directly
books.Add(new Book { Title = "1984", Author = "George Orwell", PageCount = 328 });
```

Now we can iterate through these collections:

```csharp
Console.WriteLine("People:");
foreach (Person person in people)
{
    person.Introduce();
}

Console.WriteLine("\nBooks:");
foreach (Book book in books)
{
    book.DisplayInfo();
}
```

Output:
```
People:
Hello, my name is Alice. I am 30 years old and work as a Software Developer.
Hello, my name is Bob. I am 25 years old and work as a Data Analyst.
Hello, my name is Charlie. I am 35 years old and work as a Teacher.

Books:
'The Great Gatsby' by F. Scott Fitzgerald (180 pages)
'To Kill a Mockingbird' by Harper Lee (281 pages)
'1984' by George Orwell (328 pages)
```

## Putting It All Together

Let's create a complete program that demonstrates creating and using custom types:

```csharp
using System;
using System.Collections.Generic;

namespace ClassExplorer
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Custom Types Explorer");
            Console.WriteLine("--------------------");

            // Create some Person instances
            Person person1 = new Person
            {
                Name = "Alice",
                Age = 30,
                Occupation = "Software Developer"
            };

            Person person2 = new Person
            {
                Name = "Bob",
                Age = 25,
                Occupation = "Data Analyst"
            };

            // Create some Book instances
            Book book1 = new Book
            {
                Title = "The Great Gatsby",
                Author = "F. Scott Fitzgerald",
                PageCount = 180
            };

            Book book2 = new Book
            {
                Title = "To Kill a Mockingbird",
                Author = "Harper Lee",
                PageCount = 281
            };

            // Create collections
            List<Person> people = new List<Person> { person1, person2 };
            List<Book> books = new List<Book> { book1, book2 };

            // Display all people
            Console.WriteLine("\nPeople:");
            foreach (Person person in people)
            {
                person.Introduce();
            }

            // Display all books
            Console.WriteLine("\nBooks:");
            foreach (Book book in books)
            {
                book.DisplayInfo();
            }

            // Find a specific person
            Person foundPerson = people.Find(p => p.Name == "Alice");
            if (foundPerson != null)
            {
                Console.WriteLine("\nFound person:");
                foundPerson.Introduce();
            }

            // Find books with more than 200 pages
            Console.WriteLine("\nBooks with more than 200 pages:");
            List<Book> longBooks = books.FindAll(b => b.PageCount > 200);
            foreach (Book book in longBooks)
            {
                book.DisplayInfo();
            }
        }
    }

    // Define the Person class
    public class Person
    {
        public string Name { get; set; }
        public int Age { get; set; }
        public string Occupation { get; set; }

        public void Introduce()
        {
            Console.WriteLine($"Hello, my name is {Name}. I am {Age} years old and work as a {Occupation}.");
        }
    }

    // Define the Book class
    public class Book
    {
        public string Title { get; set; }
        public string Author { get; set; }
        public int PageCount { get; set; }

        public void DisplayInfo()
        {
            Console.WriteLine($"'{Title}' by {Author} ({PageCount} pages)");
        }
    }
}
```

## Practice Exercise

Create a console application that:

1. Defines a `Product` class with properties for `Name`, `Price`, and `Category`
2. Defines a `Customer` class with properties for `Name`, `Email`, and a method to `DisplayInfo()`
3. Creates several instances of each class
4. Stores the instances in appropriate lists
5. Implements functionality to:
   - Display all products
   - Display all customers
   - Find products by category
   - Find the most expensive product

## Next Steps

In the next chapter, we'll explore exception handling in C#. You'll learn how to catch and handle errors gracefully, ensuring your applications can recover from unexpected situations.

Before moving on, make sure you're comfortable with:
- Creating custom types using the `class` keyword
- Defining properties and methods in your classes
- Creating instances of your custom types
- Working with collections of custom types

[Next Chapter: Exception Handling](./handling-exceptions.md)