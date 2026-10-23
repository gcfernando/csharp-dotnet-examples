# Call Python from C# with Python.NET (pythonnet) — .NET 8 Examples

A .NET 8 console application demonstrating **how to call Python code from C#** using [Python.NET](https://github.com/pythonnet/pythonnet) (`pythonnet` 3.0.5) — calling Python functions, passing C# objects to Python, running a scikit-learn model, and hosting a FastAPI app from a .NET process. The project covers four integration scenarios — from basic arithmetic to ML model inference and launching a FastAPI server.

## 🔎 Overview

<a id="overview"></a>

Python.NET embeds a Python interpreter (CPython) inside the .NET process. The `Py.GIL()` block acquires the Global Interpreter Lock, after which you can import Python modules, call functions, and exchange objects between C# and Python dynamically.

## 🚀 Quick start

### Build and Run

After installing the matching Python DLL and packages, build and run from the output directory so model training and loading use the same location:

```bash
cd PythonBridge
dotnet build
cd bin/Debug/net8.0
dotnet PythonBridge.dll
```

Program.cs invokes the four scenarios sequentially and waits for input if they all return. Earlier embedding or model-loading failures can prevent later scenarios from running. The `PyScript/*.py` files are copied to the build output, and each caller adds that folder to Python's `sys.path` before importing the modules.

## 📦 Prerequisites and installation

### Requirements

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- **Python 3.12** installed with `python312.dll` accessible (system PATH or application directory). Each caller sets `Runtime.PythonDLL = "python312.dll"`; change it to match your Python version.
- Python packages:

```bash
python -m pip install joblib numpy scikit-learn fastapi uvicorn
```

### NuGet Packages

| Package | Version |
|---|---|
| `pythonnet` | 3.0.5 |

## 💡 Usage

### Scenarios

#### 1. Basic Operations (`Basic.Execute`)

Imports `basic_operations.py` and calls:

| Python Function | Description |
|---|---|
| `add_numbers(10, 20)` | Returns the sum |
| `multiply_numbers(5, 6)` | Returns the product |
| `get_greeting("John")` | Returns a greeting string |

Also reads the module-level constant `PI`.

#### 2. Log Formatting (`LogFormat.Execute`)

Imports `logger.py` inside a named Python scope. Passes C# `LogEntry` objects (`Level`, `Message`) to `logger.format_entry(entry)` and prints the formatted output.

#### 3. Iris ML Inference (`IrisCode.Execute`)

- If `iris_nb.pkl` does not exist, runs `train_iris_model.py` to train a Naive Bayes classifier on the Iris dataset and save the model
- Loads the saved model with `joblib`, builds a NumPy input array, and prints the predicted class for `[5.1, 3.5, 1.4, 0.2]`
- Uses `PythonEngine.BeginAllowThreads` / `EndAllowThreads` to release the GIL during non-Python work

#### 4. FastAPI Server (`ApiCode.Execute`)

Imports `api.py` (which defines a FastAPI `app`) and starts it with `uvicorn` on `127.0.0.1:8000`. The app exposes two GET endpoints:

| Endpoint | Response |
|---|---|
| `/` | `{"message": "Hello, World! from Python.NET with FastAPI."}` |
| `/hello/{name}` | `{"message": "Hello, {name}! from Python.NET with FastAPI."}` |

`uvicorn.run` blocks while the server is running, so the application stays on this step until you stop the server (for example with `Ctrl+C`).

### Python Scripts (`PyScript/`)

| Script | Description |
|---|---|
| `basic_operations.py` | `add_numbers`, `multiply_numbers`, `get_greeting`, and `PI` constant |
| `logger.py` | `format_entry(entry)` — formats a log entry object |
| `train_iris_model.py` | Trains a Naive Bayes classifier on Iris data, saves `iris_nb.pkl` |
| `api.py` | FastAPI application |

## 🏗️ Architecture

### Project Structure

```text
PythonBridge/
├── PyScript/
│   ├── basic_operations.py      # Arithmetic and greeting functions
│   ├── logger.py                # Log entry formatter
│   ├── train_iris_model.py      # Iris model training script
│   └── api.py                   # FastAPI application
├── PythonCaller/
│   ├── Basic.cs                 # Calls basic_operations.py
│   ├── LogFormat.cs             # Calls logger.py with LogEntry objects
│   ├── IrisCode.cs              # Trains or loads Iris model, runs inference
│   └── ApiCode.cs               # Starts FastAPI server via uvicorn
├── LogEntry.cs                  # Log entry DTO (Level, Message)
├── Program.cs                   # Entry point — runs all four scenarios
├── PythonBridge.csproj
└── PythonBridge.sln
```

## 🧪 Development and testing

From the repository root, `dotnet build PythonBridge/PythonBridge.csproj` checks the .NET project. Python imports, embedding, model training and the FastAPI server need separate runtime checks; none were launched during this documentation review.

## 🛠️ Troubleshooting and limitations

### Troubleshooting

- **`Runtime.PythonDLL` / `DllNotFoundException`** — Python.NET cannot find the Python library. Install the matching Python version, or set `Runtime.PythonDLL` to the full path of your `pythonXY.dll`.
- **`ModuleNotFoundError`** — install the packages listed above into the same Python installation that the DLL belongs to.
- **Port 8000 already in use** — stop the other process or change the port in `ApiCode.cs`.

The trainer writes `iris_nb.pkl` relative to the current working directory, while `IrisCode` looks for it in `AppDomain.CurrentDomain.BaseDirectory`. With dotnet run from the source folder these locations usually differ, so first-run training can succeed but loading can fail before FastAPI starts. Build first, then run the DLL from its output directory to align the paths, or review a code fix separately. Each scenario initializes and shuts down Python; repeated interpreter initialization and native package compatibility were not runtime-tested. Only load trusted joblib/pickle models.

## 🔗 Related documentation

### Related Examples

- [The reverse direction: call a C# library from Python](https://github.com/gcfernando/python_codes/blob/main/LoadLibraries/readme.md)
- [Back to all C# and .NET code examples](../README.md)

---

### Documentation links

[Read all C# and .NET Code Examples guides online](https://gcfernando.github.io/csharp_codes/documentation/) · [Project overview](https://gcfernando.github.io/csharp_codes/) · [All developer projects](https://gcfernando.github.io/gcfernando/projects/)
