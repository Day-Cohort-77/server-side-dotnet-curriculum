# Exception Handling

In this chapter, we'll explore exception handling in C#. Exceptions are unexpected events that occur during program execution, such as attempting to divide by zero or accessing a file that doesn't exist. Proper exception handling is crucial for building robust applications.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand what exceptions are and why they occur
- Use try-catch blocks to handle exceptions
- Work with multiple catch blocks for different exception types
- Use the finally block for cleanup operations
- Create and throw custom exceptions
- Implement proper exception handling strategies

## Setting Up

Create a new console application to work with:

```bash
dotnet new console -n ExceptionHandling
cd ExceptionHandling
code .
```

## Understanding Exceptions

An exception is an object that represents an error or unexpected condition that occurs during program execution. When an exception occurs, the normal flow of the program is disrupted, and the runtime looks for an exception handler to deal with the situation.

If no handler is found, the program terminates with an error message. By handling exceptions, you can gracefully recover from errors and provide meaningful feedback to users.

### Common Exception Types

C# provides many built-in exception types, including:

- `ArgumentException`: Thrown when a method is called with an invalid argument
- `ArgumentNullException`: Thrown when a null argument is passed to a method that doesn't accept it
- `DivideByZeroException`: Thrown when an attempt is made to divide by zero
- `FileNotFoundException`: Thrown when an attempt is made to access a file that doesn't exist
- `FormatException`: Thrown when the format of an argument is invalid
- `IndexOutOfRangeException`: Thrown when an array index is outside the bounds of the array
- `InvalidOperationException`: Thrown when a method call is invalid in the object's current state
- `NullReferenceException`: Thrown when there's an attempt to access a member on a null reference
- `OverflowException`: Thrown when an arithmetic operation results in an overflow

All exception types derive from the `System.Exception` base class.

## Basic Exception Handling

The basic structure for exception handling in C# is the try-catch block:

```csharp
try
{
    // Code that might throw an exception
}
catch (Exception ex)
{
    // Code to handle the exception
    Console.WriteLine($"An error occurred: {ex.Message}");
}
```

Let's see a simple example:

```csharp
try
{
    Console.Write("Enter a number: ");
    string input = Console.ReadLine();
    int number = int.Parse(input);
    Console.WriteLine($"You entered: {number}");
}
catch (Exception ex)
{
    Console.WriteLine($"An error occurred: {ex.Message}");
}
```

If the user enters something that can't be parsed as an integer (like "abc"), a `FormatException` will be thrown, and the catch block will handle it.

## Catching Specific Exception Types

You can catch specific types of exceptions by specifying the exception type in the catch block:

```csharp
try
{
    Console.Write("Enter a number: ");
    string input = Console.ReadLine();
    int number = int.Parse(input);

    Console.Write("Enter another number: ");
    string input2 = Console.ReadLine();
    int number2 = int.Parse(input2);

    int result = number / number2;
    Console.WriteLine($"{number} / {number2} = {result}");
}
catch (FormatException ex)
{
    Console.WriteLine($"Invalid format: {ex.Message}");
    Console.WriteLine("Please enter a valid integer.");
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Division by zero: {ex.Message}");
    Console.WriteLine("The second number cannot be zero.");
}
catch (Exception ex)
{
    Console.WriteLine($"An unexpected error occurred: {ex.Message}");
}
```

In this example, we're handling three types of exceptions:
1. `FormatException`: Thrown if the user enters a non-integer value
2. `DivideByZeroException`: Thrown if the user enters 0 for the second number
3. `Exception`: A catch-all for any other exceptions that might occur

The catch blocks are evaluated in order, so it's important to place more specific exception types before more general ones. If you put `Exception` first, it would catch all exceptions, and the more specific catch blocks would never be reached.

## The finally Block

The `finally` block contains code that is executed regardless of whether an exception is thrown or not. It's typically used for cleanup operations, such as closing files or releasing resources:

```csharp
StreamReader reader = null;
try
{
    reader = new StreamReader("example.txt");
    string content = reader.ReadToEnd();
    Console.WriteLine(content);
}
catch (FileNotFoundException ex)
{
    Console.WriteLine($"File not found: {ex.Message}");
}
catch (IOException ex)
{
    Console.WriteLine($"IO error: {ex.Message}");
}
finally
{
    // This code runs whether an exception occurred or not
    if (reader != null)
    {
        reader.Close();
        Console.WriteLine("StreamReader closed.");
    }
}
```

In this example, the `finally` block ensures that the `StreamReader` is closed, even if an exception occurs while reading the file.

## Using the using Statement

For objects that implement `IDisposable` (like file streams, database connections, etc.), you can use the `using` statement to automatically dispose of the object when it's no longer needed:

```csharp
try
{
    using (StreamReader reader = new StreamReader("example.txt"))
    {
        string content = reader.ReadToEnd();
        Console.WriteLine(content);
    } // The StreamReader is automatically closed here
}
catch (FileNotFoundException ex)
{
    Console.WriteLine($"File not found: {ex.Message}");
}
catch (IOException ex)
{
    Console.WriteLine($"IO error: {ex.Message}");
}
```

The `using` statement is equivalent to a try-finally block where the `Dispose` method is called in the `finally` block.

## Exception Properties

The `Exception` class provides several properties that can help you diagnose and handle exceptions:

```csharp
try
{
    // Code that might throw an exception
    int[] numbers = { 1, 2, 3 };
    Console.WriteLine(numbers[10]); // This will throw an IndexOutOfRangeException
}
catch (Exception ex)
{
    Console.WriteLine($"Message: {ex.Message}"); // A description of the error
    Console.WriteLine($"Source: {ex.Source}"); // The name of the object that caused the error
    Console.WriteLine($"StackTrace: {ex.StackTrace}"); // A stack trace showing where the error occurred
    Console.WriteLine($"TargetSite: {ex.TargetSite}"); // The method that threw the exception

    if (ex.InnerException != null)
    {
        Console.WriteLine($"Inner Exception: {ex.InnerException.Message}");
    }
}
```

## Throwing Exceptions

You can throw exceptions in your code using the `throw` keyword:

```csharp
static void ValidateAge(int age)
{
    if (age < 0)
    {
        throw new ArgumentException("Age cannot be negative.");
    }

    if (age > 120)
    {
        throw new ArgumentException("Age is unrealistically high.");
    }

    Console.WriteLine($"Age {age} is valid.");
}

try
{
    ValidateAge(-5);
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Validation error: {ex.Message}");
}
```

You can also re-throw an exception to pass it up the call stack:

```csharp
try
{
    // Some code that might throw an exception
}
catch (Exception ex)
{
    // Log the exception
    Console.WriteLine($"Logging error: {ex.Message}");

    // Re-throw the exception
    throw;
}
```

Note that using `throw;` preserves the original stack trace, while `throw ex;` resets the stack trace to the current location.

## Creating Custom Exceptions

You can create your own exception types by deriving from `Exception` or a more specific exception type:

```csharp
// Custom exception class
public class AgeException : Exception
{
    public int InvalidAge { get; }

    public AgeException(string message, int invalidAge) : base(message)
    {
        InvalidAge = invalidAge;
    }

    public AgeException(string message, int invalidAge, Exception innerException)
        : base(message, innerException)
    {
        InvalidAge = invalidAge;
    }
}

// Using the custom exception
static void ValidateAge(int age)
{
    if (age < 0)
    {
        throw new AgeException("Age cannot be negative.", age);
    }

    if (age > 120)
    {
        throw new AgeException("Age is unrealistically high.", age);
    }

    Console.WriteLine($"Age {age} is valid.");
}

try
{
    ValidateAge(-5);
}
catch (AgeException ex)
{
    Console.WriteLine($"Age validation error: {ex.Message}");
    Console.WriteLine($"Invalid age provided: {ex.InvalidAge}");
}
```

Custom exceptions should:
- Have a name that ends with "Exception"
- Derive from `Exception` or a more specific exception type
- Provide constructors that match the base class
- Be serializable (if they need to cross application domain boundaries)

## Exception Handling Best Practices

Here are some best practices for exception handling in C#:

1. **Be specific**: Catch specific exception types rather than using a generic catch-all.
2. **Don't catch exceptions you can't handle**: If you can't do anything meaningful with an exception, let it propagate up the call stack.
3. **Clean up resources**: Use `finally` blocks or `using` statements to ensure resources are properly released.
4. **Provide meaningful error messages**: Make your exception messages clear and informative.
5. **Log exceptions**: Log exceptions for debugging and monitoring purposes.
6. **Don't use exceptions for normal flow control**: Exceptions should be exceptional; don't use them for expected conditions.
7. **Throw the most appropriate exception type**: Use built-in exception types when possible, or create custom ones when needed.
8. **Include context information**: When throwing exceptions, include relevant information that will help diagnose the problem.

## Putting It All Together

Let's create a simple calculator application that demonstrates exception handling:

```csharp
bool running = true;

Console.WriteLine("Safe Calculator");
Console.WriteLine("This calculator handles errors gracefully.");

while (running)
{
    try
    {
        Console.WriteLine("\nOptions:");
        Console.WriteLine("1. Addition");
        Console.WriteLine("2. Subtraction");
        Console.WriteLine("3. Multiplication");
        Console.WriteLine("4. Division");
        Console.WriteLine("5. Exit");
        Console.Write("Enter your choice (1-5): ");

        if (!int.TryParse(Console.ReadLine(), out int choice))
        {
            throw new FormatException("Invalid choice. Please enter a number between 1 and 5.");
        }

        if (choice < 1 || choice > 5)
        {
            throw new ArgumentOutOfRangeException("choice", "Choice must be between 1 and 5.");
        }

        if (choice == 5)
        {
            running = false;
            Console.WriteLine("Goodbye!");
            continue;
        }

        Console.Write("Enter first number: ");
        if (!double.TryParse(Console.ReadLine(), out double num1))
        {
            throw new FormatException("Invalid number format for the first number.");
        }

        Console.Write("Enter second number: ");
        if (!double.TryParse(Console.ReadLine(), out double num2))
        {
            throw new FormatException("Invalid number format for the second number.");
        }

        double result = 0;
        string operation = "";

        switch (choice)
        {
            case 1: // Addition
                result = num1 + num2;
                operation = "+";
                break;
            case 2: // Subtraction
                result = num1 - num2;
                operation = "-";
                break;
            case 3: // Multiplication
                result = num1 * num2;
                operation = "*";
                break;
            case 4: // Division
                if (num2 == 0)
                {
                    throw new DivideByZeroException("Cannot divide by zero.");
                }
                result = num1 / num2;
                operation = "/";
                break;
        }

        Console.WriteLine($"Result: {num1} {operation} {num2} = {result}");
    }
    catch (FormatException ex)
    {
        Console.WriteLine($"Input error: {ex.Message}");
        Console.WriteLine("Please enter valid numeric values.");
    }
    catch (DivideByZeroException ex)
    {
        Console.WriteLine($"Math error: {ex.Message}");
        Console.WriteLine("Please enter a non-zero value for the second number when dividing.");
    }
    catch (ArgumentOutOfRangeException ex)
    {
        Console.WriteLine($"Input error: {ex.Message}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"An unexpected error occurred: {ex.Message}");
        Console.WriteLine("Please try again.");
    }
    finally
    {
        Console.WriteLine("\nPress any key to continue...");
        Console.ReadKey(true);
        Console.Clear();
    }
}
```

This calculator application demonstrates several exception handling techniques:
- Catching specific exception types
- Throwing exceptions with meaningful messages
- Using a finally block for cleanup
- Providing user-friendly error messages

## Practice Exercise

Create a console application that:
1. Asks the user to enter a file path
2. Attempts to read the contents of the file
3. Handles different exceptions that might occur (file not found, access denied, etc.)
4. Allows the user to retry with a different file path
5. Counts and displays the number of words in the file if it's successfully read
6. Implements proper resource cleanup using `using` statements or `finally` blocks

## Next Steps

In the next chapter, we'll explore interfaces and abstraction in C#. You'll learn how to define and implement interfaces, and how to use them to create more flexible and maintainable code.

Before moving on, make sure you're comfortable with:
- Understanding what exceptions are and why they occur
- Using try-catch blocks to handle exceptions
- Working with multiple catch blocks for different exception types
- Using the finally block for cleanup operations
- Creating and throwing custom exceptions
- Implementing proper exception handling strategies