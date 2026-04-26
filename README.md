# shovel.rs: Technical & Project Specification
## The Unified Data Catalog & Orchestrator

### 1. Executive Summary
**shovel.rs** is a high-performance data engineering platform designed to bridge the gap between low-level systems architecture and high-level data science. It provides a **bilateral interface**: a high-speed **Rust** core for execution and safety, and a native **Python** heart for developer accessibility. 

By treating metadata as a versioned, statistical ledger, shovel.rs ensures that data is not just moved, but is validated, monitored for quality (drift), and governed by strict software-defined contracts.

---

### 2. The Dataset Lifecycle (Core Philosophy)
To ensure the system is understandable for both engineers and stakeholders, every dataset follows a standard four-stage lifecycle:

1.  **Register:** Define the physical boundaries. Identify the URI/Path where the data lives (S3, Local, Database).
2.  **Profile:** Survey the data. Perform automated inference of both structure (Schema) and health (Statistical distributions).
3.  **Validate:** Build to code. Enforce contracts by comparing incoming data against established "Golden" baselines.
4.  **Trace:** Document the history. Maintain an immutable, graph-based record of every transformation and dependency (Lineage).

---

### 3. Core Components

#### 3.1. The Metadata Catalog (The Brain)
The Catalog is the primary rulebook that the entire orchestrator follows. It moves beyond a simple list of files to become a living blueprint of the data ecosystem.
* **Column-Level Governance:** Tracks strict schemas, data types, and variants.
* **The "Golden" Baseline:** Upon registration, the system generates a statistical snapshot (Mean, StdDev, Null-counts). This acts as the "source of truth" for future monitoring.
* **Bilateral Bridge:** Uses **PyO3** and **Apache Arrow** to expose Rust-managed metadata to Python with zero-copy overhead, ensuring data scientists can access engineering-grade metadata instantly.
* **Lineage & Influence:** Traces upstream dependencies to see which columns or features influence the final dataset, allowing for comprehensive impact analysis.

#### 3.2. Analytical Engine (In-Process DuckDB)
Shovel embeds **DuckDB** directly into the binary to handle heavy analytical processing locally and efficiently.
* **Vectorized Calculations:** Uses DuckDB’s OLAP power to crunch millions of rows locally to generate the "Stat Sheets" required for drift monitoring.
* **Quality Grading:** A built-in service that compares incoming data against the Golden Baseline. It calculates statistical "Swings" to determine if data has deviated too far from the norm.
* **Geospatial Native:** Specifically designed to handle geospatial formats like GeoParquet, GeoJSON, and FlatGeobuf.

#### 3.3. The Orchestrator (The Supervisor)
The Orchestrator server manages task execution through **Data Contracts**. It doesn't just run scripts; it guards the data flow.
* **Contract-Defined Tasks:** Every task specifies exactly what data it expects. The orchestrator verifies these "contracts" before execution.
* **Fail or Warn:** If a dataset drifts or columns are mismatched, the supervisor can halt the pipeline (Fail) to prevent corruption or flag it for review (Warn).
* **Minimalist Execution:** Follows a "Lean" philosophy—prioritizing low overhead and high concurrency.

---

### 4. Technical Stack
* **Language:** Rust (Stable) for the core execution, memory safety, and speed.
* **FFI:** PyO3 for seamless Python bindings.
* **Storage Format:** Apache Arrow for zero-copy memory transit between Rust and Python.
* **Internal Database:** DuckDB for in-process metadata querying, SQL transactions, and statistical crunching.
* **Persistence:** A local `.shovel.db` (DuckDB-backed) for the registry and lineage ledger.

---

### 5. Implementation Roadmap

#### Phase 1: Structural Foundation
* Implement the **Registry** to persist data URIs in DuckDB.
* Develop **Automated Schema Inference** for Parquet, CSV, and GeoJSON.
* Establish the **Bilateral Bridge** (PyO3 + Arrow) to allow Python to read Rust metadata.

#### Phase 2: Observability & Health
* Launch **Statistical Profiling**: Automated generation of "Stat Sheets" using DuckDB.
* Enable **Drift Detection**: The first iteration of "Swing Tests" to monitor statistical health.
* Support for **Variants**: Managing different logical views (e.g., Bronze, Silver, Gold) for a single dataset.

#### Phase 3: The Versioned Ledger
* **Git-like Metadata:** Implement parent-child versioning for all registry entries.
* **Forks & Publishing:** Allow users to create metadata branches for experimentation and merge them back once validated.
* **Instant Rollbacks:** Recover state by reassigning variant pointers in the ledger.

#### Phase 4: Full Ecosystem Mapping
* **Column-Level Lineage:** Detailed mapping of which source features influence which output features.
* **Cloud Integration:** Native support for S3/GCS/Azure Blob storage as registered URIs.
* **Drift Dashboards:** Visual reporting of data health over time.

---

### 6. Project Structure
```text
shovel-rs/
├── Cargo.toml                # Workspace configuration
├── README.md                 # Project overview
├── shovel/                   # THE CATALOG (Crate: Library)
│   ├── src/                  
│   │   ├── lib.rs            # Rust entry point
│   │   ├── core/             # Register, Profile, Validate logic
│   │   └── registry/         # DuckDB interaction layer
│   └── python/               
│       └── lib.rs            # PyO3 module definitions
├── orchestrator/             # THE EXECUTION ENGINE (Crate: Binary)
│   └── src/                  
│       └── main.rs           # CLI / Server entry point
└── common/                   # SHARED UTILITIES
    └── src/
        └── lib.rs            # Shared Enums (Geotypes, Error types)
