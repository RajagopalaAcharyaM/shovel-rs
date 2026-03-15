# shovel-rs
A high-performance, lean Data Catalog and Orchestrator built in Rust with a bilateral Python interface.

Technical Specification: Shovel (shovel.rs)

Project Goal: A high-performance, lean Data Catalog and Orchestrator.

Core Unit: The Plot (The Data Asset).
1. Core Philosophy: The "Plot" Concept

In Shovel, we do not manage "datasets"; we manage Plots. A Plot represents a physical location of data (S3, Local, DB) that must be surveyed, prepared, and maintained.

    Why: This shifts the mindset from abstract "data management" to the physical reality of Data Engineering.

2. Phased Development Strategy

To ensure "Industrial Grade" stability while maintaining a "Lean" velocity, development is partitioned into three symbiotic phases.
Phase 1: The Foundation (Groundbreaking)

Features: Stake, Sift (Schema only), and Grade (Schema only).

    The "Why": Establishing the "Bilateral" bridge between Rust and Python is the highest technical hurdle. This phase ensures the basic registry and DuckDB-backed schema discovery are bulletproof before adding complexity.

Phase 2: The Infrastructure (The Survey)

Features: Variants, Observability, and Statistical Sifting.

    The "Why": A schema alone is a static snapshot. Real-world data is dynamic. Adding "Statistical Sheets" (Min, Max, Mean, Null-counts) per variant allows the tool to detect Data Drift.

    Implementation: DuckDB runs vectorized aggregation queries during the "Sift" phase to generate a Baseline.

Phase 3: The Ledger (The Trench)

Features: Git-like Versioning (Forks, Checkpoints, and Rollbacks).

    The "Why": Versioning metadata prevents destructive updates. By treating metadata like source code, multiple users can experiment on "Forks" of a Plot without breaking the production pipeline's "Master" variant.

3. Technical Architecture: The "DuckDB-Arrow" Engine
3.1 The Compute Engine: DuckDB

Despite being a personal project, Shovel utilizes DuckDB as its internal compute kernel.

    Respectable Performance: DuckDB's vectorized execution engine provides world-class performance on a single node, often outperforming "Enterprise" cloud warehouses for datasets under 1TB.

    Spatial Superiority: Native support for geospatial types (WKB/WKT) and spatial indexing allows Shovel to "Sift" complex geometries with zero-copy overhead.

3.2 The Exchange Format: Apache Arrow

    Bilateral Harmony: All data transit between the Rust engine and the Python API occurs via Apache Arrow.

    Zero-Copy: By sharing memory pointers, Shovel eliminates the "serialization tax," allowing a Python Geopandas user to consume Rust-processed data at native speed.

4. Feature Set Deep-Dive
Feature	Thematic Name	Technical Action	Why?
Registration	Stake	Declare path in the DuckDB registry.	To define the boundary of the Plot.
Inference	Sift	Map types + Calculate statistics.	To understand the "soil quality" (data health).
Validation	Grade	Compare incoming data vs. Baseline.	To prevent "Off-Grade" (Drift) failures.
Branching	Fork	Create a child-variant of metadata.	To allow safe schema experimentation.
Promotion	Publish	Update the Master pointer to a new ID.	To move from "Draft" to Production.
5. Storage Schema (Internal DuckDB)

The internal brain of Shovel consists of two primary tables:

    registry_plots: Stores the location and identity of every staked asset.

    metadata_ledger: A parent-child versioned table storing JSON blobs of schemas and statistical sheets.

shoveller/
├── Cargo.toml                # 
├── README.md                 # Project vision
├── TECH_SPEC.md              # The file we just wrote
├── shovel/                   # THE CATALOG (Crate 1: Library)
│   ├── Cargo.toml            # Shovel-specific dependencies (DuckDB, Serde)
│   ├── src/
│   │   ├── lib.rs            # Entry point for the Rust logic
│   │   ├── core/             # Internal logic (Sift, Stake, Grade)
│   │   └── registry/         # DuckDB interaction layer
│   └── python/               # PyO3 bridge (Bilateral layer)
│       └── lib.rs            # The #[pymodule] definition
├── excavator/                # THE ORCHESTRATOR (Crate 2: Binary)
│   ├── Cargo.toml            # Depends on ../shovel
│   └── src/
│       ├── main.rs           # The CLI entry point
│       └── engine/           # Execution & Lineage logic
└── common/                   # SHARED CODE (Optional but recommended)
    ├── Cargo.toml
    └── src/
        └── lib.rs            # Shared Enums (Geotypes, Error types)

