# Next Steps

Now that you've completed the projects, you can try one of the following exercises to deepen your learning.

## Option 1: Expand Harbor Master API

- Filtering and sorting for GET endpoints. Here is a well-formed LLM prompt you can use to get started.
    ````
    Here is my C# code for handling GET operations for the `ships` resource in my project

    ```
    paste the code from the endpoint module here
    ```

    I would like to support the following operations for the client

    1. Filter ships by type with a query string parameter: `?type=some_value`
    2. Sort ships by name is ascending or descending order: `?sortby=name&direction=asc`

    Generate the code, with comprehensive explanations, needed in the module
    ````
- Pagination for large result sets. Here is a couple LLM prompts to start you off.
    ````
    Here is my current code for seeding my database with ships.

    ```
    Paste either the SeedDatabaseAsync task here, or the seed-data.sql file, depending on which strategy you following for seeding your database
    ```

    Update this code to seed by database with 25 ships, 10 haulers, and 5 docks.
    ````

    Update your code with what the LLM generates, then follow up with this prompt

    ````
    Here is my code that handles the GET operation for all ships.

    ```
    paste your GetAllShipsAsync task code here
    ```

    I want to only provide 5 ships at a time to the client when all ships are requested. The response should include the following information in the JSON.

    1. A `ships` key with an array of 5 ships
    2. A `previous` key that will contain the URL to get the previous 5 ships when it is not the first 5 ships in the payload
    3. A `next` key that will contain the URL to get the next 5 ships when is it not that last 5 ships in the payload
    ````
- Logging and error handling middleware. Start your journey with the following LLM prompt.
    ````
    I have an existing .NET minimal Web API. Here is my existing Program.cs

    ```cs
    paste your Program.cs content here
    ```

    Here is the logic for handling HTTP operations for my `ships` resource

    ```cs
    paste the contents of ShipEndpoints.cs here
    ```

    I need you to generate instructions and code for implementing a logging middleware solution that will output a well-formed log message to both stdout and a log file that keeps a record of all HTTP requests from the client
    ````

    Once you have something that works, you generate a well-formed LLM prompt that helps you implement an error handling middleware solution for your project. Make sure you provide context in your prompts that helps you understand how to test this solution.

## Option 2: Build a React client application

You've been using your API client _(e.g. Yaak)_ to test your endpoints thus far. A real challenge at this point is to build a React project provides a interface for people to create, read, and delete data.

You can build a React-based client for either Harbor Master or Jewelry Junction.

[Back to project list >](../README.md)
