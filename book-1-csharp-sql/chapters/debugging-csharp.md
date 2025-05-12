# Debugging C# in Visual Studio Code

In this chapter, we'll explore how to debug C# applications in Visual Studio Code. Debugging is an essential skill for any developer, allowing you to identify and fix issues in your code.

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
4. VS Code will create a `.vscode/launch.json` file with default configurations and open the contents
5. Replace the default content with the following JSON. Note that the `program` key has a path to a file named `HarborMaster.dll`. The name of this file will change for every project — all other values will remain the same.

   ```json
   {
       // Use IntelliSense to learn about possible attributes.
       // Hover to view descriptions of existing attributes.
       // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
       "version": "0.2.0",
       "configurations": [
           {
               "name": ".NET Core Launch (web)",
               "type": "coreclr",
               "request": "launch",
               "preLaunchTask": "build",
               "program": "${workspaceFolder}/bin/Debug/net8.0/HarborMaster.dll",
               "args": [],
               "cwd": "${workspaceFolder}",
               "stopAtEntry": false,
               "env": {
                   "ASPNETCORE_ENVIRONMENT": "Development"
               },
               "sourceFileMap": {
                   "/Views": "${workspaceFolder}/Views"
               }
           }

       ]
   }
   ```
6. In the `.vscode` directory, create a new file named `tasks.json` and place the following JSON in it:
    ```json
    {
       "version": "2.0.0",
       "tasks": [
           {
               "label": "build",
               "command": "dotnet",
               "type": "process",
               "args": [
                   "build",
                   "${workspaceFolder}/HarborMaster.csproj",
                   "/property:GenerateFullPaths=true",
                   "/consoleloggerparameters:NoSummary"
               ],
               "problemMatcher": "$msCompile"
           }
       ]
   }
   ```

Again, notice the `"${workspaceFolder}/HarborMaster.csproj"` arg. The name of that file will change for every project.

## Starting the Debugger

To start debugging your application:

1. Click on the Run and Debug icon in the sidebar
2. Select the appropriate configuration from the dropdown _(e.g., ".NET Core Launch (web)")_
3. Click the green play button

Your application will start running, and you should see output like this in the **Debug Console** panel at the bottom of VS Code.

```sh
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:7152
Microsoft.Hosting.Lifetime: Information: Now listening on: https://localhost:7152
Microsoft.Hosting.Lifetime: Information: Now listening on: http://localhost:5285
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5285
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
Microsoft.Hosting.Lifetime: Information: Application started. Press Ctrl+C to shut down.
Microsoft.Hosting.Lifetime: Information: Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
```



[Continue to the next chapter: Seeding the Database](./harbor-master-seeding.md)