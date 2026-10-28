[C# and .NET examples](../README.md) · **SQL Server and data**

<a id="sql-server-upsert-in-c--row-by-row-vs-table-valued-parameters-vs-sqlbulkcopy-net-10"></a>

# SQL Server Upserts in C#: TVP and SqlBulkCopy

A .NET 10 solution comparing three **SQL Server upsert strategies in C#** with `Microsoft.Data.SqlClient` for a `CustomerActivityEvent` record: row-by-row transactions, **table-valued parameters (TVP)** with a stored procedure, and **`SqlBulkCopy`** into a staging table followed by a reconcile procedure. All strategies support an optional optimistic-concurrency check (`rowversion`), ignore out-of-order (older) events, and are wrapped in a shared transient-fault retry policy with overload-specific retry behavior.

> **In this guide:** [Get started](#guide-quick-start) · [Walk through the example](#guide-usage) · [Understand the details](#guide-architecture) · [Check the limits](#guide-troubleshooting-and-limitations)

<a id="guide-quick-start"></a>

## 🚀 Quick start

### Build and Run

After reviewing the configuration below and preparing a disposable database, run from the repository root (each command is independent):

```bash
# Normal strategy demo (upserts a single event)
dotnet run --project SQLWrites/UseNormal

# TVP strategy demo (sends a batch of 2 000 events)
dotnet run --project SQLWrites/UseTvp

# Bulk Staging strategy demo (streams 200 000 events)
dotnet run --project SQLWrites/UseBulk
```

## 📦 Prerequisites and installation

### Requirements

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- SQL Server (local or remote)

### NuGet Packages

| Package | Version |
|---|---|
| `Microsoft.Data.SqlClient` | 6.1.3 |

## ⚙️ Configuration

### Database Setup

Run `Query.sql` against your SQL Server database to create:

- `dbo.CustomerActivityEvents` — main table with `row_version` column and unique index on `event_id`
- `dbo.CustomerActivityEventType` — TVP type definition
- `dbo.UpsertCustomerActivityEventsType` — stored procedure for the TVP strategy
- `dbo.CustomerActivityEvents_Staging` — staging table for the bulk strategy
- `dbo.ReconcileCustomerActivityEventsBatch` — stored procedure for bulk reconciliation

<a id="configuration"></a>

Edit the connection string in each project's `Program.cs`:

```csharp
var connectionString = "Data Source=<server>;Initial Catalog=<db>;Integrated Security=True;Trust Server Certificate=True";
```

<a id="guide-usage"></a>

## 💡 Usage

### The Data Model

`CustomerActivityEvent` is a sealed record with:

| Field | Type | Description |
|---|---|---|
| `EventId` | `Guid` | Immutable event identity |
| `CustomerId` | `int` | Who performed the activity |
| `ActivityType` | `string` | What happened |
| `TimeStampUtc` | `DateTime` | When it happened |
| `DetailsJson` | `string` | JSON payload / metadata |
| `RowVersion` | `byte[]` | Current row version (from DB) |
| `ExpectedRowVersion` | `byte[]` | Optimistic concurrency check value |

Upsert logic applies updates only when the incoming `TimeStampUtc` is newer than the stored value, providing built-in out-of-order protection.

### Upsert Strategies

#### 1. Normal — `UseNormal`

**Best for:** low volume, simple correctness requirements.

- Opens a transaction per event
- `UPDATE ... WHERE event_id = @id AND time_stamp_utc < @new_ts` (with optional `row_version` check)
- `INSERT ... WHERE NOT EXISTS (SELECT 1 ... WITH (UPDLOCK, HOLDLOCK))`
- Commits; rolls back on failure
- Supports `UpsertAsync` (single event) and `UpsertManyAsync` (loop over a list)

#### 2. TVP — `UseTvp`

**Best for:** medium-volume batch ingestion via stored procedures.

- Builds a `DataTable` from a list of events
- Passes it as a Table-Valued Parameter to `dbo.UpsertCustomerActivityEventsType`
- The stored procedure stages input, updates older stored rows, and inserts new events in a single transaction. The TVP type declares `event_id` as its primary key: duplicate IDs must be resolved before sending the parameter; the procedure's `ROW_NUMBER` step does not make duplicate TVP keys acceptable.

#### 3. Bulk Staging — `UseBulk`

**Demonstrates:** staging-based ingestion; the demo generates 200,000 events, without a published benchmark.

- Generates a `batchId` (GUID)
- Streams events into `dbo.CustomerActivityEvents_Staging` via `SqlBulkCopy` (batch size: 50 000, table lock, streaming enabled)
- Calls `dbo.ReconcileCustomerActivityEventsBatch` which deduplicates, updates, and inserts — returning `RowsUpdated`, `RowsInserted`, `RowsConflicted` as OUTPUT parameters
- Cleans up the staging table after reconciliation

### Strategy Comparison

| Strategy | Best For | Notes |
|---|---|---|
| Normal | Per-event transactions | Executes update and insert commands per event |
| TVP | Medium-volume batches | Clean SQL boundary via stored proc |
| Bulk Staging | Staging-based batches | Requires a staging table and reconcile procedure; measure performance for your workload |

### Retry Policy (`Common/SqlRetryPolicy`)

The non-generic `ExecuteAsync` overload catches selected `SqlException` failures and permits five retries after the initial attempt, using exponential backoff (100 ms base, capped at 5,000 ms) with jitter. The generic `ExecuteAsync<T>` overload permits five total attempts by default and uses linear 200 ms increments; its filter also recognizes `TimeoutException` and an inner `IOException`. These overloads do not share one identical policy. See [SqlRetryPolicy.cs](https://github.com/gcfernando/csharp-dotnet-examples/blob/main/SQLWrites/Common/SqlRetryPolicy.cs) for the full filters.

<a id="guide-architecture"></a>

## 🏗️ Architecture

### Project Structure

```text
SQLWrites/
├── Common/
│   ├── Model.cs               # CustomerActivityEvent record
│   └── SqlRetryPolicy.cs      # Transient-fault retry logic
├── UseNormal/
│   ├── CustomerActivityEventWriter.cs            # Normal upsert
│   └── Program.cs
├── UseTvp/
│   ├── CustomerActivityEventWriter.cs            # CustomerActivityEventTvpWriter — TVP batch upsert
│   └── Program.cs
├── UseBulk/
│   ├── CustomerActivityEventBulkStagingWriter.cs # Bulk copy + reconcile
│   ├── CustomerActivityEventDataReader.cs        # Custom IDataReader for SqlBulkCopy
│   └── Program.cs
├── Query.sql                  # SQL DDL: tables, types, stored procedures
└── SQLWrites.sln
```

## 🧪 Development and testing

A build that does not run the writers is `dotnet build SQLWrites/SQLWrites.sln` from the repository root. Validate schema and behavior only against a disposable database. Include duplicate IDs, equal/older timestamps, mismatched row versions, partial bulk-copy failure and retries after an uncertain commit. This documentation review did not run SQL scripts or database writers.

<a id="guide-troubleshooting-and-limitations"></a>

## 🛠️ Troubleshooting and limitations

### FAQ

**Which is the fastest way to insert or update many rows in SQL Server from C#?**
For large volumes, `SqlBulkCopy` into a staging table followed by a set-based reconcile (the `UseBulk` strategy) avoids per-row round trips. TVPs are a good middle ground for batches sent to a stored procedure, and row-by-row upserts are simplest for low volumes. This project does not publish benchmark numbers — measure with your own data and server.

**Why does the insert use `UPDLOCK, HOLDLOCK`?**
Without those hints, two concurrent sessions can both see that a row is missing and both try to insert it. The hints keep the existence check and the insert consistent under concurrency.

**What happens on a deadlock or timeout?**
Both overloads recognize selected transient SQL errors, including deadlock 1205 and timeout -2. The non-generic overload uses exponential backoff with jitter; the generic overload uses linear delays and a different attempt count. See the retry-policy section above.

Use a disposable database. Query.sql creates/alters schema and drops/recreates an existing index, procedure and table type. The normal demo generates one event, TVP 2,000, and bulk 200,000; these are input sizes, not measured performance. Connection strings are hard-coded in each Program.cs and are not read from environment variables. Re-running the demos generates fresh event IDs and adds rows.

### Concurrency and retry boundaries

`ExpectedRowVersion` is optional: an empty array becomes SQL `NULL`, bypassing the row-version match. For an existing event, equal or older timestamps cause no update. `RowVersion` is carried by the record but none of the writers reads a fresh version back into it. The Normal and TVP methods return no conflict detail; the Bulk method returns counts while discarding the procedure's conflict result rows. Successful completion therefore does not mean every supplied event changed a row.

Bulk copy uses its own transactions for batches of 50,000, followed by a separate reconciliation transaction. These steps do not form one transaction. A failure can leave committed staging rows, and the retry reuses its batch ID and enumerates the input again. `StreamEvents()` in the demo creates new GUIDs and timestamps on each enumeration, so a retry can represent different events from its first attempt. For adaptation, define stable event identities, repeatable input and a staging recovery/cleanup policy; this code has not demonstrated exactly-once processing.

The reconciliation query reads staging rows with `READPAST` and then deletes rows for the batch ID. Concurrent loading/reconciliation of the same batch requires separate review; do not infer that the sample implements a queue, row claiming or a recovery protocol. Conflict detection and update are separate statements, and their behavior under competing writers was not runtime-verified.

`SQLWrites/UseTvp/bin` and `obj` contain tracked build outputs, dependency DLLs and caches in this checkout. Use the project/source files for builds and review; the committed outputs do not establish that the current source has passed validation.

## 🔗 Related documentation

### Related Examples

- [Contact Book — Python GUI using SQL Server stored procedures](https://github.com/gcfernando/python-projects-and-examples/blob/main/GUI_Apps/README.md)
- [Local SQL Server and other databases with Podman Compose](https://github.com/gcfernando/developer-tools-and-guides#-local-services-stack)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp-dotnet-examples/documentation/) · [Project overview](../README.md) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
