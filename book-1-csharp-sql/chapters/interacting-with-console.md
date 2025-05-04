# Strings and Console Interaction

In this chapter, we'll explore how to work with strings in C# and how to interact with the console for input and output operations.

## Learning Objectives

By the end of this chapter, you should be able to:
- Work with string variables and string literals
- Use string concatenation and interpolation
- Format strings for display
- Get different types of input from the console
- Display formatted output to the console

## Working with Strings

Strings in C# are sequences of characters enclosed in double quotes. Let's create a new console application to explore strings:

```bash
dotnet new console -n StringExplorer
cd StringExplorer
code .
```

### String Declaration and Initialization

There are several ways to declare and initialize strings in C#:

```csharp
// String declaration and initialization
string greeting = "Hello, World!";
string name = "John";
string emptyString = "";
string nullString = null;
string verbatim = @"This is a verbatim string literal.
It can span multiple lines.";
```

### String Concatenation

You can combine strings using the `+` operator:

```csharp
string firstName = "John";
string lastName = "Doe";
string fullName = firstName + " " + lastName;
Console.WriteLine(fullName); // Outputs: John Doe
```

### String Interpolation

String interpolation provides a more readable way to include variable values within string literals:

```csharp
string firstName = "John";
string lastName = "Doe";
int age = 30;
string message = $"Name: {firstName} {lastName}, Age: {age}";
Console.WriteLine(message); // Outputs: Name: John Doe, Age: 30
```

### String Methods

C# strings come with many useful methods:

```csharp
string sample = "Hello, World!";

// Length property
Console.WriteLine($"Length: {sample.Length}"); // Outputs: Length: 13

// ToUpper and ToLower methods
Console.WriteLine(sample.ToUpper()); // Outputs: HELLO, WORLD!
Console.WriteLine(sample.ToLower()); // Outputs: hello, world!

// Substring method
Console.WriteLine(sample.Substring(0, 5)); // Outputs: Hello

// Replace method
Console.WriteLine(sample.Replace("Hello", "Hi")); // Outputs: Hi, World!

// Contains method
Console.WriteLine(sample.Contains("World")); // Outputs: True

// StartsWith and EndsWith methods
Console.WriteLine(sample.StartsWith("Hello")); // Outputs: True
Console.WriteLine(sample.EndsWith("!")); // Outputs: True

// Trim, TrimStart, and TrimEnd methods
string paddedText = "   Padded Text   ";
Console.WriteLine($"Original: '{paddedText}'");
Console.WriteLine($"Trimmed: '{paddedText.Trim()}'");
Console.WriteLine($"TrimStart: '{paddedText.TrimStart()}'");
Console.WriteLine($"TrimEnd: '{paddedText.TrimEnd()}'");
```

## Console Interaction

The `Console` class provides methods for interacting with the console window.

### Console Output

We've already seen `Console.WriteLine()`, which outputs text followed by a newline. There's also `Console.Write()`, which outputs text without a newline:

```csharp
Console.Write("This is on ");
Console.Write("the same line.");
Console.WriteLine(); // Move to the next line
Console.WriteLine("This is on a new line.");
```

### Formatting Console Output

You can format console output using string interpolation or the `string.Format()` method:

```csharp
double price = 19.99;
int quantity = 5;
double total = price * quantity;

// Using string interpolation
Console.WriteLine($"Total: ${total}");

// Using string.Format
Console.WriteLine(string.Format("Total: ${0}", total));

// Formatting numbers
Console.WriteLine($"Total: ${total:F2}"); // F2 formats with 2 decimal places
Console.WriteLine($"Total: {total:C}"); // C formats as currency
```

### Console Input

The `Console.ReadLine()` method reads a line of text from the console:

```csharp
Console.Write("Enter your name: ");
string name = Console.ReadLine();
Console.WriteLine($"Hello, {name}!");
```

To read other data types, you need to parse the input:

```csharp
Console.Write("Enter your age: ");
string ageInput = Console.ReadLine();
int age = int.Parse(ageInput);
Console.WriteLine($"In 10 years, you'll be {age + 10} years old.");
```

A safer way to parse input is using the `TryParse` method, which doesn't throw an exception if the input is invalid:

```csharp
Console.Write("Enter your age: ");
string ageInput = Console.ReadLine();
if (int.TryParse(ageInput, out int age))
{
    Console.WriteLine($"In 10 years, you'll be {age + 10} years old.");
}
else
{
    Console.WriteLine("That's not a valid age!");
}
```

### Reading Individual Keys

You can also read individual keystrokes using `Console.ReadKey()`:

```csharp
Console.WriteLine("Press any key to continue...");
ConsoleKeyInfo keyInfo = Console.ReadKey();
Console.WriteLine($"\nYou pressed: {keyInfo.KeyChar}");
```

## Putting It All Together

Let's create a simple console application that demonstrates these concepts:

```csharp
Console.WriteLine("Welcome to the String Explorer!");
Console.Write("Enter your first name: ");
string firstName = Console.ReadLine();

Console.Write("Enter your last name: ");
string lastName = Console.ReadLine();

string fullName = $"{firstName} {lastName}";
Console.WriteLine($"Hello, {fullName}!");

Console.Write("Enter your birth year: ");
if (int.TryParse(Console.ReadLine(), out int birthYear))
{
    int currentYear = DateTime.Now.Year;
    int approximateAge = currentYear - birthYear;
    Console.WriteLine($"You are approximately {approximateAge} years old.");
}
else
{
    Console.WriteLine("That's not a valid year!");
}

Console.WriteLine("\nString information:");
Console.WriteLine($"Your full name is {fullName.Length} characters long.");
Console.WriteLine($"Uppercase: {fullName.ToUpper()}");
Console.WriteLine($"Lowercase: {fullName.ToLower()}");

if (firstName.Length > 5)
{
    Console.WriteLine($"Your first name has more than 5 characters.");
}
else
{
    Console.WriteLine($"Your first name has 5 or fewer characters.");
}

Console.WriteLine("\nPress any key to exit...");
Console.ReadKey();
```

## Practice Exercise

Create a console application that:
1. Asks the user for their full name
2. Asks the user for their favorite color
3. Asks the user for their favorite number
4. Displays a personalized message using all this information
5. Tells the user if their name `Contains()` the letter 'a'
6. Displays their name in reverse order _(hint: you can use a loop or the `Reverse()` method after converting the string `ToCharArray()`)_

## Next Steps

In the next chapter, we'll explore control flow in C# with conditionals and loops. You'll learn how to make decisions in your code and how to repeat operations.

Before moving on, make sure you're comfortable with:
- Working with strings and string methods
- Getting input from the console
- Displaying formatted output to the console
- Parsing different data types from string input