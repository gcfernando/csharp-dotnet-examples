[C# and .NET examples](../README.md) · **Azure Functions**

<a id="azure-durable-functions-examples-in-c--function-chaining-with-the-net-8-isolated-worker"></a>

# Azure Durable Functions Examples in C#

Two runnable **Azure Durable Functions** v4 demos that demonstrate stateful, chained orchestration pipelines using the **isolated worker model on .NET 8**. Each demo starts an orchestration from an HTTP trigger, calls one or more activity functions in sequence (the *function chaining* pattern), and waits for the result before responding.

> **In this guide:** [Get started](#guide-quick-start) · [Walk through the example](#guide-usage) · [Understand the details](#guide-features) · [Check the limits](#guide-troubleshooting-and-limitations)

<a id="guide-features"></a>

## ✨ Features

### Projects

| Project | Port | Pipeline |
|---|---|---|
| [DemoOne](https://github.com/gcfernando/csharp-dotnet-examples/tree/main/Durable/DemoOne) | 7239 | HTTP → Orchestrator → 1 activity (greeting) |
| [DemoTwo](https://github.com/gcfernando/csharp-dotnet-examples/tree/main/Durable/DemoTwo) | 7176 | HTTP → Orchestrator → 3 activities (validate → process → greet) |

<a id="guide-quick-start"></a>

## 🚀 Quick start

### Quick Start

From the repository root, use a separate terminal for each command group:

```bash
# Run DemoOne
cd Durable/DemoOne
func start --port 7239

# Run DemoTwo (in a separate terminal)
cd Durable/DemoTwo
func start --port 7176
```

See each project's own readme — [DemoOne](./DemoOne/readme.md) and [DemoTwo](./DemoTwo/readme.md) — for detailed usage and endpoint examples.

> **Known issue in DemoTwo:** its HTTP starter currently returns `400 Bad Request` after a completed orchestration reaches its output check, including valid names. See [DemoTwo's known issue](./DemoTwo/readme.md#known-issue) for the cause and a fix.

## 📦 Prerequisites and installation

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- [Azure Functions Core Tools v4](https://learn.microsoft.com/azure/azure-functions/functions-run-local)
- [Azurite](https://learn.microsoft.com/azure/storage/common/storage-use-azurite) (local storage emulator) or an Azure Storage account

### Shared NuGet Packages

Both projects use the same package versions:

| Package | Version |
|---|---|
| `Microsoft.Azure.Functions.Worker` | 2.0.0 |
| `Microsoft.Azure.Functions.Worker.Extensions.DurableTask` | 1.2.3 |
| `Microsoft.Azure.Functions.Worker.Extensions.Http` | 3.3.0 |
| `Microsoft.Azure.Functions.Worker.Extensions.Http.AspNetCore` | 2.0.1 |
| `Microsoft.Azure.Functions.Worker.Sdk` | 2.0.2 |

## ⚙️ Configuration

### Minimum `local.settings.json`

Each project requires its own `local.settings.json` (not committed to source control):

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated"
  }
}
```

<a id="guide-usage"></a>

## 💡 Usage

### What Are Durable Functions?

Azure Durable Functions is an extension of Azure Functions that lets you write stateful workflows in a serverless environment. Orchestrator functions define the workflow logic and call activity functions, while the Durable Task Framework manages orchestration state and checkpoints. These samples do not configure activity retry policies.

## 🧪 Development and testing

Build DemoOne and DemoTwo separately, then start each host with its own local settings and storage. Check a normal name, an empty body and malformed JSON; inspect orchestration status as well as the HTTP response. No Functions host or storage emulator was launched in this review.

<a id="guide-troubleshooting-and-limitations"></a>

## 🛠️ Troubleshooting and limitations

Both starters wait for completion without an explicit timeout or cancellation token. Activity retries are not configured in these samples. Local storage settings must be created separately for each project, with Azurite running when using the development connection string.

## 🔗 Related documentation

### Related Examples

- [Azure Functions middleware (.NET 6 and .NET 8)](../Middleware/README.md)
- [Keyed dependency injection in Azure Functions](../KeyedService/readme.md)
- [MediatR and CQRS in Azure Functions](../Mediat/readme.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
