# Azure OpenAI Chat Completion in C# — Azure.AI.OpenAI SDK vs REST API (.NET 8)

A .NET 8 console application that demonstrates two ways to call an **Azure OpenAI chat completion** endpoint from C#: via the **`Azure.AI.OpenAI` NuGet package** and via a **raw HTTP REST call**. Both approaches use the same system prompt, which instructs the model to extract a product code and attributes from a natural-language query and return structured JSON.

## 🔎 Overview

<a id="overview"></a>

The application runs two test queries against each connection method:

1. A product-related query: `"Can you help me to find the values of Sealing, Matched arrangement in 6205?"`
2. An unrelated query: `"What is the capital of Sri Lanka?"`

The system message instructs the model to extract product codes (e.g. `6205`) and attributes from the user query and return:

```json
{ "product": "<product_code>", "attributes": ["<attr1>", "<attr2>"] }
```

For queries unrelated to products, the system prompt requests a polite refusal; the code does not enforce that behavior.

## 🚀 Quick start

### Build and Run

```bash
cd Simple_ChatCompletion
dotnet run
```

## 📦 Prerequisites and installation

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- An Azure OpenAI resource with a `gpt-4o-mini` deployment

### NuGet Packages

| Package | Version |
|---|---|
| `Azure.AI.OpenAI` | 2.1.0 |
| `Newtonsoft.Json` | 13.0.3 |

## ⚙️ Configuration

<a id="configuration"></a>

All connection settings are defined in `Configuration.cs`:

| Setting | Description |
|---|---|
| `EndPoint` | Azure OpenAI resource URL |
| `Key` | API key |
| `Model` | Deployment name (`gpt-4o-mini`) |
| `Version` | API version (`2024-08-01-preview`) |
| `SystemMessage` | System prompt for product code extraction |

> **Security note:** Do not commit a real API key. The source currently reads `Configuration.Key`; implement a private runtime configuration source before using real credentials. Environment variables, user secrets, Key Vault and Entra authentication are not wired into this sample. Rotate any real key that has been exposed.

## 💡 Usage

### Connection Methods

#### PackageDemo — `Azure.AI.OpenAI` SDK

Uses `AzureOpenAIClient` and `GetChatClient` to send a `ChatCompletionOptions` request with `Temperature = 0.7f` and `MaxOutputTokenCount = 100`.

#### RestDemo — Raw HTTP

Uses `HttpClient` to POST directly to the Azure OpenAI REST endpoint, with a `Newtonsoft.Json`-serialised request body, extracting the response text from `choices[0].message.content`.

### Sample Data

`Data/6205.json` contains SKF bearing specifications for part number 6205. It is copied to the build output as sample reference data, but the current code does not read it or send it to the model — the model only extracts the product code and attributes from the question.

### Example Output

The following illustrates the requested response format; it is not a captured or guaranteed model response:

```text
** Package Demo **
{"product": "6205", "attributes": ["Sealing", "Matched arrangement"]}
I am sorry, I cannot help with your request. Your question is not related to products.

** Rest Demo **
{"product": "6205", "attributes": ["Sealing", "Matched arrangement"]}
I am sorry, I cannot help with your request. Your question is not related to products.
```

## 🏗️ Architecture

### Project Structure

```text
Simple_ChatCompletion/
├── ConnectByPackage/
│   └── PackageDemo.cs       # SDK-based chat completion
├── ConnectByRest/
│   └── RestDemo.cs          # Raw HTTP REST chat completion
├── Data/
│   └── 6205.json            # SKF bearing 6205 specification data
├── Configuration.cs         # Endpoint, key, model, and system message constants
├── Program.cs               # Runs both demos with two test queries
├── Simple_ChatCompletion.csproj
└── Simple_ChatCompletion.sln
```

## 🧪 Development and testing

Build the project after reviewing credential handling. A separate integration check would need an authorized Azure OpenAI deployment and should inspect SDK and REST failures as well as successful responses. No service requests or credentials were used in this documentation review.

## 🛠️ Troubleshooting and limitations

### FAQ

**Should I use the SDK or the REST API?**
The `Azure.AI.OpenAI` SDK gives typed requests and responses and handles authentication options for you. The raw REST call shows exactly what goes over the wire and works from any HTTP client; it is useful for debugging or when you cannot take an SDK dependency.

**What is the `Model` setting?**
In Azure OpenAI, requests target a *deployment name* that you create in your resource. This demo expects a deployment named `gpt-4o-mini`.

Settings are compile-time constants. Neither caller reads environment variables, user secrets, Key Vault or Entra credentials as written; adding those requires a code change. The REST URL concatenation expects EndPoint to end with a slash. The sample config names an API version and deployment, but current resource availability and access were not verified. Four requests are attempted in sequence if all earlier calls succeed. JSON output is requested by a prompt only, with no response schema or JSON validation; model text and refusals are not guaranteed.

## 🔗 Related documentation

### Related Examples

- [Local AI chat app with Ollama and Streamlit](https://github.com/gcfernando/developer-guides-and-cheatsheets/blob/main/local_ai_learning_lab.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
