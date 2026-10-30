# Keyed Dependency Injection in .NET 8 Azure Functions

An Azure Functions v4 demo on .NET 8 that demonstrates **keyed dependency injection** (`AddKeyedScoped` / `[FromKeyedServices]`) in the isolated worker model. Three HTTP endpoints each receive a different implementation of the same `INotification` interface, selected by its registered key — a way to choose between email, SMS, and push strategies without writing a factory class.

[C# and .NET examples](../README.md) · **Dependency injection**

<a id="keyed-services-in-net-8-dependency-injection--azure-functions-example-c"></a>

## Start here

Read the requirements and any configuration below, then follow the usage example. Source paths below are relative to the repository root; run commands from the working directory stated in Usage. This example’s source is in `KeyedService/`. Build the selected project with the SDK and platform declared in its project file.

<a id="-overview"></a>

## Overview

<a id="overview"></a>

.NET 8 introduced first-class keyed service registration in the DI container. This project shows how to register multiple implementations of the same interface under different string keys and inject the correct one into each Azure Function using the `[FromKeyedServices("key")]` attribute.

<a id="guide-features"></a>

<a id="-features"></a>

## Features

### Functions

| Function | HTTP Method | Key | Notification Type |
|---|---|---|---|
| `emailnotificationfunction` | GET | `"email"` | `EmailNotification` |
| `pushnotificationfunction` | GET | `"push"` | `PushNotification` |
| `smsnotificationfunction` | GET | `"sms"` | `SmsNotification` |

Each function calls `INotification.NotifyAsync(message)` on its injected implementation and returns a plain-text response confirming which channel was used.

<a id="guide-quick-start"></a>

<a id="-quick-start"></a>

## Quick start

```bash
cd KeyedService
func start --port 7194
```

<a id="-prerequisites-and-installation"></a>

## Prerequisites and installation

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- [Azure Functions Core Tools v4](https://learn.microsoft.com/azure/azure-functions/functions-run-local)
- A `local.settings.json` file in the project root (not committed to source control)

#### Minimum `local.settings.json`

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
    "APPLICATIONINSIGHTS_CONNECTION_STRING": ""
  }
}
```

### NuGet Packages

| Package | Version |
|---|---|
| `Microsoft.Azure.Functions.Worker` | 1.20.1 |
| `Microsoft.Azure.Functions.Worker.Extensions.Http` | 3.1.0 |
| `Microsoft.Azure.Functions.Worker.Extensions.Http.AspNetCore` | 1.2.0 |
| `Microsoft.Azure.Functions.Worker.Sdk` | 1.16.4 |
| `Microsoft.ApplicationInsights.WorkerService` | 2.21.0 |
| `Microsoft.Azure.Functions.Worker.ApplicationInsights` | 1.1.0 |

<a id="guide-usage"></a>

<a id="-usage"></a>

## Usage

### DI Registration

```csharp
services.AddKeyedScoped<INotification, EmailNotification>("email");
services.AddKeyedScoped<INotification, PushNotification>("push");
services.AddKeyedScoped<INotification, SmsNotification>("sms");
```

### Injection in Functions

```csharp
public class EmailNotificationFunction(
    [FromKeyedServices("email")] INotification notification,
    ILogger<EmailNotificationFunction> logger)
```

<a id="usage"></a>

```bash
# Email notification
curl http://localhost:7194/api/emailnotificationfunction

# Push notification
curl http://localhost:7194/api/pushnotificationfunction

# SMS notification
curl http://localhost:7194/api/smsnotificationfunction
```

Each returns a plain-text confirmation such as `notification sent [EMAIL]`.

The `EmailNotification`, `PushNotification`, and `SmsNotification` implementations only log the message (they await `Task.CompletedTask`); replace them with real email, push, or SMS providers in your own application.

<a id="-architecture"></a>

## Architecture

### Project Structure

```text
KeyedService/
├── Functions/
│   ├── EmailNotificationFunction.cs   # Injects INotification keyed "email"
│   ├── PushNotificationFunction.cs    # Injects INotification keyed "push"
│   └── SmsNotificationFunction.cs     # Injects INotification keyed "sms"
├── Logic/
│   ├── Contract/
│   │   └── INotification.cs           # Interface: NotifyAsync(string message)
│   └── Manager/
│       ├── EmailNotification.cs       # Email implementation
│       ├── PushNotification.cs        # Push implementation
│       └── SmsNotification.cs         # SMS implementation
├── Properties/
│   └── launchSettings.json            # Default port: 7194
├── Program.cs                         # Host builder with keyed DI registration
├── host.json
└── KeyedService.csproj
```

<a id="-development-and-testing"></a>

## Development and testing

Build with `dotnet build KeyedService/KeyedService.csproj` from the repository root. In a configured local host, invoke each of the three GET routes and compare its confirmation with the logged channel. No host or telemetry service was launched in this review.

<a id="guide-troubleshooting-and-limitations"></a>

<a id="-troubleshooting-and-limitations"></a>

## Troubleshooting and limitations

### FAQ

**What are keyed services in .NET?**
Introduced in .NET 8, keyed services let you register several implementations of one interface under different keys (`AddKeyedScoped`, `AddKeyedSingleton`, `AddKeyedTransient`) and request a specific one with `[FromKeyedServices("key")]` or `GetRequiredKeyedService<T>("key")`.

**Do keyed services work in Azure Functions?**
Yes — this project uses them in the .NET 8 isolated worker model with constructor injection into function classes.

The three notification implementations only log messages. No email, SMS or push service is contacted. When using `UseDevelopmentStorage=true`, start Azurite before the Functions host. Example routes are local; the source declares function-level authorization for hosted endpoints.

<a id="-related-documentation"></a>

## Related documentation

### Related Examples

- [MediatR and CQRS in Azure Functions](../Mediat/readme.md)
- [Azure Functions middleware](../Middleware/README.md)
- [Durable Functions orchestration](../Durable/readme.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
