# Setting Up a Console Application

In this chapter, you'll learn how to create and set up a C# console application using the .NET CLI (Command Line Interface). This will be the foundation for our C# Quick Intro track.

## Learning Objectives

By the end of this chapter, you should be able to:
- Use the .NET CLI to create a new console application
- Understand the basic structure of a C# console application
- Run a C# console application from the command line
- Modify the default program to display custom messages

## Creating a New Console Application

The .NET CLI provides a simple way to create a new console application. Follow these steps:

1. Open a terminal or command prompt
2. Navigate to the directory where you want to create your project
3. Run the following command:

```bash
dotnet new console -n HelloWorld
```

This command creates a new console application named "HelloWorld". The `-n` flag specifies the name of the project.

4. Navigate into the newly created project directory:

```bash
cd HelloWorld
```

## Understanding the Project Structure

Let's examine the files that were created:

- **HelloWorld.csproj**: This is the project file that contains configuration information for your application.
- **Program.cs**: This is the main source code file that contains the entry point for your application.

Open the `Program.cs` file in Visual Studio Code:

```bash
code .
```

You should see content similar to this:

```csharp
// Program.cs
Console.WriteLine("Hello, World!");
```

This is a very simple program that outputs "Hello, World!" to the console.

## Running Your Console Application

To run your console application, use the following command from within your project directory:

```bash
dotnet run
```

You should see the output:

```
Hello, World!
```

Congratulations! You've just created and run your first C# console application.

## Modifying the Program

Let's modify the program to display a custom message. Open the `Program.cs` file in your editor and change it to:

```csharp
// See https://aka.ms/new-console-template for more information
Console.WriteLine("Hello, C# Learner!");
Console.WriteLine("Welcome to the C# Quick Intro track!");
Console.WriteLine("Today's date is: " + DateTime.Now.ToShortDateString());
```

Save the file and run the application again:

```bash
dotnet run
```

You should now see your custom messages and the current date.

## Understanding the Entry Point

In newer versions of .NET (6.0 and later), the `Program.cs` file uses a simplified syntax called "top-level statements." This allows you to write code directly without having to define a class and a `Main` method.

In previous versions of .NET, the `Program.cs` file would look like this:

```csharp
using System;

namespace HelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");
        }
    }
}
```

The newer syntax is more concise, but it's helpful to understand the traditional structure as you might encounter it in existing codebases.

## Adding User Input

Let's enhance our program to accept user input. Modify your `Program.cs` file:

```csharp
Console.WriteLine("Hello, C# Learner!");
Console.WriteLine("What is your name?");

string name = Console.ReadLine();

Console.WriteLine($"Nice to meet you, {name}!");
Console.WriteLine($"Today's date is: {DateTime.Now.ToShortDateString()}");
```

The `Console.ReadLine()` method reads a line of text from the console. We store this input in a variable called `name` and then use it in our greeting.

Notice the use of string interpolation with the `$` symbol, which allows us to embed variables directly within string literals.

## Practice Exercise

Modify your program to:
1. Ask the user for their favorite color
2. Display a message that includes their name and favorite color
3. Add the current time (not just the date) to the output

## Next Steps

In the next chapter, we'll explore strings and console interaction in more depth. You'll learn about different ways to format strings, how to work with string methods, and more advanced console input/output techniques.

Before moving on, make sure you're comfortable with:
- Creating a new console application
- Running the application
- Modifying the code to display custom messages
- Getting input from the user

If you have any questions or issues, review this chapter or ask your instructor for help.