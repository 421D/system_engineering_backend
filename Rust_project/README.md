# 📊 RSheet – Concurrent, Dependency-Aware Spreadsheet Engine (Rust)

A high-performance spreadsheet backend implemented in **Rust**, supporting **multi-threaded updates**, **dependency tracking**, **expression parsing**, and **version-based conflict prevention**.  
Designed as a robust systems-engineering project demonstrating strong concurrency, dataflow, and backend logic skills.

---

## 🚀 Overview

RSheet functions as a real spreadsheet **compute engine**, capable of:

- Evaluating expressions like `A1 + B2 * 5`
- Parsing ranges into vectors/matrices (`A1_A5`, `B2_D4`)
- Tracking dependencies between cells
- Safely propagating updates across dependent cells
- Running concurrent background workers
- Preventing outdated writes from overriding newer values using version numbers
- Handling GET/SET commands over terminal or network

Built in pure Rust using standard libraries:

- `Arc`, `Mutex`, `Condvar`
- `mpsc` channels
- `HashMap`, `HashSet`

---

## 🔑 Key Features

### ✔ 1. Full Expression Parsing
Supports:

- Single cell refs: `A1`, `B7`
- Ranges:
  - Column vectors: `A1_A7`
  - Row vectors: `C3_F3`
  - Matrices: `B2_D10`
- Scalars, vectors, and matrices are converted into:
  - `CellArgument::Value`
  - `CellArgument::Vector`
  - `CellArgument::Matrix`

Expressions are evaluated through the `CellExpr` API.

---

### ✔ 2. Dependency Graph With Auto-Propagation
Maintains two synchronized maps:

| Map | Meaning |
|-----|---------|
| `dependents` | Which cells depend on me |
| `uses_sources` | Which cells I depend on |

Whenever `SET` happens:

1. Old dependencies are removed  
2. New dependencies are inserted  
3. All dependents are queued for re-evaluation  
4. Worker recomputes them in the correct order

A classic DAG-based spreadsheet architecture.

---

### ✔ 3. Multi-threaded Worker With Safe Concurrency

A worker thread receives `(cell_key, seq)` tasks through:

```rust
mpsc::channel::<(String, u64)>
```
It performs:

Expression evaluation

Error propagation

Version-checking

Dependency scheduling

Uses Rust concurrency primitives:
```
Arc<Mutex<Shared>>
Mutex<WorkerState>
Condvar
```
Ensures no data races and correct synchronization.

✔ 4. Version-Based Write Protection (Anti-race Condition)

Every cell has a version counter:
```
versions: HashMap<String, u64>
```

