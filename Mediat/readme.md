# MediatR and CQRS with Azure Functions in C#

An Azure Functions v4 solution on .NET 8 that demonstrates **MediatR** and **CQRS** (Command Query Responsibility Segregation) with a layered architecture and **MediatR pipeline behaviours** for logging and performance monitoring. The solution implements user creation and retrieval through CQRS commands and queries, wired with two MediatR pipeline behaviours.

[C# and .NET examples](../README.md) · **Application architecture**

<a id="mediatr-cqrs-example-with-azure-functions-in-c-net-8-clean-architecture"></a>

<a id="mediatr-cqrs-example-with-azure-functions-in-c-net-8-layered-architecture"></a>

## Start here

Read the requirements and any configuration below, then follow the usage example. Source paths below are relative to the repository root; run commands from the working directory stated in Usage. This example’s source is in `Mediat/`. Build the selected project with the SDK and platform declared in its project file.

<a id="guide-features"></a>

<a id="-features"></a>

## Features

### Projects

| Project | Description |
|---|---|
| `Mediat.AzureFunction` | Azure Functions host — HTTP triggers for create and get user |
| `Mediat.Business` | `CreateUserCommand` + handler, `GetUserQuery` + handler |
| `Mediat.Data` | `IUserRepository` interface and in-memory `UserRepository` |
| `Mediat.Infrastructure` | `LoggingBehavior` and `PerformanceMonitoringBehavior` |
| `Mediat.Model` | `User` entity (`Id`, `Name`, `Email`) |

<a id="guide-quick-start"></a>

<a id="-quick-start"></a>

## Quick start

```bash
cd Mediat/Mediat.AzureFunction
func start --port 7094
```

<a id="-prerequisites-and-installation"></a>

## Prerequisites and installation

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- [Azure Functions Core Tools v4](https://learn.microsoft.com/azure/azure-functions/functions-run-local)
- A `local.settings.json` file in `Mediat.AzureFunction` (not committed to source control)

#### Minimum `local.settings.json`

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated"
  }
}
```

### NuGet Packages (Mediat.AzureFunction)

| Package | Version |
|---|---|
| `MediatR.Extensions.Microsoft.DependencyInjection` | 11.0.0 |
| `Microsoft.Azure.Functions.Worker` | 2.0.0 |
| `Microsoft.Azure.Functions.Worker.Extensions.Http` | 3.3.0 |
| `Microsoft.Azure.Functions.Worker.Extensions.Http.AspNetCore` | 2.0.1 |
| `Microsoft.Azure.Functions.Worker.Sdk` | 2.0.2 |
| `Newtonsoft.Json` | 13.0.3 |

<a id="guide-usage"></a>

<a id="-usage"></a>

## Usage

### HTTP Endpoints

| Function | Method | Route | Description |
|---|---|---|---|
| `create-user-function` | POST | `/api/user` | Creates a new user from JSON body |
| `get-user-function` | GET | `/api/user/{id}` | Retrieves a user by integer ID |

#### Request Body (POST)

```json
{
  "name": "Alice",
  "email": "alice@example.com"
}
```

#### Response (POST / GET)

```json
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com"
}
```

### Pipeline Behaviours

| Behaviour | Description |
|---|---|
| `LoggingBehavior<TRequest, TResponse>` | Logs `Handling {request type}` before and `Handled {response type}` after each handler call |
| `PerformanceMonitoringBehavior<TRequest, TResponse>` | Logs a warning after a successful downstream return when elapsed time exceeds 500 ms; thrown failures bypass that warning |

### Data Storage

`UserRepository` uses a static in-memory `List<User>`. IDs are assigned sequentially (count + 1). Data is not persisted between application restarts.

<a id="usage"></a>

```bash
# Create a user
curl -X POST http://localhost:7094/api/user \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'

# Retrieve a user
curl http://localhost:7094/api/user/1
```

`GET /api/user/{id}` returns `404 Not Found` when no user has that ID, and selected null-request/null-command paths return `400 Bad Request`; malformed JSON and invalid field values are not comprehensively handled.

<a id="-architecture"></a>

## Architecture

<a id="architecture"></a>

```text
HTTP Request
     │
     ▼
Mediat.AzureFunction     ← Azure Functions host (HTTP triggers)
     │  sends IRequest via IMediator
     ▼
Mediat.Business          ← Command/Query handlers
     │  calls repository
     ▼
Mediat.Data              ← In-memory repository implementation
     │  uses model
     ▼
Mediat.Model             ← User entity
```

Pipeline behaviours (Mediat.Infrastructure) wrap every handler call:

```text
LoggingBehavior → PerformanceMonitoringBehavior → Handler
```

### Project Structure

```text
Mediat/
├── Mediat.AzureFunction/
│   ├── CreateUserFunction.cs          # POST /api/user
│   ├── GetUserFunction.cs             # GET /api/user/{id}
│   └── Program.cs                     # MediatR and DI registration
├── Mediat.Business/
│   ├── Commands/UserCommand/
│   │   ├── CreateUserCommand.cs
│   │   └── CreateUserCommandHandler.cs
│   └── Queries/UserQuery/
│       ├── GetUserQuery.cs
│       └── GetUserQueryHandler.cs
├── Mediat.Data/
│   ├── Contract/IUserRepository.cs
│   └── Repository/UserRepository.cs   # In-memory implementation
├── Mediat.Infrastructure/
│   ├── LoggingBehavior.cs
│   └── PerformanceMonitoringBehavior.cs
├── Mediat.Model/
│   └── User.cs                        # Id, Name, Email
└── Mediat.sln
```

<a id="-development-and-testing"></a>

## Development and testing

Check restore/build compatibility for `Mediat/Mediat.sln` with its `x64` solution platform, then test create, get, missing ID and restart behavior in a local Functions host. The Release solution configuration lacks a `Build.0` entry for `Mediat.Infrastructure`; inspect Configuration Manager and project-reference build behavior before treating a solution build as evidence for every project. Package references and handlers were inspected; restore, host execution and concurrent requests were not tested here.

<a id="guide-troubleshooting-and-limitations"></a>

<a id="-troubleshooting-and-limitations"></a>

## Troubleshooting and limitations

### Notes

- `Mediat.AzureFunction` references `MediatR.Extensions.Microsoft.DependencyInjection` 11.0.0, while `Mediat.Business` and `Mediat.Infrastructure` reference `MediatR` 12.4.1, which includes the `AddMediatR` registration itself. The separate DI extension package is not needed with MediatR 12 and later.
- Pipeline behaviours run in registration order, so `LoggingBehavior` wraps `PerformanceMonitoringBehavior`, which wraps the handler.

The static `List<User>` and count-based ID allocation are unsynchronized and lose data on restart. Neither name nor email is validated. A null deserialization result gets 400, but malformed JSON exceptions are not caught and converted to 400. The host references MediatR DI extensions 11.0.0 while business/infrastructure use MediatR 12.4.1; package compatibility and restore require separate verification. Start Azurite for the development storage setting.

<a id="-related-documentation"></a>

## Related documentation

### Related Examples

- [Keyed dependency injection in Azure Functions](../KeyedService/readme.md)
- [Azure Functions middleware](../Middleware/README.md)
- [Durable Functions orchestration](../Durable/readme.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
