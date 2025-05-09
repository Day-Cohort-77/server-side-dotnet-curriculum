# DateTime Handling

In this chapter, we'll explore how to work with dates and times in C#. The ability to manipulate and format dates and times is essential for many applications, from scheduling systems to data analysis tools.

## Learning Objectives

By the end of this chapter, you should be able to:
- Create and initialize DateTime objects
- Access and modify components of a DateTime
- Perform calculations with dates and times
- Format dates and times for display

## Setting Up

Create a new console application to work with:

```bash
dotnet new console -n DateTimeExplorer
cd DateTimeExplorer
code .
```

## Introduction to DateTime

The `DateTime` struct in C# represents a specific date and time. It provides properties and methods for working with dates and times.

### Creating DateTime Objects

There are several ways to create DateTime objects:

```csharp
// Current date and time
DateTime now = DateTime.Now;
Console.WriteLine($"Current date and time: {now}");

// Current date (time set to 00:00:00)
DateTime today = DateTime.Today;
Console.WriteLine($"Today's date: {today}");

// Create a specific date and time
DateTime specificDate = new DateTime(2025, 5, 1, 14, 30, 0);
Console.WriteLine($"Specific date and time: {specificDate}");

// Create a date (time defaults to 00:00:00)
DateTime dateOnly = new DateTime(2025, 5, 1);
Console.WriteLine($"Date only: {dateOnly}");

// Parse a date string
DateTime parsedDate = DateTime.Parse("2025-05-01 14:30:00");
Console.WriteLine($"Parsed date: {parsedDate}");

// Try to parse a date string
if (DateTime.TryParse("2025-05-01 14:30:00", out DateTime result))
{
    Console.WriteLine($"Successfully parsed: {result}");
}
else
{
    Console.WriteLine("Failed to parse date");
}
```

## Accessing DateTime Components

You can access individual components of a DateTime:

```csharp
DateTime now = DateTime.Now;

Console.WriteLine($"Year: {now.Year}");
Console.WriteLine($"Month: {now.Month}");
Console.WriteLine($"Day: {now.Day}");
Console.WriteLine($"Hour: {now.Hour}");
Console.WriteLine($"Minute: {now.Minute}");
Console.WriteLine($"Second: {now.Second}");
Console.WriteLine($"Millisecond: {now.Millisecond}");

// Get the day of the week
Console.WriteLine($"Day of week: {now.DayOfWeek}");

// Get the day of the year (1-366)
Console.WriteLine($"Day of year: {now.DayOfYear}");

// Check if it's a leap year
bool isLeapYear = DateTime.IsLeapYear(now.Year);
Console.WriteLine($"Is leap year: {isLeapYear}");

// Get the number of days in the month
int daysInMonth = DateTime.DaysInMonth(now.Year, now.Month);
Console.WriteLine($"Days in month: {daysInMonth}");
```

## DateTime Calculations

You can perform various calculations with DateTime objects:

### Adding and Subtracting Time

```csharp
DateTime now = DateTime.Now;
Console.WriteLine($"Current time: {now}");

// Add days
DateTime futureDate = now.AddDays(7);
Console.WriteLine($"7 days from now: {futureDate}");

// Add hours
DateTime laterToday = now.AddHours(3);
Console.WriteLine($"3 hours from now: {laterToday}");

// Add minutes
DateTime soonFromNow = now.AddMinutes(30);
Console.WriteLine($"30 minutes from now: {soonFromNow}");

// Add seconds
DateTime veryClose = now.AddSeconds(45);
Console.WriteLine($"45 seconds from now: {veryClose}");

// Add months
DateTime nextMonth = now.AddMonths(1);
Console.WriteLine($"1 month from now: {nextMonth}");

// Add years
DateTime nextYear = now.AddYears(1);
Console.WriteLine($"1 year from now: {nextYear}");

// Subtract (use negative values)
DateTime lastWeek = now.AddDays(-7);
Console.WriteLine($"7 days ago: {lastWeek}");
```

### Calculating the Difference Between Dates

You can calculate the difference between two dates using the `Subtract` method or by using the `-` operator:

```csharp
DateTime startDate = new DateTime(2025, 1, 1);
DateTime endDate = new DateTime(2025, 12, 31);

// Calculate the difference in days
double daysDifference = (endDate - startDate).TotalDays;
Console.WriteLine($"Days between dates: {daysDifference}");

// Calculate the difference in hours
double hoursDifference = (endDate - startDate).TotalHours;
Console.WriteLine($"Hours between dates: {hoursDifference}");
```

## Comparing DateTime Objects

You can compare DateTime objects using comparison operators:

```csharp
DateTime date1 = new DateTime(2025, 5, 1);
DateTime date2 = new DateTime(2025, 6, 1);

bool isBefore = date1 < date2;
Console.WriteLine($"date1 is before date2: {isBefore}");

bool isAfter = date1 > date2;
Console.WriteLine($"date1 is after date2: {isAfter}");

bool isEqual = date1 == date2;
Console.WriteLine($"date1 is equal to date2: {isEqual}");

// Compare to the current date
bool isPast = date1 < DateTime.Now;
Console.WriteLine($"date1 is in the past: {isPast}");

bool isFuture = date1 > DateTime.Now;
Console.WriteLine($"date1 is in the future: {isFuture}");
```

## Formatting DateTime Objects

You can format DateTime objects in various ways:

```csharp
DateTime now = DateTime.Now;

// Standard date and time formats
Console.WriteLine($"Short date: {now:d}"); // e.g., 5/1/2025
Console.WriteLine($"Long date: {now:D}"); // e.g., Thursday, May 1, 2025
Console.WriteLine($"Short time: {now:t}"); // e.g., 2:30 PM
Console.WriteLine($"Long time: {now:T}"); // e.g., 2:30:45 PM
Console.WriteLine($"Full date/time (short): {now:f}"); // e.g., Thursday, May 1, 2025 2:30 PM
Console.WriteLine($"Full date/time (long): {now:F}"); // e.g., Thursday, May 1, 2025 2:30:45 PM
Console.WriteLine($"General date/time (short): {now:g}"); // e.g., 5/1/2025 2:30 PM
Console.WriteLine($"General date/time (long): {now:G}"); // e.g., 5/1/2025 2:30:45 PM
Console.WriteLine($"Month/day: {now:M}"); // e.g., May 1
Console.WriteLine($"Year/month: {now:Y}"); // e.g., May 2025

// Custom formats
Console.WriteLine($"Custom format: {now:yyyy-MM-dd HH:mm:ss}"); // e.g., 2025-05-01 14:30:45
Console.WriteLine($"Custom format: {now:MM/dd/yyyy hh:mm tt}"); // e.g., 05/01/2025 02:30 PM
```

Here are some common format specifiers:

| Specifier | Description | Example |
|-----------|-------------|---------|
| `d` | Short date pattern | 5/1/2025 |
| `D` | Long date pattern | Thursday, May 1, 2025 |
| `t` | Short time pattern | 2:30 PM |
| `T` | Long time pattern | 2:30:45 PM |
| `f` | Full date/time pattern (short time) | Thursday, May 1, 2025 2:30 PM |
| `F` | Full date/time pattern (long time) | Thursday, May 1, 2025 2:30:45 PM |
| `g` | General date/time pattern (short time) | 5/1/2025 2:30 PM |
| `G` | General date/time pattern (long time) | 5/1/2025 2:30:45 PM |
| `M` or `m` | Month/day pattern | May 1 |
| `Y` or `y` | Year/month pattern | May 2025 |

For custom formats, here are some common format specifiers:

| Specifier | Description | Example |
|-----------|-------------|---------|
| `yyyy` | Four-digit year | 2025 |
| `yy` | Two-digit year | 25 |
| `MM` | Two-digit month | 05 |
| `MMM` | Three-letter month name | May |
| `MMMM` | Full month name | May |
| `dd` | Two-digit day | 01 |
| `d` | Day of the month | 1 |
| `ddd` | Three-letter day name | Thu |
| `dddd` | Full day name | Thursday |
| `HH` | Two-digit hour (24-hour) | 14 |
| `hh` | Two-digit hour (12-hour) | 02 |
| `mm` | Two-digit minute | 30 |
| `ss` | Two-digit second | 45 |
| `tt` | AM/PM designator | PM |

## Putting It All Together

Let's create a simple birthday calculator application:

```csharp
Console.WriteLine("Birthday Calculator");
Console.WriteLine("------------------");

// Get the user's birthday
Console.Write("Enter your birthday (e.g., 1990-01-15): ");
if (DateTime.TryParse(Console.ReadLine(), out DateTime birthday))
{
    // Calculate age
    DateTime today = DateTime.Today;
    int age = today.Year - birthday.Year;

    // Adjust age if birthday hasn't occurred yet this year
    if (birthday.Month > today.Month || (birthday.Month == today.Month && birthday.Day > today.Day))
    {
        age--;
    }

    Console.WriteLine($"You are {age} years old.");

    // Display the day of the week they were born on
    Console.WriteLine($"You were born on a {birthday.DayOfWeek}.");

    // Calculate days until next birthday
    DateTime nextBirthday = new DateTime(today.Year, birthday.Month, birthday.Day);
    if (nextBirthday < today)
    {
        nextBirthday = nextBirthday.AddYears(1);
    }

    int daysUntilBirthday = (nextBirthday - today).Days;
    Console.WriteLine($"Your next birthday is in {daysUntilBirthday} days.");

    // Display if they were born in a leap year
    bool bornInLeapYear = DateTime.IsLeapYear(birthday.Year);
    Console.WriteLine($"Born in a leap year: {bornInLeapYear}");
}
else
{
    Console.WriteLine("Invalid date format. Please try again.");
}
```

## Practice Exercise

No practice exercise for this chapter.

## Next Steps

In the next chapter, we'll explore exception handling in C#. You'll learn how to catch and handle errors gracefully, ensuring your applications can recover from unexpected situations.

Before moving on, make sure you're comfortable with:
- Creating and initializing DateTime objects
- Accessing and modifying components of a DateTime
- Performing calculations with dates and times
- Formatting dates and times for display

[Next Chapter: Exception Handling](./handling-exceptions.md)