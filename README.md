# C# and .NET Examples — Azure Functions, Async and SQL Server

Focused C# samples for learning Azure Functions, asynchronous programming, SQL Server upserts, Windows Forms, terminal interfaces and Python interoperability.

## 🔎 Overview

Choose one sample at a time. This repository contains independent solutions targeting .NET 6, .NET 8 and .NET 10, with different runtime and service requirements. It has no single application or root solution.

The [documentation index](documentation.md) lists every sample guide and its prerequisites. These are learning examples; read the limitations before adapting a pattern.

## ✨ Features

| Area | Examples | Target |
|---|---|---|
| Azure Functions | [Middleware](Middleware/README.md), [Durable orchestration](Durable/readme.md), [keyed services](KeyedService/readme.md), [MediatR / CQRS](Mediat/readme.md) | .NET 6 or 8, Functions v4 |
| Async and memory | [Task.WhenAll failures](TaskErrors/readme.md), [SemaphoreSlim queue](SemaphoreSlim/readme.md), [Unsafe.As](UnsafeAs/readme.md) | .NET 8 |
| Data | [SQL Server upserts](SQLWrites/readme.md), [JSONPath queries](JsonSearchPatterns/readme.md) | .NET 10 / .NET 6 |
| Integration | [Python.NET bridge](PythonBridge/readme.md), [Azure OpenAI chat completion](Simple_ChatCompletion/readme.md), [public IP lookup](PublicIP/readme.md) | .NET 8 / .NET 6 |
| Interfaces and tests | [Windows Forms DPI](FourKWinForm/README.md), [Spectre.Console](SpectreDemo/readme.md), [FakeLogger tests](Fake/readme.md) | .NET 8 |

## 🚀 Quick start

Run a small console example that needs no external service, from a terminal with the .NET 8 SDK and runtime installed:

```bash
git clone https://github.com/gcfernando/csharp_codes.git
cd csharp_codes
dotnet run --project TaskErrors/TaskErrors.csproj
```

The two failing tasks are caught and their messages are printed:

```text
Custom Error Method One
Custom Error Method Two
```

This is expected output inferred from [Program.cs](TaskErrors/Program.cs), not a live execution result from this documentation review.

## 📦 Prerequisites and installation

Install an SDK capable of building the selected target and the corresponding runtime for execution. A newer SDK alone does not guarantee an older runtime is installed. Restore uses the package references in each `.csproj`; there is no repository-wide dependency installation step.

| Sample type | Additional requirements |
|---|---|
| Functions | Azure Functions Core Tools v4; local settings; Azurite when using `UseDevelopmentStorage=true` |
| SQLWrites | .NET 10 and a disposable SQL Server database; the sample writes records and changes schema |
| PythonBridge | Compatible Python 3.12 DLL and architecture; Python packages listed in its guide |
| Simple_ChatCompletion | Azure OpenAI endpoint, deployment and credentials; execution makes paid service requests |
| FourKWinForm | Windows and the .NET 8 Windows Desktop runtime |

## ⚙️ Configuration

Functions projects expect an untracked `local.settings.json` in the function project folder. The correct `FUNCTIONS_WORKER_RUNTIME` differs between the .NET 6 in-process baseline and isolated projects; follow the [middleware guide](Middleware/README.md).

SQL connection strings, Python DLL selection and Azure OpenAI constants currently live in sample source files. These applications do not all read environment variables or .NET user secrets automatically. Read the sample guide before changing configuration and keep real credentials out of commits.

## 💡 Usage

From the repository root, choose an explicit project or solution:

```bash
dotnet test Fake/Fake.sln
dotnet run --project SpectreDemo/SpectreDemo.csproj
```

SpectreDemo needs an interactive terminal. JSONPath reads `Data.json` relative to the working directory, so run it from its folder:

```bash
cd JsonSearchPatterns
dotnet run
```

Start Functions with `func start` from the function project folder after its local settings and storage emulator are ready. See each guide for routes and ports.

## 🏗️ Architecture

Folders isolate the examples. `Fake`, `SQLWrites` and `Mediat` contain multiple projects and solution files; `Durable` and `Middleware` group several independently hosted apps. Most other folders contain one console or desktop application. The `.csproj` files are the source of truth for target frameworks and packages.

The `_config.yml`, `_layouts` and `assets` folders support the documentation site, independently of the C# applications.

## 🧪 Development and testing

Build only the sample you are changing, for example `dotnet build TaskErrors/TaskErrors.csproj`. The committed automated test project is [Fake.Test](Fake/Fake.Test/CalculateTest.cs), with two MSTest checks for log messages and error levels. There is no shared test suite or CI workflow in this checkout.

Documentation verification checked source, project references and local links. It did not launch GUIs, Functions hosts, SQL scripts, Python embedding or Azure OpenAI requests.

## 🛠️ Troubleshooting and limitations

- **Wrong SDK or runtime:** inspect the selected `.csproj` and `dotnet --list-sdks` / `dotnet --list-runtimes`.
- **Functions startup:** confirm the worker model, local settings and storage availability before testing HTTP routes.
- **Known sample defects:** [DemoTwo](Durable/DemoTwo/readme.md#known-issue) returns 400 after a completed orchestration, and [SemaphoreSlim](SemaphoreSlim/readme.md) seeds fewer workers than its configured limit. The guides explain these existing behaviors.
- **Integration examples:** PythonBridge has a model-file path mismatch; Mediat has mixed MediatR package generations and an unsynchronized in-memory store. Review their guides before use.
- **Performance:** SQLWrites contains example batch sizes, not published benchmark results.

## 🤝 Contributing

Keep changes focused on one sample, state its target framework and prerequisites, and include relevant build/test evidence. Update the sample guide when changing configuration, output or known limitations. Use placeholder data and credentials.

## 🔒 Security

Do not include secrets or personal data in issues, logs or examples. For sensitive reports, use a private reporting option if the repository provides one; otherwise contact the maintainer privately before sharing details. This checkout defines no dedicated security policy.

## 📄 Support and license

Use [repository issues](https://github.com/gcfernando/csharp_codes/issues) for reproducible questions and defects. No repository license file is present in this checkout; confirm permission before reusing or redistributing code. Package dependencies retain their own licenses.

[Documentation index](documentation.md) · [Python examples](https://github.com/gcfernando/python_codes) · [VB.NET binary clock](https://github.com/gcfernando/vb_codes)
