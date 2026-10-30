# Limit Async Concurrency with SemaphoreSlim in C#

A .NET 8 console application demonstrating how to **limit concurrency with `SemaphoreSlim` in C#** — throttling the number of asynchronous operations that run at the same time, a common need when calling APIs, downloading files, or querying databases in parallel. The `FileDownloader` class uses a queue and a semaphore around simulated work; its current startup loop seeds fewer workers than the configured limit.

[C# and .NET examples](../README.md) · **Async programming**

<a id="semaphoreslim-example-in-c--throttle-concurrent-async-downloads-net-8"></a>

## Start here

Read the requirements and any configuration below, then follow the usage example. Source paths below are relative to the repository root; run commands from the working directory stated in Usage. This example’s source is in `SemaphoreSlim/`. Build the selected project with the SDK and platform declared in its project file.

<a id="guide-quick-start"></a>

<a id="-quick-start"></a>

## Quick start

```bash
cd SemaphoreSlim
dotnet run
```

<a id="-prerequisites-and-installation"></a>

## Prerequisites and installation

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download)

<a id="guide-usage"></a>

<a id="-usage"></a>

## Usage

### How It Works

`FileDownloader` accepts a list of file names and a maximum concurrency count. Internally it uses:

- **`ConcurrentQueue<string>`** — thread-safe queue of pending file names
- **`SemaphoreSlim`** — limits the number of downloads running at the same time

`StartDownloadAsync` seeds the initial wave of concurrent tasks (up to the semaphore capacity). Each `DownloadFileAsync` call acquires the semaphore, simulates a download with a random delay (500–2000 ms), releases the semaphore, then immediately picks the next file from the queue — continuing through the queue with the workers that were started.

The class implements `IDisposable`; disposing it releases the semaphore. Its finalizer calls `Dispose(false)`, which does not dispose the semaphore.

### Demo

The `Main` method creates a downloader for 15 files with a maximum of 3 concurrent downloads:

```csharp
using var downloader = new FileDownloader(files, 3);
await downloader.StartDownloadAsync();
```

Console output shows which files are downloading and when each completes, followed by `All files have been downloaded.`

<a id="guide-architecture"></a>

<a id="-architecture"></a>

## Architecture

### Project Structure

```text
SemaphoreSlim/
├── Program.cs         # FileDownloader class + Main entry point
├── Test.csproj
└── Test.sln
```

<a id="-development-and-testing"></a>

## Development and testing

When running locally, check that each of the fifteen simulated file names appears in start/completion messages and that the final message prints. Observe the documented two-worker startup behavior separately from the configured limit of three. This review used source inspection rather than timed execution.

<a id="guide-troubleshooting-and-limitations"></a>

<a id="-troubleshooting-and-limitations"></a>

## Troubleshooting and limitations

### Known behaviour: only two downloads run at once

Although the downloader is created with a limit of 3, source inspection shows that the startup loop initially seeds two tasks:

```csharp
while (tasks.Count < _semaphore.CurrentCount && _fileQueue.TryDequeue(out var file))
```

Each `DownloadFileAsync` call acquires the semaphore synchronously before returning, so `CurrentCount` drops while `tasks.Count` rises: after two tasks, `2 < 1` is false and the loop stops. Each started task then processes the rest of the queue one file at a time, so the effective concurrency is 2.

Comparing against the configured limit instead of the live count — for example, storing `maxConcurrentDownloads` in a field and using `tasks.Count < _maxConcurrency` — starts the intended number of workers.

The names represent simulated downloads: Task.Delay performs no network or file I/O. Completion order varies. With the current startup loop, the default limit of three seeds two workers because CurrentCount decreases while tasks.Count increases. Queue continuation is recursive and there is no cancellation token; review this implementation before adapting it for a large queue.

<a id="-related-documentation"></a>

## Related documentation

### Related Examples

- [Capturing all exceptions from Task.WhenAll](../TaskErrors/readme.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
