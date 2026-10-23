# Azure Durable Functions Hello World in C# — DemoOne (.NET 8 Isolated Worker)

A minimal **Azure Durable Functions** v4 example in C# that runs a single-activity orchestration pipeline on the .NET 8 isolated worker. It is a good first step for learning how an HTTP starter (`DurableTaskClient`), an orchestrator (`TaskOrchestrationContext`), and an activity function work together. An HTTP trigger receives a JSON body, starts an orchestrator, and the orchestrator calls one activity that returns a personalised greeting.

## ✨ Features

### Functions

| Function | Trigger | Description |
|---|---|---|
| `func-message-validator` | HTTP POST | Reads request body, starts orchestration, waits for completion, returns result |
| `func-message-orchestrator` | Orchestration | Extracts `name` from JSON payload, calls `func-say-hello` |
| `func-say-hello` | Activity | Returns `"Hello {name}!"` |

## 🚀 Quick start

### Running Locally

```bash
cd Durable/DemoOne
func start --port 7239
```

## 📦 Prerequisites and installation

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
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated"
  }
}
```

> Requires [Azurite](https://learn.microsoft.com/azure/storage/common/storage-use-azurite) or a real Azure Storage connection string for durable task state.

### NuGet Packages

| Package | Version |
|---|---|
| `Microsoft.Azure.Functions.Worker` | 2.0.0 |
| `Microsoft.Azure.Functions.Worker.Extensions.DurableTask` | 1.2.3 |
| `Microsoft.Azure.Functions.Worker.Extensions.Http` | 3.3.0 |
| `Microsoft.Azure.Functions.Worker.Extensions.Http.AspNetCore` | 2.0.1 |
| `Microsoft.Azure.Functions.Worker.Sdk` | 2.0.2 |

## 💡 Usage

<a id="usage"></a>

```bash
curl -X POST http://localhost:7239/api/func-message-validator \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice"}'
```

**Expected greeting value (HTTP serialization is host-dependent and not runtime-verified here):**

```text
"Hello Alice!"
```

**Empty body returns 400:**

```text
Request body cannot be empty.
```

## 🏗️ Architecture

<a id="architecture"></a>

```text
POST /api/func-message-validator
          │
          ▼
 func-message-validator        (HTTP client trigger)
          │  schedules orchestration
          ▼
 func-message-orchestrator     (orchestrator)
          │  calls activity
          ▼
 func-say-hello                (activity — returns "Hello {name}!")
```

### Project Structure

```text
DemoOne/
├── Functions/
│   ├── FunctionClient.cs          # HTTP trigger — starts orchestration
│   ├── FunctionOrchestrator.cs    # Orchestrator — extracts name, calls activity
│   └── Triggers/
│       └── ModifyMessage.cs       # Activity (func-say-hello)
├── Properties/
│   └── launchSettings.json        # Default port: 7239
├── Program.cs                     # Host builder entry point
├── host.json
└── DemoOne.csproj
```

## 🧪 Development and testing

From the repository root, build with `dotnet build Durable/DemoOne/DemoOne.csproj -p:Platform=x64`. The project declares only the `x64` platform; its solution's Any CPU configurations map to x64 but omit the corresponding `Build.0` entries, so select x64 or build the project explicitly when verifying it. In a configured local host, verify the greeting value, exact HTTP encoding and empty-body 400 response. Source and paths were checked; host execution was not performed.

## 🛠️ Troubleshooting and limitations

### Notes

- The HTTP starter waits for the orchestration to finish (`WaitForInstanceCompletionAsync`) and returns the orchestration's serialized output, so the value passed to OkObjectResult is already JSON-serialized text. The HTTP formatter can add further encoding; inspect a live response before relying on its exact wire format.
- Waiting within the HTTP request keeps the demo simple; the wait itself is asynchronous. For long-running workflows, return the orchestration status URLs instead (for example with `CreateCheckStatusResponseAsync`) and let the client poll.

Only an empty request body is rejected explicitly. Malformed JSON and a missing `name` are not validated as a complete API contract. The starter waits without an explicit timeout or cancellation token and does not inspect the orchestration failure status. The exact HTTP serialization has not been checked against a running host.

## 🔗 Related documentation

### Related Examples

- [DemoTwo — three chained activities with input validation](../DemoTwo/readme.md)
- [Durable Functions overview](../readme.md)
- [Back to all C# and .NET code examples](../../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp_codes/documentation/) · [Project overview](https://gcfernando.github.io/csharp_codes/) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
