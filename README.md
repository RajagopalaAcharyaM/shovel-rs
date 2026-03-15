Shovel (shovel.rs)

The Bilateral Geospatial Data Catalog & Orchestrator.

Shovel is a high-performance, lean data engine built in Rust with a native Python heart. It is designed to bridge the gap between low-level data engineering and high-level data science by treating metadata as a versioned, statistical ledger.
🏗️ Core Philosophy

Shovel operates on the principle of The Plot. In this system, a dataset is a physical "Plot" of land that must be surveyed and maintained.

    Stake (Register): Define the URI/Path boundaries of the data.

    Sift / Clarify (Discovery): Vectorized inference of both structure (Schema) and health (Statistics).

    Grade (Validation): Enforce contracts by comparing incoming data against established baselines.

    Trench (Lineage): An immutable, graph-based record of every transformation and move.

🚀 Phased Implementation Strategy
Phase 1: The Foundation (Structural)

    Registry: Persistent storage of data URIs in an internal DuckDB instance.

    Structural Sifting: Automated inference of column names and data types (Parquet, CSV, GeoJSON).

    Bilateral Bridge: Using PyO3 and Apache Arrow to expose Rust structs to Python with zero-copy overhead.

Phase 2: The Infrastructure (Observability)

    Statistical Profiling: Automated calculation of "Stat Sheets" (Mean, StdDev, Null-counts, Quantiles) using DuckDB’s vectorized kernels.

    Metadata Variants: Support for multiple logical views (e.g., Bronze, Silver, Gold) for a single physical Stake.

    Drift Detection: Automated "Swing Tests" that trigger warnings when new data statistics deviate from the baseline variant.

Phase 3: The Ledger (Versioning)

    Git-like Metadata: Parent-child versioning for all metadata entries.

    Forks & Publishing: Create isolated metadata branches (Forks) for experimentation and Publish them to the Master pointer once graded.

    Rollbacks: Instant state recovery by reassigning variant pointers in the ledger.

🛠️ Technical Stack

    Language: Rust (Stable) for the core execution and safety.

    FFI: PyO3 for Python bindings.

    Engine: DuckDB for in-process OLAP, SQL-based metadata querying, and geospatial processing.

    Memory Format: Apache Arrow for shared-memory transit between Rust and Python.

    Persistence: Internal .shovel.db (DuckDB) for registry and lineage.

📂 Project Structure
Plaintext

```shoveller/
├── Cargo.toml                # Workspace configuration
├── README.md                 # Project overview
├── TECH_SPEC.md              # Detailed technical requirements
├── shovel/                   # THE CATALOG (Crate: Library)
│   ├── Cargo.toml
│   ├── src/                  
│   │   ├── lib.rs            # Rust entry point
│   │   ├── core/             # Stake, Sift, Grade logic
│   │   └── registry/         # DuckDB interaction layer
│   └── python/               
│       └── lib.rs            # PyO3 module definitions
├── excavator/                # THE ORCHESTRATOR (Crate: Binary)
│   ├── Cargo.toml
│   └── src/
│       └── main.rs           # CLI / Execution entry point
└── common/                   # SHARED UTILS
    └── src/
        └── lib.rs            # Shared Enums (Geotypes, Error types)```

⌨️ Development
Prerequisites

    Rust (1.75+)

    Python (3.9+)

    maturin (for building Python bindings)

Building the Workspace
Bash

# Build all crates in the workspace
cargo build --release

# Build and install the Python bindings locally
cd shovel && maturin develop

Running the Orchestrator
Bash

cargo run -p excavator -- --help

📜 License

Personal Project - Industrial Grade Engineering.

Would you like me to generate the common/src/lib.rs file next? This will define the core Enums (like DataType and StatValue) that both shovel and excavator will need to share.