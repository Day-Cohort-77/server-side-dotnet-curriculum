# Collections: Lists

In this chapter, we'll explore one of the most commonly used collection types in C#: the List. Lists allow you to store multiple items of the same type in a single collection, and they provide many useful methods for working with that data.

## Learning Objectives

By the end of this chapter, you should be able to:
- Create and initialize Lists
- Add, remove, and modify elements in a List
- Search and sort Lists
- Iterate through Lists using loops
- Understand the difference between Lists and Arrays
- Use common List methods and properties

## Setting Up

Create a new console application to work with:

```bash
dotnet new console -n ListExplorer
cd ListExplorer
code .
```

## Introduction to Lists

A List is a collection of items that can grow or shrink dynamically. Unlike arrays, which have a fixed size, Lists can be resized as needed. Lists are defined in the `System.Collections.Generic` namespace.

### Creating Lists

To use Lists, you need to include the appropriate namespace:

```csharp
using System.Collections.Generic;
```

Here are different ways to create and initialize Lists:

```csharp
// Create an empty List of strings
List<string> names = new List<string>();

// Create a List with initial capacity
List<int> numbers = new List<int>(10);

// Create and initialize a List in one line
List<string> fruits = new List<string> { "Apple", "Banana", "Cherry" };
```

The angle brackets `<>` specify the type of elements the List will contain. This is called a generic type parameter.

## Adding Elements to a List

You can add elements to a List using the `Add` method:

```csharp
List<string> names = new List<string>();
names.Add("Alice");
names.Add("Bob");
names.Add("Charlie");

Console.WriteLine($"Count: {names.Count}"); // Output: Count: 3
```

You can also add multiple elements at once using the `AddRange` method:

```csharp
List<string> moreNames = new List<string> { "David", "Eve" };
names.AddRange(moreNames);

Console.WriteLine($"Count after AddRange: {names.Count}"); // Output: Count after AddRange: 5
```

## Accessing Elements in a List

You can access elements in a List using an index, just like with arrays:

```csharp
List<string> fruits = new List<string> { "Apple", "Banana", "Cherry" };

Console.WriteLine(fruits[0]); // Output: Apple
Console.WriteLine(fruits[1]); // Output: Banana
Console.WriteLine(fruits[2]); // Output: Cherry
```

Remember that indexes are zero-based, meaning the first element is at index 0.

## Modifying Elements in a List

You can modify elements by assigning a new value to a specific index:

```csharp
List<string> fruits = new List<string> { "Apple", "Banana", "Cherry" };
fruits[1] = "Blueberry";

Console.WriteLine(fruits[1]); // Output: Blueberry
```

## Removing Elements from a List

There are several ways to remove elements from a List:

```csharp
List<string> fruits = new List<string> { "Apple", "Banana", "Cherry", "Date", "Elderberry" };

// Remove a specific element
fruits.Remove("Banana");
Console.WriteLine(string.Join(", ", fruits)); // Output: Apple, Cherry, Date, Elderberry

// Remove element at a specific index
fruits.RemoveAt(2); // Removes "Date"
Console.WriteLine(string.Join(", ", fruits)); // Output: Apple, Cherry, Elderberry

// Remove a range of elements
fruits.RemoveRange(0, 2); // Removes 2 elements starting at index 0
Console.WriteLine(string.Join(", ", fruits)); // Output: Elderberry

// Clear all elements
fruits.Clear();
Console.WriteLine($"Count after Clear: {fruits.Count}"); // Output: Count after Clear: 0
```

## Checking if an Element Exists

You can check if a List contains a specific element using the `Contains` method:

```csharp
List<string> fruits = new List<string> { "Apple", "Banana", "Cherry" };

bool hasBanana = fruits.Contains("Banana");
Console.WriteLine($"Contains Banana: {hasBanana}"); // Output: Contains Banana: True

bool hasOrange = fruits.Contains("Orange");
Console.WriteLine($"Contains Orange: {hasOrange}"); // Output: Contains Orange: False
```

## Finding Elements in a List

You can find the index of an element using the `IndexOf` method:

```csharp
List<string> fruits = new List<string> { "Apple", "Banana", "Cherry", "Banana" };

int firstBananaIndex = fruits.IndexOf("Banana");
Console.WriteLine($"First Banana index: {firstBananaIndex}"); // Output: First Banana index: 1

int lastBananaIndex = fruits.LastIndexOf("Banana");
Console.WriteLine($"Last Banana index: {lastBananaIndex}"); // Output: Last Banana index: 3

int orangeIndex = fruits.IndexOf("Orange");
Console.WriteLine($"Orange index: {orangeIndex}"); // Output: Orange index: -1 (not found)
```

## Iterating Through a List

You can iterate through a List using various loop constructs:

### Using a foreach Loop

```csharp
List<string> fruits = new List<string> { "Apple", "Banana", "Cherry" };

foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);
}
```

### Using a for Loop

```csharp
List<string> fruits = new List<string> { "Apple", "Banana", "Cherry" };

for (int i = 0; i < fruits.Count; i++)
{
    Console.WriteLine($"Index {i}: {fruits[i]}");
}
```

## Sorting a List

You can sort a List using the `Sort` method:

```csharp
List<string> fruits = new List<string> { "Cherry", "Apple", "Banana" };

fruits.Sort();
Console.WriteLine(string.Join(", ", fruits)); // Output: Apple, Banana, Cherry

// Sort in reverse order
fruits.Sort();
fruits.Reverse();
Console.WriteLine(string.Join(", ", fruits)); // Output: Cherry, Banana, Apple
```

## Converting Between Lists and Arrays

You can convert between Lists and arrays:

```csharp
// Convert array to List
string[] fruitArray = { "Apple", "Banana", "Cherry" };
List<string> fruitList = new List<string>(fruitArray);

// Convert List to array
List<string> colorList = new List<string> { "Red", "Green", "Blue" };
string[] colorArray = colorList.ToArray();
```

## Lists of Custom Types

You can create Lists of any type, including your own custom classes:

```csharp
// Create and use a List of Person objects
List<Person> people = new List<Person>
{
    new Person { Name = "Alice", Age = 30 },
    new Person { Name = "Bob", Age = 25 },
    new Person { Name = "Charlie", Age = 35 }
};

foreach (Person person in people)
{
    Console.WriteLine(person);
}

// Define a simple Person class
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public override string ToString()
    {
        return $"{Name} ({Age})";
    }
}

```

## Putting It All Together

Let's create a simple task list application that demonstrates working with Lists:

```csharp
List<string> tasks = new List<string>();
bool running = true;

Console.WriteLine("Task List Application");

while (running)
{
    Console.WriteLine("\nOptions:");
    Console.WriteLine("1. Add a task");
    Console.WriteLine("2. View all tasks");
    Console.WriteLine("3. Remove a task");
    Console.WriteLine("4. Clear all tasks");
    Console.WriteLine("5. Exit");
    Console.Write("Enter your choice (1-5): ");

    string choice = Console.ReadLine();

    switch (choice)
    {
        case "1":
            Console.Write("Enter task description: ");
            string task = Console.ReadLine();
            tasks.Add(task);
            Console.WriteLine("Task added successfully!");
            break;

        case "2":
            if (tasks.Count == 0)
            {
                Console.WriteLine("No tasks in the list.");
            }
            else
            {
                Console.WriteLine("\nCurrent Tasks:");
                for (int i = 0; i < tasks.Count; i++)
                {
                    Console.WriteLine($"{i + 1}. {tasks[i]}");
                }
            }
            break;

        case "3":
            if (tasks.Count == 0)
            {
                Console.WriteLine("No tasks to remove.");
            }
            else
            {
                Console.WriteLine("\nCurrent Tasks:");
                for (int i = 0; i < tasks.Count; i++)
                {
                    Console.WriteLine($"{i + 1}. {tasks[i]}");
                }

                Console.Write("Enter the number of the task to remove: ");
                if (int.TryParse(Console.ReadLine(), out int taskNumber) &&
                    taskNumber >= 1 && taskNumber <= tasks.Count)
                {
                    string removedTask = tasks[taskNumber - 1];
                    tasks.RemoveAt(taskNumber - 1);
                    Console.WriteLine($"Task '{removedTask}' removed successfully!");
                }
                else
                {
                    Console.WriteLine("Invalid task number.");
                }
            }
            break;

        case "4":
            tasks.Clear();
            Console.WriteLine("All tasks cleared!");
            break;

        case "5":
            running = false;
            Console.WriteLine("Goodbye!");
            break;

        default:
            Console.WriteLine("Invalid choice. Please try again.");
            break;
    }
}
```

## Practice Exercise

Create a console application that:
1. Creates a List of integers
2. Asks the user to enter 5 numbers and adds them to the List
3. Displays the sum, average, minimum, and maximum of the numbers
4. Sorts the List and displays the sorted numbers
5. Asks the user for a number to search for and tells them if it's in the List
6. Removes all even numbers from the List and displays the remaining numbers

## Next Steps

In the next chapter, we'll explore dictionaries, another important collection type in C#. Dictionaries allow you to store key-value pairs and provide efficient lookups based on keys.

[Next Chapter: Dictionaries](./dictionaries.md)
