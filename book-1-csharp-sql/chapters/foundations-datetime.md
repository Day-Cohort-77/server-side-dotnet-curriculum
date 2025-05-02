# DateTime Handling

In this chapter, we'll explore how to work with dates and times in C#. The ability to manipulate and format dates and times is essential for many applications, from scheduling systems to data analysis tools.

## Learning Objectives

By the end of this chapter, you should be able to:
- Create and initialize DateTime objects
- Access and modify components of a DateTime
- Perform calculations with dates and times
- Format dates and times for display
- Work with time spans
- Handle time zones and UTC

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

// Current UTC date and time
DateTime utcNow = DateTime.UtcNow;
Console.WriteLine($"Current UTC date and time: {utcNow}");

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

You can calculate the difference between two dates using the `Subtract` method or by using the `-` operator. The result is a `TimeSpan` object:

```csharp
DateTime startDate = new DateTime(2025, 1, 1);
DateTime endDate = new DateTime(2025, 12, 31);

// Calculate the difference
TimeSpan duration = endDate - startDate; // or endDate.Subtract(startDate)

Console.WriteLine($"Total days: {duration.TotalDays}");
Console.WriteLine($"Total hours: {duration.TotalHours}");
Console.WriteLine($"Total minutes: {duration.TotalMinutes}");
Console.WriteLine($"Total seconds: {duration.TotalSeconds}");

// Access components of the TimeSpan
Console.WriteLine($"Days: {duration.Days}");
Console.WriteLine($"Hours: {duration.Hours}");
Console.WriteLine($"Minutes: {duration.Minutes}");
Console.WriteLine($"Seconds: {duration.Seconds}");
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

## Working with TimeSpan

The `TimeSpan` struct represents a time interval (duration). You can create TimeSpan objects in several ways:

```csharp
// Create a TimeSpan using constructor
TimeSpan duration1 = new TimeSpan(2, 30, 0); // 2 hours and 30 minutes
Console.WriteLine($"Duration 1: {duration1}");

// Create a TimeSpan using static methods
TimeSpan duration2 = TimeSpan.FromHours(2.5); // 2.5 hours
Console.WriteLine($"Duration 2: {duration2}");

TimeSpan duration3 = TimeSpan.FromMinutes(150); // 150 minutes
Console.WriteLine($"Duration 3: {duration3}");

TimeSpan duration4 = TimeSpan.FromDays(1.5); // 1.5 days
Console.WriteLine($"Duration 4: {duration4}");

// Create a TimeSpan by subtracting DateTime objects
DateTime start = DateTime.Now;
DateTime end = start.AddHours(3);
TimeSpan duration5 = end - start;
Console.WriteLine($"Duration 5: {duration5}");

// Access TimeSpan properties
Console.WriteLine($"Total days: {duration4.TotalDays}");
Console.WriteLine($"Total hours: {duration4.TotalHours}");
Console.WriteLine($"Days component: {duration4.Days}");
Console.WriteLine($"Hours component: {duration4.Hours}");
```

You can perform arithmetic operations with TimeSpan objects:

```csharp
TimeSpan time1 = TimeSpan.FromHours(2);
TimeSpan time2 = TimeSpan.FromHours(3);

// Addition
TimeSpan sum = time1 + time2;
Console.WriteLine($"Sum: {sum}");

// Subtraction
TimeSpan difference = time2 - time1;
Console.WriteLine($"Difference: {difference}");

// Multiplication
TimeSpan doubled = time1 * 2;
Console.WriteLine($"Doubled: {doubled}");

// Division
TimeSpan halved = time1 / 2;
Console.WriteLine($"Halved: {halved}");
```

## Working with Time Zones

C# provides the `TimeZoneInfo` class for working with time zones:

```csharp
// Get the local time zone
TimeZoneInfo localZone = TimeZoneInfo.Local;
Console.WriteLine($"Local time zone: {localZone.DisplayName}");

// Get the UTC time zone
TimeZoneInfo utcZone = TimeZoneInfo.Utc;
Console.WriteLine($"UTC time zone: {utcZone.DisplayName}");

// Get all time zones
Console.WriteLine("\nAll time zones:");
foreach (TimeZoneInfo zone in TimeZoneInfo.GetSystemTimeZones())
{
    Console.WriteLine(zone.Id);
}

// Convert between time zones
DateTime utcTime = DateTime.UtcNow;
Console.WriteLine($"\nUTC time: {utcTime}");

// Convert UTC to local time
DateTime localTime = TimeZoneInfo.ConvertTimeFromUtc(utcTime, localZone);
Console.WriteLine($"Local time: {localTime}");

// Convert to a specific time zone (e.g., Pacific Standard Time)
try
{
    TimeZoneInfo pacificZone = TimeZoneInfo.FindSystemTimeZoneById("Pacific Standard Time");
    DateTime pacificTime = TimeZoneInfo.ConvertTimeFromUtc(utcTime, pacificZone);
    Console.WriteLine($"Pacific time: {pacificTime}");
}
catch (TimeZoneNotFoundException)
{
    Console.WriteLine("Pacific Standard Time zone not found");
}
```

Note that time zone IDs may vary between operating systems. On Windows, you might use "Pacific Standard Time", while on Linux or macOS, you might use "America/Los_Angeles".

## DateOnly and TimeOnly (C# 10 and later)

In C# 10 and later, there are two new types: `DateOnly` and `TimeOnly`. These types are useful when you only need to work with dates or times, without the other component:

```csharp
// DateOnly represents just a date without a time component
DateOnly date = new DateOnly(2025, 5, 1);
Console.WriteLine($"Date only: {date}");

// TimeOnly represents just a time without a date component
TimeOnly time = new TimeOnly(14, 30, 0);
Console.WriteLine($"Time only: {time}");

// You can convert between DateTime and DateOnly/TimeOnly
DateTime dateTime = new DateTime(2025, 5, 1, 14, 30, 0);
DateOnly dateFromDateTime = DateOnly.FromDateTime(dateTime);
TimeOnly timeFromDateTime = TimeOnly.FromDateTime(dateTime);

Console.WriteLine($"Date from DateTime: {dateFromDateTime}");
Console.WriteLine($"Time from DateTime: {timeFromDateTime}");

// You can also combine DateOnly and TimeOnly to create a DateTime
DateTime combined = dateFromDateTime.ToDateTime(timeFromDateTime);
Console.WriteLine($"Combined DateTime: {combined}");
```

## Putting It All Together

Let's create a simple appointment scheduler application that demonstrates working with dates and times:

```csharp
List<(string Title, DateTime Start, DateTime End)> appointments = new List<(string, DateTime, DateTime)>();
bool running = true;

Console.WriteLine("Appointment Scheduler");

while (running)
{
    Console.WriteLine("\nOptions:");
    Console.WriteLine("1. Add an appointment");
    Console.WriteLine("2. View all appointments");
    Console.WriteLine("3. View appointments for a specific day");
    Console.WriteLine("4. Calculate appointment duration");
    Console.WriteLine("5. Exit");
    Console.Write("Enter your choice (1-5): ");

    if (int.TryParse(Console.ReadLine(), out int choice))
    {
        switch (choice)
        {
            case 1: // Add an appointment
                Console.Write("Enter appointment title: ");
                string title = Console.ReadLine();

                Console.Write("Enter start date and time (e.g., 2025-05-01 14:30): ");
                if (DateTime.TryParse(Console.ReadLine(), out DateTime startTime))
                {
                    Console.Write("Enter end date and time (e.g., 2025-05-01 15:30): ");
                    if (DateTime.TryParse(Console.ReadLine(), out DateTime endTime))
                    {
                        if (endTime > startTime)
                        {
                            appointments.Add((title, startTime, endTime));
                            Console.WriteLine("Appointment added successfully!");
                        }
                        else
                        {
                            Console.WriteLine("End time must be after start time.");
                        }
                    }
                    else
                    {
                        Console.WriteLine("Invalid end date and time format.");
                    }
                }
                else
                {
                    Console.WriteLine("Invalid start date and time format.");
                }
                break;

            case 2: // View all appointments
                if (appointments.Count == 0)
                {
                    Console.WriteLine("No appointments scheduled.");
                }
                else
                {
                    Console.WriteLine("\nAll Appointments:");
                    for (int i = 0; i < appointments.Count; i++)
                    {
                        var (appTitle, appStart, appEnd) = appointments[i];
                        TimeSpan duration = appEnd - appStart;
                        Console.WriteLine($"{i + 1}. {appTitle}");
                        Console.WriteLine($"   Start: {appStart:f}");
                        Console.WriteLine($"   End: {appEnd:f}");
                        Console.WriteLine($"   Duration: {duration.Hours}h {duration.Minutes}m");
                        Console.WriteLine();
                    }
                }
                break;

            case 3: // View appointments for a specific day
                Console.Write("Enter date (e.g., 2025-05-01): ");
                if (DateTime.TryParse(Console.ReadLine(), out DateTime date))
                {
                    var dayAppointments = appointments.Where(a =>
                        a.Start.Year == date.Year &&
                        a.Start.Month == date.Month &&
                        a.Start.Day == date.Day).ToList();

                    if (dayAppointments.Count == 0)
                    {
                        Console.WriteLine($"No appointments scheduled for {date:D}.");
                    }
                    else
                    {
                        Console.WriteLine($"\nAppointments for {date:D}:");
                        foreach (var (appTitle, appStart, appEnd) in dayAppointments)
                        {
                            TimeSpan duration = appEnd - appStart;
                            Console.WriteLine($"- {appTitle}");
                            Console.WriteLine($"  Start: {appStart:t}");
                            Console.WriteLine($"  End: {appEnd:t}");
                            Console.WriteLine($"  Duration: {duration.Hours}h {duration.Minutes}m");
                            Console.WriteLine();
                        }
                    }
                }
                else
                {
                    Console.WriteLine("Invalid date format.");
                }
                break;

            case 4: // Calculate appointment duration
                if (appointments.Count == 0)
                {
                    Console.WriteLine("No appointments to calculate duration for.");
                }
                else
                {
                    Console.WriteLine("\nSelect an appointment:");
                    for (int i = 0; i < appointments.Count; i++)
                    {
                        Console.WriteLine($"{i + 1}. {appointments[i].Title} ({appointments[i].Start:g})");
                    }

                    Console.Write("Enter appointment number: ");
                    if (int.TryParse(Console.ReadLine(), out int appNumber) &&
                        appNumber >= 1 && appNumber <= appointments.Count)
                    {
                        var appointment = appointments[appNumber - 1];
                        TimeSpan duration = appointment.End - appointment.Start;

                        Console.WriteLine($"\nDuration of '{appointment.Title}':");
                        Console.WriteLine($"Total days: {duration.TotalDays:F2}");
                        Console.WriteLine($"Total hours: {duration.TotalHours:F2}");
                        Console.WriteLine($"Total minutes: {duration.TotalMinutes:F2}");
                        Console.WriteLine($"Total seconds: {duration.TotalSeconds:F2}");

                        Console.WriteLine($"\nComponents:");
                        Console.WriteLine($"Days: {duration.Days}");
                        Console.WriteLine($"Hours: {duration.Hours}");
                        Console.WriteLine($"Minutes: {duration.Minutes}");
                        Console.WriteLine($"Seconds: {duration.Seconds}");
                    }
                    else
                    {
                        Console.WriteLine("Invalid appointment number.");
                    }
                }
                break;

            case 5: // Exit
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
        Console.WriteLine("Invalid choice. Please enter a number between 1 and 5.");
    }
}
```

## Practice Exercise

Create a console application that:
1. Asks the user to enter their date of birth
2. Calculates and displays their current age in years, months, and days
3. Calculates and displays the number of days until their next birthday
4. Displays the day of the week they were born on
5. Shows how many leap years they have lived through
6. Creates a timeline of significant dates in their life (e.g., 18th birthday, 21st birthday, etc.)

## Next Steps

In the next chapter, we'll explore exception handling in C#. You'll learn how to catch and handle errors gracefully, ensuring your applications can recover from unexpected situations.

Before moving on, make sure you're comfortable with:
- Creating and initializing DateTime objects
- Accessing and modifying components of a DateTime
- Performing calculations with dates and times
- Formatting dates and times for display
- Working with time spans
- Handling time zones and UTC