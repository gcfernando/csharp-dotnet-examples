[C# and .NET examples](../README.md) · **Async programming**

<a id="taskwhenall-exception-handling-in-c--capturing-all-exceptions-net-8"></a>

# Handle Every Task.WhenAll Exception in C#

A .NET 8 console application demonstrating how to **catch every exception from `Task.WhenAll` in C#**, not just the first one, when several concurrent tasks fail.

> **In this guide:** [Get started](#guide-quick-start) · [Walk through the example](#guide-usage) · [Understand the details](#guide-architecture) · [Check the limits](#guide-troubleshooting-and-limitations)

<a id="guide-quick-start"></a>

## 🚀 Quick start

### Build and Run

```bash
cd TaskErrors
dotnet run
```

## 📦 Prerequisites and installation

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download)

<a id="guide-usage"></a>

## 💡 Usage

### The Problem

When `Task.WhenAll` is awaited and multiple tasks throw, one exception is rethrown into the catch block. Other failures remain on the task objects and are missed if you inspect only the caught `Exception`.

### The Solution

After catching, iterate over each task in your list and collect their `Exception.InnerExceptions`:

```csharp
catch (Exception ex)
{
    var exceptions = tasks.SelectMany(
        task => task.Exception?.InnerExceptions ?? Enumerable.Empty<Exception>());

    foreach (var exception in exceptions)
    {
        Console.WriteLine(exception.Message);
    }
}
```

### Demo

The application runs two async methods concurrently:

| Method | Delay | Exception Message |
|---|---|---|
| `MethodOneAsync` | 1 second | `"Custom Error Method One"` |
| `MethodTwoAsync` | 2 seconds | `"Custom Error Method Two"` |

Both tasks are stored in a `List<Task>` before `Task.WhenAll` is awaited. Inside the catch block, the code collects inner exceptions from all tasks and prints both error messages.

### Expected Output

```text
Custom Error Method One
Custom Error Method Two
```

<a id="guide-architecture"></a>

## 🏗️ Architecture

### Project Structure

```text
TaskErrors/
├── Program.cs          # Two throwing async methods + WhenAll error capture pattern
├── TaskErrors.csproj
└── TaskErrors.sln
```

The expected output follows from the two exception messages in `Program.cs`; execution was not repeated in this review.

## 🧪 Development and testing

From the repository root, `dotnet run --project TaskErrors/TaskErrors.csproj` should print the two messages shown above. Both task definitions and the catch block were inspected; this documentation review did not re-run the application.

<a id="guide-troubleshooting-and-limitations"></a>

## 🛠️ Troubleshooting and limitations

### FAQ

**Why does `await Task.WhenAll(...)` only throw one exception?**
`Task.WhenAll` returns a task whose `Exception` is an `AggregateException` containing every failure, but `await` unwraps it and rethrows one of the inner exceptions. Keeping a reference to the tasks (or to the `WhenAll` task) lets you read all of them afterwards.

**Is there a shorter alternative?**
You can store the `WhenAll` task in a variable, await it inside `try`, and in the `catch` read `whenAllTask.Exception.InnerExceptions` — it contains the same exceptions as collecting them from each task.

Awaiting WhenAll throws one of the failures; other failures remain stored on the task objects. They are not discarded. This example enumerates each faulted task after awaiting completion; cancellation and nested aggregate handling are not demonstrated.

## 🔗 Related documentation

### Related Examples

- [Unit testing ILogger with FakeLogger](../Fake/readme.md)
- [SemaphoreSlim concurrency example](../SemaphoreSlim/readme.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
