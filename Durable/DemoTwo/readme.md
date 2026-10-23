# Azure Durable Functions Function Chaining Example in C# — DemoTwo (.NET 8)

An **Azure Durable Functions** v4 example of the **function chaining** pattern in C#. An HTTP trigger starts an orchestrator that calls three activity functions in sequence — input validation with a regular expression, processing, and greeting generation — passing each activity's output to the next one.

## ✨ Features

### Functions

| Function | Trigger | Description |
|---|---|---|
| `func-message-validator` | HTTP POST | Starts orchestration, waits for result, returns 200 or 400 |
| `func-message-orchestrator` | Orchestration | Chains the three activity functions |
| `func-validate-input` | Activity | Accepts only letters and spaces (`^[A-Za-z\s]+$`); returns empty string if invalid |
| `func-process-input` | Activity | Passes the value through; returns empty string if input is blank |
| `func-say-hello` | Activity | Returns `"Hello {name}!"` or empty string if name is blank |

## 🚀 Quick start

### Running Locally

```bash
cd Durable/DemoTwo
func start --port 7176
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

### Validation Rule

The `func-validate-input` activity enforces the pattern `^[A-Za-z\s]+$`. Any name containing digits or special characters will be rejected and an empty string will propagate through the pipeline, resulting in a 400 response.

<a id="usage"></a>

**Valid name:**

```bash
curl -X POST http://localhost:7176/api/func-message-validator \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice"}'
```

Intended response: `"Hello Alice!"` — but see [Known issue](#known-issue): the current code returns `400 Bad Request` here too.

**Invalid name (digits or special characters):**

```bash
curl -X POST http://localhost:7176/api/func-message-validator \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice123"}'
```

Response: `400 Bad Request`

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
          │
          ├─► func-validate-input   (validates: letters and spaces only)
          │
          ├─► func-process-input    (passes value through; returns empty if blank)
          │
          └─► func-say-hello        (returns "Hello {name}!" or empty string)
```

### Project Structure

```text
DemoTwo/
├── Functions/
│   ├── FunctionClient.cs          # HTTP trigger — starts orchestration
│   ├── FunctionOrchestrator.cs    # Orchestrator — chains three activities
│   └── Triggers/
│       ├── ValidateInput.cs       # Activity: regex validation
│       ├── ProcessInput.cs        # Activity: pass-through / blank check
│       └── ModifyMessage.cs       # Activity: greeting generation
├── Properties/
│   └── launchSettings.json        # Default port: 7176
├── Program.cs                     # Host builder entry point
├── host.json
└── DemoTwo.csproj
```

## 🧪 Development and testing

From the repository root, build with `dotnet build Durable/DemoTwo/DemoTwo.csproj`. In a configured local host, compare valid and invalid names, inspect all three activity calls and confirm the existing final-response defect. This review did not execute the host or the illustrative fix.

## 🛠️ Troubleshooting and limitations

### Known issue

`FunctionClient.RunAsync` decides whether to return `400` with this check:

```csharp
if (response.SerializedOutput.Contains(""))
```

`string.Contains("")` is always `true`, so every non-null serialized output that reaches this check leads to an empty `400 Bad Request`. (The `badResponse` object that is written with "Request body cannot be processed." is also never returned.) The activity chain is defined separately; malformed inputs or activity failures can still prevent a successful result.

A fix is to deserialize the output and test for an empty string, for example:

```csharp
var output = response.ReadOutputAs<string>();
if (string.IsNullOrEmpty(output))
{
    return new BadRequestObjectResult("Request body cannot be processed.");
}
return new OkObjectResult(output);
```

The regex permits ASCII letters and whitespace (`\s`), including whitespace beyond ordinary spaces. Missing `name` can pass null to the regex and throw; malformed JSON can fail the orchestrator. The final 400 behavior described above applies when orchestration output reaches the final check, not every possible failure path. No timeout or activity retry policy is configured.

## 🔗 Related documentation

### Related Examples

- [DemoOne — the minimal single-activity version](../DemoOne/readme.md)
- [Durable Functions overview](../readme.md)
- [Back to all C# and .NET code examples](../../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp_codes/documentation/) · [Project overview](https://gcfernando.github.io/csharp_codes/) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
