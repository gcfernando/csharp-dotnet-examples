# Get Your Public IP Address in C# with HttpClient

A minimal .NET 6 console application that shows **how to get your public IP address in C#**: it calls the [ipify](https://www.ipify.org/) API with `HttpClient` and prints the result.

[C# and .NET examples](../README.md) · **HTTP and integration**

<a id="get-your-public-ip-address-in-c--httpclient-and-the-ipify-api-net-6"></a>

## Start here

Read the requirements and any configuration below, then follow the usage example. Source paths below are relative to the repository root; run commands from the working directory stated in Usage. This example’s source is in `PublicIP/`. Build the selected project with the SDK and platform declared in its project file.

<a id="guide-quick-start"></a>

<a id="-quick-start"></a>

## Quick start

```bash
cd PublicIP
dotnet run
```

<a id="-prerequisites-and-installation"></a>

## Prerequisites and installation

### Requirements

- [.NET 6 SDK](https://dotnet.microsoft.com/download) plus a compatible .NET 6 runtime
- An active internet connection

<a id="guide-usage"></a>

<a id="-usage"></a>

## Usage

### How It Works

The application sends an HTTP GET request to `http://api.ipify.org`, which returns your public IPv4 address as plain text. The result is printed to the console. If the request fails, the error message is printed instead.

### Example Output

```text
Your public IP Address: 203.0.113.42
```

<a id="guide-architecture"></a>

<a id="-architecture"></a>

## Architecture

### Project Structure

```text
PublicIP/
├── Program.cs          # Fetches and prints the public IP address
├── PublicIP.csproj
└── PublicIP.sln
```

<a id="-development-and-testing"></a>

## Development and testing

If you choose to run the network example, verify either an address message or a handled error, then supply console input to exit. This documentation review inspected the URL and error handler without contacting the external service.

<a id="guide-troubleshooting-and-limitations"></a>

<a id="-troubleshooting-and-limitations"></a>

## Troubleshooting and limitations

### Notes

- The request uses plain `http://api.ipify.org`. ipify also offers `https://api.ipify.org`, and `https://api64.ipify.org` for IPv6 where available.
- The code creates one `HttpClient` for a single request, which is fine for a console demo. In long-running applications, reuse `HttpClient` or use `IHttpClientFactory`.

The source uses plain HTTP and depends on an external service. Network or proxy failures are caught and printed; there is no retry policy or caller-provided cancellation token. After the request, Console.Read waits for input. The example IP address is illustrative, not a captured network result.

<a id="-related-documentation"></a>

## Related documentation

### Related Examples

- [Python internet speed test](https://github.com/gcfernando/python-projects-and-examples/blob/main/Internet%20Speed%20Test/readme.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
