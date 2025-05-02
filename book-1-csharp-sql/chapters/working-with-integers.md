# Working with Numbers

In this chapter, we'll explore how to work with numeric types in C#, focusing on integers and other numeric data types. Understanding how to work with numbers is fundamental to programming in any language.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the different numeric types in C#
- Perform basic arithmetic operations
- Use mathematical functions and methods
- Convert between numeric types
- Handle numeric overflow and underflow
- Format numeric output for display

## Setting Up

Create a new console application to work with:

```bash
dotnet new console -n NumberExplorer
cd NumberExplorer
code .
```

## Numeric Types in C#

C# provides several numeric types to represent different kinds of numbers:

### Integer Types

Integer types store whole numbers without fractional parts:

| Type    | Size     | Range                                                   |
|---------|----------|--------------------------------------------------------|
| `sbyte` | 8 bits   | -128 to 127                                             |
| `byte`  | 8 bits   | 0 to 255                                                |
| `short` | 16 bits  | -32,768 to 32,767                                       |
| `ushort`| 16 bits  | 0 to 65,535                                             |
| `int`   | 32 bits  | -2,147,483,648 to 2,147,483,647                         |
| `uint`  | 32 bits  | 0 to 4,294,967,295                                      |
| `long`  | 64 bits  | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 |
| `ulong` | 64 bits  | 0 to 18,446,744,073,709,551,615                         |

### Floating-Point Types

Floating-point types store numbers with fractional parts:

| Type     | Size     | Precision                |
|----------|----------|--------------------------|
| `float`  | 32 bits  | ~6-9 digits (~7 decimal places)  |
| `double` | 64 bits  | ~15-17 digits (~15-16 decimal places) |

### Decimal Type

The `decimal` type is used for financial and monetary calculations where precision is critical:

| Type      | Size     | Precision               |
|-----------|----------|-------------------------|
| `decimal` | 128 bits | 28-29 significant digits |

## Declaring and Initializing Numeric Variables

Here's how to declare and initialize variables of different numeric types:

```csharp
// Integer types
byte smallUnsignedNumber = 255;
short smallNumber = -32000;
int number = 42;
long largeNumber = 9223372036854775807L; // Note the 'L' suffix for long literals

// Unsigned integer types
uint unsignedNumber = 4294967295U; // Note the 'U' suffix for uint literals
ulong largeUnsignedNumber = 18446744073709551615UL; // Note the 'UL' suffix for ulong literals

// Floating-point types
float singlePrecision = 3.14159F; // Note the 'F' suffix for float literals
double doublePrecision = 3.141592653589793;

// Decimal type
decimal moneyAmount = 1234.56M; // Note the 'M' suffix for decimal literals
```

## Basic Arithmetic Operations

C# supports the standard arithmetic operations:

```csharp
int a = 10;
int b = 3;

// Addition
int sum = a + b;
Console.WriteLine($"a + b = {sum}"); // Output: a + b = 13

// Subtraction
int difference = a - b;
Console.WriteLine($"a - b = {difference}"); // Output: a - b = 7

// Multiplication
int product = a * b;
Console.WriteLine($"a * b = {product}"); // Output: a * b = 30

// Division
int quotient = a / b;
Console.WriteLine($"a / b = {quotient}"); // Output: a / b = 3 (integer division)

// Remainder (modulo)
int remainder = a % b;
Console.WriteLine($"a % b = {remainder}"); // Output: a % b = 1
```

Note that when dividing two integers, the result is an integer (the fractional part is truncated). To get a floating-point result, at least one of the operands must be a floating-point type:

```csharp
int a = 10;
int b = 3;

// Integer division
int intDivision = a / b;
Console.WriteLine($"Integer division: {intDivision}"); // Output: Integer division: 3

// Floating-point division
double doubleDivision = (double)a / b;
Console.WriteLine($"Floating-point division: {doubleDivision}"); // Output: Floating-point division: 3.3333333333333335
```

## Compound Assignment Operators

C# provides compound assignment operators that combine an arithmetic operation with assignment:

```csharp
int x = 10;

// Addition assignment
x += 5; // Equivalent to: x = x + 5;
Console.WriteLine($"After x += 5: {x}"); // Output: After x += 5: 15

// Subtraction assignment
x -= 3; // Equivalent to: x = x - 3;
Console.WriteLine($"After x -= 3: {x}"); // Output: After x -= 3: 12

// Multiplication assignment
x *= 2; // Equivalent to: x = x * 2;
Console.WriteLine($"After x *= 2: {x}"); // Output: After x *= 2: 24

// Division assignment
x /= 4; // Equivalent to: x = x / 4;
Console.WriteLine($"After x /= 4: {x}"); // Output: After x /= 4: 6

// Remainder assignment
x %= 4; // Equivalent to: x = x % 4;
Console.WriteLine($"After x %= 4: {x}"); // Output: After x %= 4: 2
```

## Increment and Decrement Operators

C# provides increment (`++`) and decrement (`--`) operators:

```csharp
int i = 5;

// Postfix increment
int j = i++; // j = 5, i = 6
Console.WriteLine($"After postfix increment: j = {j}, i = {i}");

// Prefix increment
int k = ++i; // k = 7, i = 7
Console.WriteLine($"After prefix increment: k = {k}, i = {i}");

// Postfix decrement
int l = i--; // l = 7, i = 6
Console.WriteLine($"After postfix decrement: l = {l}, i = {i}");

// Prefix decrement
int m = --i; // m = 5, i = 5
Console.WriteLine($"After prefix decrement: m = {m}, i = {i}");
```

## Math Class

The `Math` class in the `System` namespace provides many useful mathematical functions:

```csharp
// Absolute value
Console.WriteLine($"Absolute value of -5: {Math.Abs(-5)}"); // Output: 5

// Power
Console.WriteLine($"2 raised to the power of 3: {Math.Pow(2, 3)}"); // Output: 8

// Square root
Console.WriteLine($"Square root of 16: {Math.Sqrt(16)}"); // Output: 4

// Rounding
Console.WriteLine($"Round 3.4: {Math.Round(3.4)}"); // Output: 3
Console.WriteLine($"Round 3.5: {Math.Round(3.5)}"); // Output: 4
Console.WriteLine($"Round 3.6: {Math.Round(3.6)}"); // Output: 4

// Ceiling (smallest integer greater than or equal to)
Console.WriteLine($"Ceiling of 3.1: {Math.Ceiling(3.1)}"); // Output: 4

// Floor (largest integer less than or equal to)
Console.WriteLine($"Floor of 3.9: {Math.Floor(3.9)}"); // Output: 3

// Maximum and minimum
Console.WriteLine($"Maximum of 5 and 10: {Math.Max(5, 10)}"); // Output: 10
Console.WriteLine($"Minimum of 5 and 10: {Math.Min(5, 10)}"); // Output: 5

// Trigonometric functions
Console.WriteLine($"Sine of 0: {Math.Sin(0)}"); // Output: 0
Console.WriteLine($"Cosine of 0: {Math.Cos(0)}"); // Output: 1
Console.WriteLine($"Tangent of 0: {Math.Tan(0)}"); // Output: 0

// Constants
Console.WriteLine($"PI: {Math.PI}"); // Output: 3.141592653589793
Console.WriteLine($"E: {Math.E}"); // Output: 2.718281828459045
```

## Type Conversion

You can convert between numeric types in several ways:

### Implicit Conversion

Implicit conversion happens automatically when there's no risk of data loss:

```csharp
byte smallNumber = 10;
int largerNumber = smallNumber; // Implicit conversion from byte to int
Console.WriteLine(largerNumber); // Output: 10

int intValue = 500;
long longValue = intValue; // Implicit conversion from int to long
Console.WriteLine(longValue); // Output: 500
```

### Explicit Conversion (Casting)

Explicit conversion (casting) is required when there's a risk of data loss:

```csharp
int intValue = 300;
byte byteValue = (byte)intValue; // Explicit conversion from int to byte
Console.WriteLine(byteValue); // Output: 44 (data loss: 300 % 256 = 44)

double doubleValue = 3.75;
int truncatedValue = (int)doubleValue; // Explicit conversion from double to int
Console.WriteLine(truncatedValue); // Output: 3 (fractional part is truncated)
```

### Using Conversion Methods

The `Convert` class provides methods for converting between types:

```csharp
string numberString = "123";
int parsedInt = Convert.ToInt32(numberString);
Console.WriteLine(parsedInt); // Output: 123

double doubleValue = 3.75;
int roundedInt = Convert.ToInt32(doubleValue); // Rounds to nearest integer
Console.WriteLine(roundedInt); // Output: 4 (rounded)
```

### Using Parse and TryParse

For converting strings to numeric types, you can use `Parse` or `TryParse`:

```csharp
// Using Parse (throws an exception if conversion fails)
string validNumber = "42";
int parsedNumber = int.Parse(validNumber);
Console.WriteLine(parsedNumber); // Output: 42

// Using TryParse (returns a boolean indicating success)
string input = "123";
if (int.TryParse(input, out int result))
{
    Console.WriteLine($"Conversion successful: {result}");
}
else
{
    Console.WriteLine("Conversion failed");
}
```

## Handling Numeric Overflow and Underflow

By default, C# doesn't check for overflow in arithmetic operations for performance reasons. However, you can enable overflow checking using the `checked` keyword:

```csharp
byte smallNumber = 255; // Maximum value for byte

// Without checked (overflow occurs silently)
byte result1 = (byte)(smallNumber + 1);
Console.WriteLine($"Without checked: {result1}"); // Output: 0 (overflow)

// With checked (throws an exception on overflow)
try
{
    byte result2 = checked((byte)(smallNumber + 1));
    Console.WriteLine($"With checked: {result2}");
}
catch (OverflowException ex)
{
    Console.WriteLine($"Overflow occurred: {ex.Message}");
}
```

You can also use the `unchecked` keyword to explicitly disable overflow checking in a checked context.

## Formatting Numeric Output

You can format numeric output using format specifiers:

```csharp
double value = 1234.5678;

// Currency format
Console.WriteLine($"Currency: {value:C}"); // Output: Currency: $1,234.57

// Decimal format with specified number of decimal places
Console.WriteLine($"Decimal (2 places): {value:F2}"); // Output: Decimal (2 places): 1234.57

// Number format with thousand separators
Console.WriteLine($"Number: {value:N}"); // Output: Number: 1,234.57

// Percent format
double percentage = 0.1234;
Console.WriteLine($"Percent: {percentage:P}"); // Output: Percent: 12.34%

// Custom format
Console.WriteLine($"Custom: {value:#,##0.000}"); // Output: Custom: 1,234.568
```

## Putting It All Together

Let's create a simple calculator application that demonstrates working with numbers:

```csharp
bool running = true;

Console.WriteLine("Simple Calculator");

while (running)
{
    Console.WriteLine("\nOptions:");
    Console.WriteLine("1. Addition");
    Console.WriteLine("2. Subtraction");
    Console.WriteLine("3. Multiplication");
    Console.WriteLine("4. Division");
    Console.WriteLine("5. Power");
    Console.WriteLine("6. Square Root");
    Console.WriteLine("7. Exit");
    Console.Write("Enter your choice (1-7): ");

    if (int.TryParse(Console.ReadLine(), out int choice))
    {
        double num1, num2, result;

        switch (choice)
        {
            case 1: // Addition
                Console.Write("Enter first number: ");
                if (double.TryParse(Console.ReadLine(), out num1))
                {
                    Console.Write("Enter second number: ");
                    if (double.TryParse(Console.ReadLine(), out num2))
                    {
                        result = num1 + num2;
                        Console.WriteLine($"{num1} + {num2} = {result}");
                    }
                    else
                    {
                        Console.WriteLine("Invalid second number.");
                    }
                }
                else
                {
                    Console.WriteLine("Invalid first number.");
                }
                break;

            case 2: // Subtraction
                Console.Write("Enter first number: ");
                if (double.TryParse(Console.ReadLine(), out num1))
                {
                    Console.Write("Enter second number: ");
                    if (double.TryParse(Console.ReadLine(), out num2))
                    {
                        result = num1 - num2;
                        Console.WriteLine($"{num1} - {num2} = {result}");
                    }
                    else
                    {
                        Console.WriteLine("Invalid second number.");
                    }
                }
                else
                {
                    Console.WriteLine("Invalid first number.");
                }
                break;

            case 3: // Multiplication
                Console.Write("Enter first number: ");
                if (double.TryParse(Console.ReadLine(), out num1))
                {
                    Console.Write("Enter second number: ");
                    if (double.TryParse(Console.ReadLine(), out num2))
                    {
                        result = num1 * num2;
                        Console.WriteLine($"{num1} * {num2} = {result}");
                    }
                    else
                    {
                        Console.WriteLine("Invalid second number.");
                    }
                }
                else
                {
                    Console.WriteLine("Invalid first number.");
                }
                break;

            case 4: // Division
                Console.Write("Enter first number: ");
                if (double.TryParse(Console.ReadLine(), out num1))
                {
                    Console.Write("Enter second number: ");
                    if (double.TryParse(Console.ReadLine(), out num2))
                    {
                        if (num2 != 0)
                        {
                            result = num1 / num2;
                            Console.WriteLine($"{num1} / {num2} = {result}");
                        }
                        else
                        {
                            Console.WriteLine("Cannot divide by zero.");
                        }
                    }
                    else
                    {
                        Console.WriteLine("Invalid second number.");
                    }
                }
                else
                {
                    Console.WriteLine("Invalid first number.");
                }
                break;

            case 5: // Power
                Console.Write("Enter base: ");
                if (double.TryParse(Console.ReadLine(), out num1))
                {
                    Console.Write("Enter exponent: ");
                    if (double.TryParse(Console.ReadLine(), out num2))
                    {
                        result = Math.Pow(num1, num2);
                        Console.WriteLine($"{num1} ^ {num2} = {result}");
                    }
                    else
                    {
                        Console.WriteLine("Invalid exponent.");
                    }
                }
                else
                {
                    Console.WriteLine("Invalid base.");
                }
                break;

            case 6: // Square Root
                Console.Write("Enter number: ");
                if (double.TryParse(Console.ReadLine(), out num1))
                {
                    if (num1 >= 0)
                    {
                        result = Math.Sqrt(num1);
                        Console.WriteLine($"Square root of {num1} = {result}");
                    }
                    else
                    {
                        Console.WriteLine("Cannot calculate square root of a negative number.");
                    }
                }
                else
                {
                    Console.WriteLine("Invalid number.");
                }
                break;

            case 7: // Exit
                running = false;
                Console.WriteLine("Goodbye!");
                break;

            default:
                Console.WriteLine("Invalid choice. Please try again.");
                break;
        }
    }
    else
    {
        Console.WriteLine("Invalid choice. Please enter a number between 1 and 7.");
    }
}
```

## Practice Exercise

Create a console application that:
1. Asks the user to enter a starting balance for a bank account
2. Provides options to deposit money, withdraw money, or check the balance
3. Calculates and adds interest (e.g., 2% of the current balance) when requested
4. Formats all monetary values as currency
5. Prevents the account from being overdrawn (balance going below zero)
6. Keeps track of the number of transactions and calculates the average transaction amount

## Next Steps

In the next chapter, we'll explore working with dates and times in C#. You'll learn how to create, manipulate, and format DateTime objects.

Before moving on, make sure you're comfortable with:
- Understanding the different numeric types in C#
- Performing basic arithmetic operations
- Using mathematical functions from the Math class
- Converting between numeric types
- Handling numeric overflow and underflow
- Formatting numeric output for display