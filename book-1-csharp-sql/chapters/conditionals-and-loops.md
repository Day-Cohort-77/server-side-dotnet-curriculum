# Loops and Conditions

In this chapter, we'll explore control flow in C# using conditional statements and loops. These constructs allow your programs to make decisions and repeat operations, which are fundamental to writing useful applications.

## Learning Objectives

By the end of this chapter, you should be able to:
- Use conditional statements (if, else, switch) to make decisions in code
- Implement different types of loops (for, while, do-while, foreach)
- Understand logical operators and how to combine conditions
- Use break and continue statements to control loop execution

## Setting Up

Create a new console application to work with:

```bash
dotnet new console -n ControlFlow
cd ControlFlow
code .
```

## Conditional Statements

Conditional statements allow your program to make decisions based on certain conditions.

### If Statement

The `if` statement executes a block of code if a specified condition is true:

```csharp
int age = 20;

if (age >= 18)
{
    Console.WriteLine("You are an adult.");
}
```

### If-Else Statement

The `if-else` statement executes one block of code if a condition is true and another block if the condition is false:

```csharp
int age = 15;

if (age >= 18)
{
    Console.WriteLine("You are an adult.");
}
else
{
    Console.WriteLine("You are a minor.");
}
```

### If-Else If-Else Statement

You can chain multiple conditions using `else if`:

```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("Grade: A");
}
else if (score >= 80)
{
    Console.WriteLine("Grade: B");
}
else if (score >= 70)
{
    Console.WriteLine("Grade: C");
}
else if (score >= 60)
{
    Console.WriteLine("Grade: D");
}
else
{
    Console.WriteLine("Grade: F");
}
```

### Nested If Statements

You can nest if statements inside other if statements:

```csharp
bool isWeekend = true;
bool isRaining = false;

if (isWeekend)
{
    if (isRaining)
    {
        Console.WriteLine("Stay home and watch a movie.");
    }
    else
    {
        Console.WriteLine("Go out and enjoy the day!");
    }
}
else
{
    Console.WriteLine("It's a workday.");
}
```

### Switch Statement

The `switch` statement is useful when you have multiple possible values for a single variable:

```csharp
int day = 3;
string dayName;

switch (day)
{
    case 1:
        dayName = "Monday";
        break;
    case 2:
        dayName = "Tuesday";
        break;
    case 3:
        dayName = "Wednesday";
        break;
    case 4:
        dayName = "Thursday";
        break;
    case 5:
        dayName = "Friday";
        break;
    case 6:
        dayName = "Saturday";
        break;
    case 7:
        dayName = "Sunday";
        break;
    default:
        dayName = "Invalid day";
        break;
}

Console.WriteLine($"Day {day} is {dayName}");
```

### Ternary Operator

The ternary operator is a shorthand way to write simple if-else statements:

```csharp
int age = 20;
string status = (age >= 18) ? "adult" : "minor";
Console.WriteLine($"You are a {status}.");
```

## Logical Operators

Logical operators allow you to combine multiple conditions:

- `&&` (AND): Both conditions must be true
- `||` (OR): At least one condition must be true
- `!` (NOT): Inverts a boolean value

```csharp
bool isWeekend = true;
bool isRaining = false;

if (isWeekend && !isRaining)
{
    Console.WriteLine("Perfect day for outdoor activities!");
}

int age = 25;
bool hasID = true;

if (age >= 21 && hasID)
{
    Console.WriteLine("You can purchase alcohol.");
}

bool isHoliday = false;
bool isVacation = true;

if (isHoliday || isVacation)
{
    Console.WriteLine("No work today!");
}
```

## Loops

Loops allow you to execute a block of code repeatedly.

### For Loop

The `for` loop is ideal when you know exactly how many times you want to execute a block of code:

```csharp
// Print numbers from 1 to 5
for (int i = 1; i <= 5; i++)
{
    Console.WriteLine(i);
}
```

The `for` loop has three parts:
1. Initialization (`int i = 1`): Executed once before the loop starts
2. Condition (`i <= 5`): Checked before each iteration
3. Iteration (`i++`): Executed after each iteration

### While Loop

The `while` loop executes a block of code as long as a specified condition is true:

```csharp
// Print numbers from 1 to 5
int i = 1;
while (i <= 5)
{
    Console.WriteLine(i);
    i++;
}
```

### Do-While Loop

The `do-while` loop is similar to the `while` loop, but it executes the block of code once before checking the condition:

```csharp
// Print numbers from 1 to 5
int i = 1;
do
{
    Console.WriteLine(i);
    i++;
} while (i <= 5);
```

The key difference between `while` and `do-while` is that `do-while` always executes the block at least once, even if the condition is initially false.

### Foreach Loop

The `foreach` loop is used to iterate over elements in a collection (like arrays or lists):

```csharp
string[] fruits = { "Apple", "Banana", "Cherry", "Date", "Elderberry" };

foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);
}
```

### Break and Continue

The `break` statement exits a loop immediately:

```csharp
for (int i = 1; i <= 10; i++)
{
    if (i == 5)
    {
        break; // Exit the loop when i is 5
    }
    Console.WriteLine(i);
}
// Output: 1 2 3 4
```

The `continue` statement skips the current iteration and moves to the next one:

```csharp
for (int i = 1; i <= 10; i++)
{
    if (i % 2 == 0)
    {
        continue; // Skip even numbers
    }
    Console.WriteLine(i);
}
// Output: 1 3 5 7 9
```

## Putting It All Together

Let's create a simple number guessing game that demonstrates conditionals and loops:

```csharp
Random random = new Random();
int secretNumber = random.Next(1, 101); // Random number between 1 and 100
int guess = 0;
int attempts = 0;
bool hasWon = false;

Console.WriteLine("Welcome to the Number Guessing Game!");
Console.WriteLine("I'm thinking of a number between 1 and 100.");

while (!hasWon && attempts < 7)
{
    Console.Write("Enter your guess: ");
    string input = Console.ReadLine();

    if (int.TryParse(input, out guess))
    {
        attempts++;

        if (guess < secretNumber)
        {
            Console.WriteLine("Too low! Try again.");
        }
        else if (guess > secretNumber)
        {
            Console.WriteLine("Too high! Try again.");
        }
        else
        {
            hasWon = true;
            Console.WriteLine($"Congratulations! You guessed the number in {attempts} attempts.");
        }
    }
    else
    {
        Console.WriteLine("Please enter a valid number.");
    }
}

if (!hasWon)
{
    Console.WriteLine($"Sorry, you've used all your attempts. The number was {secretNumber}.");
}
```

## Practice Exercise

Create a console application that:
1. Asks the user to enter a positive integer
2. Prints all the factors of that number
3. Determines if the number is prime (a number is prime if its only factors are 1 and itself)
4. Uses a loop to calculate the factorial of the number (n! = n × (n-1) × (n-2) × ... × 2 × 1)

## Next Steps

In the next chapter, we'll explore working with integers in more depth. You'll learn about different integer types, arithmetic operations, and more advanced mathematical operations.

Before moving on, make sure you're comfortable with:
- Using conditional statements to make decisions in your code
- Implementing different types of loops
- Using logical operators to combine conditions
- Controlling loop execution with break and continue statements