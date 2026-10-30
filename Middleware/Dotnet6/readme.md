# Azure Functions Middleware: The .NET 6 Comparison

A side-by-side comparison of two Azure Functions v4 projects on .NET 6: one using **isolated worker middleware** (`WithMiddleware`, built on `IFunctionsWorkerMiddleware`) and one without middleware (`WithoutMiddleware`, an in-process function). Both expose a `SendMessage` HTTP POST endpoint that greets the caller by name, but the middleware version transforms the name to uppercase before the function handler runs.

[C# and .NET examples](../../README.md) · **Azure Functions**

<a id="azure-functions-middleware-vs-no-middleware--net-6-c-comparison"></a>

## Start here

Read the requirements and any configuration below, then follow the usage example. Source paths below are relative to the repository root; run commands from the working directory stated in Usage. This example’s source is in `Middleware/Dotnet6/`. Build the selected project with the SDK and platform declared in its project file.

<a id="guide-features"></a>

<a id="-features"></a>

## Features

### Projects

| Project | Port | Description |
|---|---|---|
| `WithMiddleware` | 7052 | Registers `UppercaseNameMiddleware`; name is uppercased in middleware before reaching the handler |
| `WithoutMiddleware` | 7162 | In-process model (`Microsoft.NET.Sdk.Functions`), no middleware; the raw `name` field from the JSON body is used as-is |

This comparison also changes the hosting model: it is not a comparison of two otherwise identical apps with just one middleware registration removed. An isolated worker app can also run without custom middleware.

<a id="guide-quick-start"></a>

<a id="-quick-start"></a>

## Quick start

From the repository root, use a separate terminal for each command group:

```bash
# With middleware
cd Middleware/Dotnet6/WithMiddleware
func start --port 7052

# Without middleware (separate terminal)
cd Middleware/Dotnet6/WithoutMiddleware
func start --port 7162
```

<a id="-prerequisites-and-installation"></a>

## Prerequisites and installation

### Requirements

- [.NET 6 SDK](https://dotnet.microsoft.com/download)
- [Azure Functions Core Tools v4](https://learn.microsoft.com/azure/azure-functions/functions-run-local)
- A `local.settings.json` in each project (not committed to source control)

#### Minimum `local.settings.json` (WithMiddleware)

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated"
  }
}
```

For the in-process `WithoutMiddleware` project, set `"FUNCTIONS_WORKER_RUNTIME": "dotnet"` instead of `"dotnet-isolated"`.

### NuGet Packages (WithMiddleware)

| Package | Version |
|---|---|
| `Microsoft.Azure.Functions.Worker` | 1.19.0 |
| `Microsoft.Azure.Functions.Worker.Extensions.Http` | 3.0.13 |
| `Microsoft.Azure.Functions.Worker.Sdk` | 1.14.0 |
| `Newtonsoft.Json` | 13.0.1 |

`WithoutMiddleware` references only `Microsoft.NET.Sdk.Functions` 4.2.0.

<a id="guide-usage"></a>

<a id="-usage"></a>

## Usage

### How the Middleware Works

`UppercaseNameMiddleware` implements `IFunctionsWorkerMiddleware`:

1. Reads the HTTP request body as a string
2. Parses it as JSON (`JObject`)
3. Converts the `name` property value to uppercase (or sets it to `"name not found"` if absent)
4. Stores the updated `JObject` in `context.Items["updated_body"]`
5. Resets the request body stream position
6. Calls `next(context)` to pass control to the function handler

The `SendMessageFunction` then reads `context.Items["updated_body"]` and returns `Hello {name}.`

<a id="usage"></a>

```bash
# With middleware — name is returned in uppercase
curl -X POST http://localhost:7052/api/SendMessage \
  -H "Content-Type: application/json" \
  -d '{"name": "alice"}'
```

Response: `Hello ALICE.`

```bash
# Missing name field
curl -X POST http://localhost:7052/api/SendMessage \
  -H "Content-Type: application/json" \
  -d '{}'
```

Response: `Hello name not found.`

```bash
# Without middleware — name is used as-is
curl -X POST http://localhost:7162/api/SendMessage \
  -H "Content-Type: application/json" \
  -d '{"name": "alice"}'
```

Response: `Hello, alice. This HTTP triggered function executed successfully.` (with no `name`, it asks you to pass one).

<a id="-architecture"></a>

## Architecture

### Project Structure

```text
Dotnet6/
├── WithMiddleware/
│   ├── Functions/
│   │   └── SendMessageFunction.cs      # Reads updated_body from context.Items
│   ├── Middlewares/
│   │   └── UppercaseNameMiddleware.cs  # Transforms name to uppercase
│   ├── Program.cs                      # Registers middleware
│   └── WithMiddleware.csproj
├── WithoutMiddleware/
│   ├── Functions/
│   │   └── SendMessageFunction.cs      # Reads name directly from request body
│   └── WithoutMiddleware.csproj
└── readme.md
```

<a id="-development-and-testing"></a>

## Development and testing

Build the two projects independently and compare the same valid-name request on ports 7052 and 7162. Also check missing names, malformed JSON and whether the selected host provides a seekable request stream. These checks were not executed against a running host in this review.

<a id="guide-troubleshooting-and-limitations"></a>

<a id="-troubleshooting-and-limitations"></a>

## Troubleshooting and limitations

Start Azurite for the development storage setting. The isolated middleware sets `requestData.Body.Position = 0`; a non-seekable body stream can fail at that line. It parses and logs the entire request body without an exception handler or sensitive-field redaction. The examples use Bash curl syntax; on PowerShell, use curl.exe with appropriate JSON quoting or Invoke-RestMethod.

<a id="-related-documentation"></a>

## Related documentation

### Related Examples

- [.NET 8 version with exception-handling middleware](../Dotnet8/readme.md)
- [Azure Functions middleware overview](../README.md)
- [Back to all C# and .NET code examples](../../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
