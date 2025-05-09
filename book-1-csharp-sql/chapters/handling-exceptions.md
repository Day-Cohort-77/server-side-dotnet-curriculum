# Exception Handling

In this chapter, we'll explore exception handling in C#. Exceptions are unexpected events that occur during program execution, such as attempting to divide by zero or accessing a file that doesn't exist. Proper exception handling is crucial for building robust applications.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand what exceptions are and why they occur
- Use try-catch blocks to handle exceptions
- Work with multiple catch blocks for different exception types
- Use the finally block for cleanup operations
- Implement proper exception handling strategies

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

The `finally` block contains code that is executed regardless of whether an exception is thrown or not. It's typically used for cleanup operations:

```csharp
try
{
    // Code that might throw an exception
    Console.Write("Enter a number to divide 100 by: ");
    string input = Console.ReadLine();
    int divisor = int.Parse(input);
    int result = 100 / divisor;
    Console.WriteLine($"100 / {divisor} = {result}");
}
catch (FormatException ex)
{
    Console.WriteLine($"Invalid format: {ex.Message}");
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
finally
{
    // This code runs whether an exception occurred or not
    Console.WriteLine("This is the finally block - it always executes!");
    Console.WriteLine("It's useful for cleanup operations that should happen regardless of errors.");
}
```

In this example, the `finally` block will execute regardless of whether the user enters a valid number, an invalid format, or zero.

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

## Exception Handling Best Practices

Here are some best practices for exception handling in C#:

1. **Be specific**: Catch specific exception types rather than using a generic catch-all.
2. **Don't catch exceptions you can't handle**: If you can't do anything meaningful with an exception, let it propagate up the call stack.
3. **Clean up resources**: Use `finally` blocks to ensure resources are properly released.
4. **Provide meaningful error messages**: Make your exception messages clear and informative.
5. **Log exceptions**: Log exceptions for debugging and monitoring purposes.
6. **Don't use exceptions for normal flow control**: Exceptions should be exceptional; don't use them for expected conditions.
7. **Include context information**: When handling exceptions, include relevant information that will help diagnose the problem.

## Practice: Adding Exception Handling

Let's practice adding exception handling to a simple calculator application. Create a new console application:

```bash
dotnet new console -n ExceptionHandlingPractice
cd ExceptionHandlingPractice
code .
```

Replace the contents of `Program.cs` with the following code that has NO exception handling:

```csharp
// Calculator without exception handling
Console.WriteLine("Simple Calculator");

bool running = true;

while (running)
{
    Console.WriteLine("\nOptions:");
    Console.WriteLine("1. Addition");
    Console.WriteLine("2. Subtraction");
    Console.WriteLine("3. Multiplication");
    Console.WriteLine("4. Division");
    Console.WriteLine("5. Exit");
    Console.Write("Enter your choice (1-5): ");

    int choice = int.Parse(Console.ReadLine());

    if (choice == 5)
    {
        running = false;
        Console.WriteLine("Goodbye!");
        continue;
    }

    Console.Write("Enter first number: ");
    double num1 = double.Parse(Console.ReadLine());

    Console.Write("Enter second number: ");
    double num2 = double.Parse(Console.ReadLine());

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
            result = num1 / num2;
            operation = "/";
            break;
        default:
            Console.WriteLine("Invalid choice!");
            continue;
    }

    Console.WriteLine($"Result: {num1} {operation} {num2} = {result}");

    Console.WriteLine("\nPress any key to continue...");
    Console.ReadKey(true);
    Console.Clear();
}
```

Now, modify this code to add proper exception handling:

1. Add a try-catch block around the code that might throw exceptions
2. Handle specific exceptions:
   - `FormatException` when the user enters non-numeric values
   - `DivideByZeroException` when dividing by zero
   - `ArgumentOutOfRangeException` for invalid menu choices
   - A general `Exception` catch-all for any other unexpected errors
3. Add a finally block to handle cleanup operations
4. Provide user-friendly error messages

Your improved code should:
- Prevent the application from crashing when users enter invalid input
- Provide helpful error messages to guide users
- Continue running even after errors occur
- Use the finally block to ensure the "Press any key to continue" message always appears

Test your application with various inputs, including:
- Letters instead of numbers
- Zero as the second number when dividing
- Numbers outside the valid range for menu choices

## Next Steps

In the next chapter, we'll explore interfaces and abstraction in C#. You'll learn how to define and implement interfaces, and how to use them to create more flexible and maintainable code.

Before moving on, make sure you're comfortable with:
- Understanding what exceptions are and why they occur
- Using try-catch blocks to handle exceptions
- Working with multiple catch blocks for different exception types
- Using the finally block for cleanup operations
- Implementing proper exception handling strategies