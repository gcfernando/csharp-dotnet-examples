---
layout: default
title: "C# and .NET Code Examples — Documentation and Guides"
description: "Practical C# examples for Azure Functions, async programming, SQL Server, dependency injection and testing."
permalink: /documentation/
---

Browse the published guides, examples and reference pages for **C# and .NET Code Examples**. Each guide has a readable web page; the Markdown sources remain available on GitHub.

## 🔎 Choose a sample guide

Use this index to find the relevant framework, dependencies and limitations before running a sample. Start with [the repository quick start](README.md) for a console-only example.

| Guide | What to inspect first |
|---|---|
| [Durable Functions overview](Durable/readme.md) | Shared storage/local settings and function chaining |
| [Durable DemoOne](Durable/DemoOne/readme.md) | HTTP starter, single activity and serialized output |
| [Durable DemoTwo](Durable/DemoTwo/readme.md) | Three activities and the existing final-response defect |
| [Middleware overview](Middleware/README.md) | Isolated versus in-process hosting |
| [.NET 6 middleware](Middleware/Dotnet6/readme.md) | Different worker runtime settings; seekable body assumption |
| [.NET 8 middleware](Middleware/Dotnet8/readme.md) | Middleware registration order and exception coverage |
| [Keyed services](KeyedService/readme.md) | Key registration and log-only notification implementations |
| [MediatR / CQRS](Mediat/readme.md) | Layer boundaries, mixed package generations and volatile storage |
| [FakeLogger tests](Fake/readme.md) | Two MSTest cases and their exact assertions |
| [JSONPath queries](JsonSearchPatterns/readme.md) | .NET 6; working directory must contain Data.json |
| [Public IP lookup](PublicIP/readme.md) | .NET 6; external plain-HTTP request |
| [Python.NET bridge](PythonBridge/readme.md) | Python DLL/packages and model save/load path mismatch |
| [SemaphoreSlim](SemaphoreSlim/readme.md) | Simulated work and worker-seeding limitation |
| [Azure OpenAI SDK / REST](Simple_ChatCompletion/readme.md) | Compile-time credentials/settings and unvalidated model output |
| [Spectre.Console](SpectreDemo/readme.md) | Interactive terminal and simulated progress |
| [SQL Server upserts](SQLWrites/readme.md) | .NET 10, disposable database and retry overload differences |
| [Task.WhenAll failures](TaskErrors/readme.md) | Inspecting all task exceptions |
| [Unsafe.As](UnsafeAs/readme.md) | Memory layout and little-endian output assumptions |
| [High-DPI WinForms](FourKWinForm/README.md) | Windows and system DPI awareness |

## 🧭 Reading and verification

Each guide points to its source structure and records execution limits. The project files define package versions and targets; documentation examples do not establish runtime compatibility or benchmark results. No root-level build or test command covers all samples.

The sections below retain the site's generated page/source navigation for published documentation.

## Read online

<table>
<thead><tr><th>Guide</th><th>Markdown or HTML source</th></tr></thead>
<tbody>
{% assign guides = site.html_pages | sort: 'url' %}
{% for guide in guides %}{% if guide.sitemap != false and guide.url != page.url and guide.url != '/404.html' %}
<tr><td><a href="{{ guide.url | absolute_url }}">{{ guide.title | default: guide.name | escape }}</a></td><td><a href="https://github.com/{{ site.repository }}/blob/main/{{ guide.path | uri_escape }}">View source</a></td></tr>
{% endif %}{% endfor %}
{% for guide in site.static_files %}{% if guide.extname == '.html' and guide.sitemap != false %}
<tr><td><a href="{{ guide.path | replace: '/index.html', '/' | absolute_url }}">{% if guide.path == '/Linux_Commands/index.html' %}Linux Command Reference{% elsif guide.path contains 'ai_coding_plugins_beginners_guide' %}AI Coding Add-ons: Beginner’s Guide{% else %}{{ guide.name | escape }} — {{ guide.path | escape }}{% endif %}</a></td><td><a href="https://github.com/{{ site.repository }}/blob/main{{ guide.path | uri_escape }}">View HTML source</a></td></tr>
{% endif %}{% endfor %}
</tbody>
</table>

## Markdown source library

[View this documentation index source](https://github.com/{{ site.repository }}/blob/main/documentation.md). These are the authored Markdown files. Some are package-specific documents or alternate source formats of the same guide.

{% assign sources = site.static_files | sort: 'path' %}
{% for source in sources %}{% if source.extname == '.md' %}
- [{{ source.path | escape }}](https://github.com/{{ site.repository }}/blob/main{{ source.path | uri_escape }})
{% endif %}{% endfor %}

## Explore more

[All developer projects](https://gcfernando.github.io/gcfernando/projects/) · [Repository on GitHub](https://github.com/gcfernando/csharp_codes) · [XML sitemap](https://gcfernando.github.io/csharp_codes/sitemap.xml)
