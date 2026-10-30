# Spectre.Console: Rich Terminal Interfaces in C#

A .NET 8 console application demonstrating how to build a **rich terminal UI in C#** with the [Spectre.Console](https://spectreconsole.net/) library. The app simulates an interactive data collection workflow with live panels, multi-task progress bars, and a selection prompt — all with coloured text and animated UI elements.

[C# and .NET examples](../README.md) · **Terminal interfaces**

<a id="spectreconsole-example-in-c--live-panels-progress-bars-and-prompts-net-8"></a>

## Start here

Read the requirements and any configuration below, then follow the usage example. Source paths below are relative to the repository root; run commands from the working directory stated in Usage. This example’s source is in `SpectreDemo/`. Build the selected project with the SDK and platform declared in its project file.

<a id="guide-features"></a>

<a id="-features"></a>

## Features

<a id="features"></a>

- **Live panel** — updates in-place through four stages: Connecting, Fetching Data, Processing, and Complete
- **Multi-task progress bar** — three simulated task displays (Fetching User Data, Downloading Reports, Analyzing Logs) with percentage, spinner, and remaining-time columns
- **Selection prompt** — asks whether to restart the workflow or exit
- **Continuous loop** — the entire workflow repeats until the user selects "No"

<a id="guide-quick-start"></a>

<a id="-quick-start"></a>

## Quick start

```bash
cd SpectreDemo
dotnet run
```

<a id="-prerequisites-and-installation"></a>

## Prerequisites and installation

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download)

### NuGet Packages

| Package | Version |
|---|---|
| `Spectre.Console` | 0.49.1 |
| `Spectre.Console.Cli` | 0.49.1 |

<a id="guide-usage"></a>

<a id="-usage"></a>

## Usage

### Workflow

1. Live panel cycles through four status messages with brief delays
2. Progress bar advances three tasks to completion (each at a different increment rate)
3. User is prompted to restart or exit

### Spectre.Console APIs Used

| API | Used for |
|---|---|
| `AnsiConsole.Live` with `Panel` and `UpdateTarget` | Updating a panel in place |
| `AnsiConsole.Progress` with `AddTask` and `Increment` | Several progress bars advancing together |
| `SelectionPrompt<string>` with `AddChoices` | The restart / exit prompt |
| Markup such as `[green]...[/]` | Coloured and styled text |

`Spectre.Console.Cli` is referenced in the project file but not used by `Program.cs`.

<a id="-architecture"></a>

## Architecture

### Project Structure

```text
SpectreDemo/
├── Program.cs          # Full application — live panel, progress bar, selection prompt
├── SpectreDemo.csproj
└── SpectreDemo.sln
```

<a id="-development-and-testing"></a>

## Development and testing

In an interactive terminal, check the four panel stages, completion of all three progress bars and both restart choices. The source and package references were checked; terminal interaction was not executed in this review.

<a id="guide-troubleshooting-and-limitations"></a>

<a id="-troubleshooting-and-limitations"></a>

## Troubleshooting and limitations

The progress bars are advanced by one synchronous loop with Thread.Sleep. They represent simulated jobs, not parallel downloads or analysis. Use an interactive terminal for the selection prompt; redirected output may not render live updates or accept selections.

<a id="-related-documentation"></a>

## Related documentation

### Related Examples

- [SemaphoreSlim concurrency example](../SemaphoreSlim/readme.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
