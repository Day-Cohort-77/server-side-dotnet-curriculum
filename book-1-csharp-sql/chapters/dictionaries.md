# Collections: Dictionaries

In this chapter, we'll explore dictionaries, a powerful collection type in C# that stores key-value pairs. Dictionaries provide efficient lookups based on keys and are essential for many programming tasks.

## Learning Objectives

By the end of this chapter, you should be able to:
- Create and initialize dictionaries
- Add, remove, and modify key-value pairs
- Access values using keys
- Check if keys exist in a dictionary
- Iterate through dictionary keys, values, and key-value pairs
- Understand when to use dictionaries instead of lists or arrays

## Setting Up

Create a new console application to work with:

```bash
dotnet new console -n DictionaryExplorer
cd DictionaryExplorer
code .
```

## Introduction to Dictionaries

A dictionary is a collection of key-value pairs where each key is unique. Dictionaries provide fast lookups based on keys, making them ideal when you need to quickly find values associated with specific identifiers.

Dictionaries are defined in the `System.Collections.Generic` namespace as the `Dictionary<TKey, TValue>` class, where `TKey` is the type of the keys and `TValue` is the type of the values.

### Creating Dictionaries

To use dictionaries, include the appropriate namespace:

```csharp
using System.Collections.Generic;
```

Here are different ways to create and initialize dictionaries:

```csharp
// Create an empty dictionary with string keys and int values
Dictionary<string, int> ages = new Dictionary<string, int>();

// Create and initialize a dictionary in one line
Dictionary<string, string> capitals = new Dictionary<string, string>
{
    { "USA", "Washington D.C." },
    { "UK", "London" },
    { "France", "Paris" }
};

// Alternative initialization syntax
Dictionary<string, string> countryCodes = new Dictionary<string, string>
{
    ["USA"] = "US",
    ["United Kingdom"] = "GB",
    ["France"] = "FR"
};
```

## Adding Elements to a Dictionary

You can add elements to a dictionary using the `Add` method or the indexer syntax:

```csharp
Dictionary<string, int> ages = new Dictionary<string, int>();

// Using the Add method
ages.Add("Alice", 30);
ages.Add("Bob", 25);

// Using indexer syntax
ages["Charlie"] = 35;
ages["David"] = 40;

Console.WriteLine($"Count: {ages.Count}"); // Output: Count: 4
```

Note that if you try to add a key that already exists using the `Add` method, an exception will be thrown. However, using the indexer syntax will update the value if the key already exists.

```csharp
// This will update the existing value
ages["Alice"] = 31;

// This would throw an exception if uncommented
// ages.Add("Alice", 32);
```

## Accessing Values in a Dictionary

You can access values in a dictionary using the key:

```csharp
Dictionary<string, string> capitals = new Dictionary<string, string>
{
    { "USA", "Washington D.C." },
    { "UK", "London" },
    { "France", "Paris" }
};

Console.WriteLine(capitals["USA"]); // Output: Washington D.C.
Console.WriteLine(capitals["UK"]); // Output: London
```

If you try to access a key that doesn't exist, an exception will be thrown. To avoid this, you can check if a key exists first or use the `TryGetValue` method:

```csharp
// Check if key exists before accessing
if (capitals.ContainsKey("Germany"))
{
    Console.WriteLine(capitals["Germany"]);
}
else
{
    Console.WriteLine("Germany is not in the dictionary.");
}

// Using TryGetValue
if (capitals.TryGetValue("Germany", out string germanCapital))
{
    Console.WriteLine(germanCapital);
}
else
{
    Console.WriteLine("Germany is not in the dictionary.");
}
```

## Removing Elements from a Dictionary

You can remove elements from a dictionary using the `Remove` method:

```csharp
Dictionary<string, string> capitals = new Dictionary<string, string>
{
    { "USA", "Washington D.C." },
    { "UK", "London" },
    { "France", "Paris" }
};

// Remove a specific key-value pair
bool removed = capitals.Remove("UK");
Console.WriteLine($"UK removed: {removed}"); // Output: UK removed: True
Console.WriteLine($"Count after removal: {capitals.Count}"); // Output: Count after removal: 2

// Attempting to remove a non-existent key
bool removedGermany = capitals.Remove("Germany");
Console.WriteLine($"Germany removed: {removedGermany}"); // Output: Germany removed: False

// Clear all elements
capitals.Clear();
Console.WriteLine($"Count after Clear: {capitals.Count}"); // Output: Count after Clear: 0
```

## Checking if a Key Exists

You can check if a key exists in a dictionary using the `ContainsKey` method:

```csharp
Dictionary<string, string> capitals = new Dictionary<string, string>
{
    { "USA", "Washington D.C." },
    { "UK", "London" },
    { "France", "Paris" }
};

bool hasUSA = capitals.ContainsKey("USA");
Console.WriteLine($"Contains USA: {hasUSA}"); // Output: Contains USA: True

bool hasGermany = capitals.ContainsKey("Germany");
Console.WriteLine($"Contains Germany: {hasGermany}"); // Output: Contains Germany: False
```

## Checking if a Value Exists

You can check if a value exists in a dictionary using the `ContainsValue` method:

```csharp
Dictionary<string, string> capitals = new Dictionary<string, string>
{
    { "USA", "Washington D.C." },
    { "UK", "London" },
    { "France", "Paris" }
};

bool hasLondon = capitals.ContainsValue("London");
Console.WriteLine($"Contains London: {hasLondon}"); // Output: Contains London: True

bool hasBerlin = capitals.ContainsValue("Berlin");
Console.WriteLine($"Contains Berlin: {hasBerlin}"); // Output: Contains Berlin: False
```

Note that `ContainsValue` is less efficient than `ContainsKey` because it needs to search through all values in the dictionary.

## Iterating Through a Dictionary

You can iterate through a dictionary in several ways:

### Iterating Through Key-Value Pairs

```csharp
Dictionary<string, string> capitals = new Dictionary<string, string>
{
    { "USA", "Washington D.C." },
    { "UK", "London" },
    { "France", "Paris" }
};

foreach (KeyValuePair<string, string> pair in capitals)
{
    Console.WriteLine($"The capital of {pair.Key} is {pair.Value}");
}
```

You can also use the `var` keyword for more concise code:

```csharp
foreach (var pair in capitals)
{
    Console.WriteLine($"The capital of {pair.Key} is {pair.Value}");
}
```

### Iterating Through Keys

```csharp
foreach (string country in capitals.Keys)
{
    Console.WriteLine(country);
}
```

### Iterating Through Values

```csharp
foreach (string capital in capitals.Values)
{
    Console.WriteLine(capital);
}
```

## Dictionary with Complex Types

You can use any type as a key or value in a dictionary, including custom classes:

```csharp
// Create a dictionary with string keys and Person values
Dictionary<string, Person> people = new Dictionary<string, Person>
{
    { "employee1", new Person { Name = "Alice", Age = 30 } },
    { "employee2", new Person { Name = "Bob", Age = 25 } },
    { "employee3", new Person { Name = "Charlie", Age = 35 } }
};

// Access and display a person
Person employee2 = people["employee2"];
Console.WriteLine(employee2); // Output: Bob (25)

// Iterate through the dictionary
foreach (var pair in people)
{
    Console.WriteLine($"ID: {pair.Key}, Person: {pair.Value}");
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

## When to Use Dictionaries

Dictionaries are ideal when:

- You need to look up values based on a unique key
- You need to check if a specific key exists
- You need to associate values with identifiers
- You need fast lookups (dictionaries provide O(1) average case lookup time)

Lists are better when:

- You need to maintain a specific order of elements
- You need to access elements by index
- You need to store duplicate values
- You need to frequently iterate through all elements

## Practice Exercise

Create a console application that:

1. Creates a dictionary to store student grades (student name as key, grade as value)
2. Allows the user to add, view, update, and remove student grades
3. Calculates and displays the average grade
4. Displays the student with the highest grade
5. Displays the student with the lowest grade
6. Counts how many students have a passing grade (60 or higher)

## Next Steps

In the next chapter, we'll explore how to create custom types using the `class` keyword in C#.

[Next Chapter: Custom Types with Classes](./custom-types-with-classes.md)
