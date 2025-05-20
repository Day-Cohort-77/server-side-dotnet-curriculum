# Introduction to Integration Testing

## Learning Objectives

- Understand what integration testing is and why it's important
- Learn the difference between unit tests and integration tests
- Understand how to test HTTP APIs
- Preview the simplified test project we'll be building

## Introduction

Welcome to the Testy Tester project! In this series of chapters, you'll learn how to write integration tests for a minimal API project that uses Entity Framework Core. We'll focus on the most essential testing concepts to give you a solid foundation.

### What is Integration Testing?

Integration testing is a type of software testing where individual components are combined and tested as a group. The purpose is to verify that different parts of your application work together correctly.

Unlike unit tests (which test isolated components), integration tests examine how components interact with each other in a more realistic environment.

### Why Test APIs?

When building web APIs, it's crucial to ensure that:

1. Endpoints return the expected status codes
2. Response data matches the expected format
3. Basic database operations work correctly

Integration tests help verify these aspects by making actual HTTP requests to your API and checking the responses.

## Our Project: Student Exam Tracker

We'll build a simple API for tracking student exam results, along with focused tests for the most critical functionality. The application will have three models:

1. **Student** - Basic information about students
2. **Exam** - Information about exams
3. **StudentExam** - Records linking students to exams

This project is intentionally small to help you focus on the core testing concepts without getting overwhelmed.

## Testing Approach

Our focused testing approach will include:

1. Setting up a test project that can make HTTP requests to our API
2. Creating simple test data to work with
3. Writing tests for the most critical endpoints (GET and POST)
4. Making basic assertions about responses

## Benefits of Integration Testing

By the end of this series, you'll understand how integration tests can:

- Catch bugs that unit tests might miss
- Provide confidence that your API works as expected
- Serve as documentation for how your API should behave

## Next Steps

In the next chapter, we'll set up our test project and configure it to work with our API. We'll use a popular testing framework to make requests to our API endpoints.

Let's get started!

[Next: Setting Up the Test Project](./testy-setup.md)