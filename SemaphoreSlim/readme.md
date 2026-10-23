# SemaphoreSlim Example in C# — Throttle Concurrent Async Downloads (.NET 8)

A .NET 8 console application demonstrating how to **limit concurrency with `SemaphoreSlim` in C#** — throttling the number of asynchronous operations that run at the same time, a common need when calling APIs, downloading files, or querying databases in parallel. The `FileDownloader` class uses a queue and a semaphore around simulated work; its current startup loop seeds fewer workers than the configured limit.

## 🚀 Quick start

### Build and Run

```bash
cd SemaphoreSlim
dotnet run
```

## 📦 Prerequisites and installation

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download)

## 💡 Usage

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

## 🏗️ Architecture

### Project Structure

```text
SemaphoreSlim/
├── Program.cs         # FileDownloader class + Main entry point
├── Test.csproj
└── Test.sln
```

## 🧪 Development and testing

When running locally, check that each of the fifteen simulated file names appears in start/completion messages and that the final message prints. Observe the documented two-worker startup behavior separately from the configured limit of three. This review used source inspection rather than timed execution.

## 🛠️ Troubleshooting and limitations

### Known behaviour: only two downloads run at once

Although the downloader is created with a limit of 3, source inspection shows that the startup loop initially seeds two tasks:

```csharp
while (tasks.Count < _semaphore.CurrentCount && _fileQueue.TryDequeue(out var file))
```

Each `DownloadFileAsync` call acquires the semaphore synchronously before returning, so `CurrentCount` drops while `tasks.Count` rises: after two tasks, `2 < 1` is false and the loop stops. Each started task then processes the rest of the queue one file at a time, so the effective concurrency is 2.

Comparing against the configured limit instead of the live count — for example, storing `maxConcurrentDownloads` in a field and using `tasks.Count < _maxConcurrency` — starts the intended number of workers.

The names represent simulated downloads: Task.Delay performs no network or file I/O. Completion order varies. With the current startup loop, the default limit of three seeds two workers because CurrentCount decreases while tasks.Count increases. Queue continuation is recursive and there is no cancellation token; review this implementation before adapting it for a large queue.

## 🔗 Related documentation

### Related Examples

- [Capturing all exceptions from Task.WhenAll](../TaskErrors/readme.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp_codes/documentation/) · [Project overview](https://gcfernando.github.io/csharp_codes/) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
