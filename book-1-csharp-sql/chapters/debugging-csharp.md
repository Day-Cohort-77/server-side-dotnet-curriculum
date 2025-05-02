# Debugging C# in Visual Studio Code

In this chapter, we'll explore how to debug C# applications in Visual Studio Code. Debugging is an essential skill for any developer, allowing you to identify and fix issues in your code.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the debugging process
- Set up debugging for C# applications in VS Code
- Use breakpoints to pause execution
- Inspect variables and evaluate expressions
- Step through code execution
- Use the Debug Console
- Debug web API applications

## Understanding the Debugging Process

Debugging is the process of finding and resolving bugs or defects in your code. It typically involves:

1. **Reproducing the issue**: Understanding the conditions under which the bug occurs
2. **Isolating the problem**: Narrowing down the code that's causing the issue
3. **Identifying the root cause**: Understanding why the code is behaving unexpectedly
4. **Fixing the issue**: Making the necessary changes to correct the behavior
5. **Verifying the fix**: Ensuring that the issue is resolved and no new issues are introduced

## Setting Up Debugging in VS Code

Visual Studio Code provides excellent debugging support for C# through the C# Dev Kit extension. If you haven't installed it yet, follow these steps:

1. Open Visual Studio Code
2. Click on the Extensions icon in the sidebar (or press Ctrl+Shift+X)
3. Search for "C# Dev Kit"
4. Click "Install"

Once the extension is installed, you can set up debugging for your C# application:

1. Open your C# project in VS Code
2. Click on the Run and Debug icon in the sidebar (or press Ctrl+Shift+D)
3. Click on "create a launch.json file" and select ".NET Core" from the dropdown
4. VS Code will create a `.vscode/launch.json` file with default configurations

The default `launch.json` file should look something like this:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": ".NET Core Launch (console)",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "build",
            "program": "${workspaceFolder}/bin/Debug/net8.0/YourProjectName.dll",
            "args": [],
            "cwd": "${workspaceFolder}",
            "console": "internalConsole",
            "stopAtEntry": false
        },
        {
            "name": ".NET Core Attach",
            "type": "coreclr",
            "request": "attach"
        }
    ]
}
```

You may need to adjust the `program` path to match your project's output path.

## Using Breakpoints

Breakpoints allow you to pause the execution of your program at a specific line of code. To set a breakpoint:

1. Click in the gutter (the space to the left of the line numbers) next to the line where you want to pause execution
2. A red dot will appear, indicating that a breakpoint has been set

You can also set a breakpoint by placing your cursor on a line and pressing F9.

To remove a breakpoint, simply click on the red dot or press F9 again.

## Starting the Debugger

To start debugging your application:

1. Click on the Run and Debug icon in the sidebar
2. Select the appropriate configuration from the dropdown (e.g., ".NET Core Launch (console)")
3. Click the green play button or press F5

Your application will start running, and execution will pause when it reaches a breakpoint.

## Inspecting Variables

When execution is paused at a breakpoint, you can inspect the values of variables:

1. Hover over a variable in the code to see its current value
2. Look at the Variables panel in the Run and Debug view to see all variables in the current scope
3. Expand objects to see their properties

## Evaluating Expressions

You can evaluate expressions while debugging:

1. Select an expression in your code
2. Right-click and select "Evaluate Expression" from the context menu
3. The result will be displayed in a popup

You can also use the Debug Console (accessible from the Debug toolbar) to evaluate expressions. Simply type an expression and press Enter to see its value.

## Stepping Through Code

When execution is paused, you can step through your code using the following controls in the Debug toolbar:

- **Continue (F5)**: Resume execution until the next breakpoint
- **Step Over (F10)**: Execute the current line and move to the next line
- **Step Into (F11)**: If the current line contains a function call, step into that function
- **Step Out (Shift+F11)**: Complete the execution of the current function and return to the calling function
- **Restart (Ctrl+Shift+F5)**: Restart the debugging session
- **Stop (Shift+F5)**: End the debugging session

## Using the Debug Console

The Debug Console allows you to interact with your application while it's being debugged. You can:

- Evaluate expressions
- Call methods
- View the output of `Console.WriteLine()` statements
- See error messages

To open the Debug Console, click on the "Debug Console" tab in the panel at the bottom of the VS Code window.

## Debugging Web API Applications

Debugging web API applications is similar to debugging console applications, but there are a few differences:

1. The application runs as a web server, so you'll need to send HTTP requests to trigger your code
2. You may need to use a tool like Postman or the VS Code REST Client extension to send requests
3. You'll need to set breakpoints in your API controller methods or middleware

To debug a web API application:

1. Set breakpoints in your API controller methods
2. Start the debugger using the appropriate configuration (e.g., ".NET Core Launch (web)")
3. Send HTTP requests to your API endpoints using a tool like Postman
4. When a request hits an endpoint with a breakpoint, execution will pause

## Common Debugging Scenarios

### Debugging Null Reference Exceptions

Null reference exceptions occur when you try to access a member of a null object. To debug these:

1. Set a breakpoint before the line that's throwing the exception
2. Inspect the variables to see which one is null
3. Determine why the variable is null and fix the issue

### Debugging Logic Errors

Logic errors occur when your code doesn't behave as expected, even though it doesn't throw exceptions. To debug these:

1. Set breakpoints at key points in your code
2. Step through the code and inspect variables to see where the behavior diverges from what you expect
3. Use the Debug Console to evaluate expressions and test hypotheses

### Debugging Asynchronous Code

Debugging asynchronous code can be challenging because execution can jump between different parts of your code. To debug asynchronous code:

1. Set breakpoints in both the calling code and the async methods
2. Use "Step Into" to follow the execution flow
3. Pay attention to the call stack to understand where you are in the execution flow

## Debugging Tips and Tricks

- **Conditional Breakpoints**: Right-click on a breakpoint and select "Edit Breakpoint" to set a condition. The breakpoint will only be hit when the condition is true.
- **Logpoints**: Right-click in the gutter and select "Add Logpoint" to log a message to the console without modifying your code.
- **Data Breakpoints**: In some cases, you can set breakpoints that are triggered when a variable's value changes.
- **Exception Breakpoints**: Configure the debugger to break when specific exceptions are thrown.
- **Call Stack**: Use the Call Stack panel to see the sequence of function calls that led to the current point in execution.

## Conclusion

Debugging is an essential skill for any developer. With the tools and techniques covered in this chapter, you should be able to effectively debug your C# applications in Visual Studio Code.

In the next chapter, we'll explore how to create a minimal API project, which will be the foundation for our Harbor Master API.

## Practice Exercise

Enhance your debugging skills by:
1. Creating a simple C# console application with a deliberate bug (e.g., a null reference exception or an off-by-one error in a loop)
2. Setting breakpoints at strategic locations
3. Using the debugger to identify and fix the issue
4. Experimenting with different debugging features like conditional breakpoints and logpoints
5. Debugging an asynchronous method that uses `async` and `await`