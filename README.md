# shovel-rs
A high-performance, lean Data Catalog and Orchestrator built in Rust with a bilateral Python interface.
Technical Specification: Shovel (shovel.rs)

Project Goal: A high-performance, lean Data Catalog and Orchestrator built in Rust with a bilateral Python interface.
1. Core Features & Functional Requirements
1.1 Automated Data Asset Registry (Stake)

    Feature: DataAsset::new(path)

    Description: Shovel must automatically initialize a data asset when a path (Local, S3, or Database) is provided.

    Requirement: The system must not require the user to manually define a schema. It must perform lazy discovery to determine the file type (Parquet, CSV, GeoJSON, etc.).

1.2 Automated Schema Inference (Sift)

    Feature: Bilateral Schema Discovery.

    Description: Shovel "Sifts" the raw data to extract column names, data types, and geospatial metadata (SRID, Geometry Type).

    Requirement: Inference must be performed in the Rust core for speed, then exposed to Python as a typed object.

1.3 Versioned Metadata Ledger (Git-like Branching)

    Feature: Persistent Metadata Variants.

    Description: Instead of a single metadata column, Shovel maintains a dedicated metadata_variants table in an internal DuckDB instance.

    Capabilities:

        Checkpoints: Immutable snapshots of a schema at a specific time.

        Forks (Branches): Users can create an isolated metadata variant (e.g., "Experimental") to test schema changes without affecting the "Master" production line.

        Publishing (Merge): Promoting a verified variant to the "Master" status.

        Rollback: Instantly reverting the "Master" pointer to a previous Checkpoint ID.

1.4 Data Contract Enforcement (Grade)

    Feature: "Shovel-Ready" Validation.

    Description: Before the orchestrator executes a task, it "Grades" the incoming data against the active Metadata Variant.

    Requirement: If the incoming data drift (e.g., a column changes from Int to String), Shovel must halt the pipeline and report the "Off-Grade" mismatch.

1.5 Immutable Lineage (Trench)

    Feature: Structural History.

    Description: Every move, transformation, or schema change is recorded in the "Trench."

    Requirement: This must be a Directed Acyclic Graph (DAG) stored in the internal DB, allowing users to trace the origin and evolution of any asset.

2. Technical Architecture
2.1 Storage Strategy (The "Internal Brain")

    Database: Private DuckDB instance (.shovel.db) managed exclusively by the tool.

    Schema Storage: High-complexity column metadata is stored as JSON blobs within the relational tables to allow for flexible geospatial attributes (Statistics, Bounding Boxes, SRIDs) without constant SQL migrations.

2.2 Bilateral Execution Model

    Core: Rust (for performance-critical Sifting, Grading, and Orchestration).

    Bridge: PyO3 + Apache Arrow. Data is traded between Rust and Python via memory pointers to ensure zero-copy overhead.

    Nomenclature: The API supports dual-naming (e.g., sift() and clarify()) to provide both a unique brand identity and industry-standard familiarity.

3. Rationale for Versioned Metadata

Why not a static catalog?
In modern data engineering, "Data Drift" is the primary cause of pipeline failure. By treating metadata like source code (with branching and merging), Shovel allows for:

    Zero-Downtime Schema Updates: Prepare the new metadata variant before the new data arrives.

    Safe Multi-user Collaboration: Different team members can "Sift" and "Fork" the same asset without collisions.

    Recovery: Metadata is no longer a destructive update; it is a permanent ledger.

4. Implementation Constraints

    Language: Rust (Stable).

    Interactions: Python 3.9+.

    Data Engine: DuckDB (Storage and compute).

    Spatial Support: PostGIS-compatible (WKB/WKT).


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

