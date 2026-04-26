# shovel.rs: Technical & Project Specification
## The Bilateral Data Catalog & Orchestrator

### 1. The Problem Statement: Why Shovel?
In the current data landscape, we suffer from **"The Opaque Pipeline Problem."** Traditional orchestration treats data as a passive passenger. We know if a task *finished*, but we rarely know if the data it produced is actually *correct* until a dashboard breaks or a model starts hallucinating.

* **Silent Failures:** Most pipelines fail "successfully." The script exits with Code 0, but the data is garbage.
* **Context Loss:** By the time data reaches the Data Scientist, the engineering context (lineage, original schema, statistical baseline) is lost.
* **The Runtime Tax:** High-level orchestrators often introduce massive overhead for small ETL tasks, while low-level tools lack the observability required for complex data governance.

---

### 2. Market Comparison: A Shift in Philosophy

| Feature | Traditional Orchestrators (e.g., Airflow) | Modern Observability (e.g., Great Expectations) | **shovel.rs** (Bilateral Engine) |
| :--- | :--- | :--- | :--- |
| **Focus** | **Task-Centric:** "Did the script run?" | **Post-Hoc:** "Is the data I already moved bad?" | **Data-Centric:** "Should this task even start?" |
| **Safety** | Success/Failure of the process. | External testing suites. | **In-Process Contracts:** Native validation. |
| **Performance** | High overhead (JVM/Heavy Python). | Heavy serialization/deserialization. | **Zero-Copy:** Rust + Arrow memory sharing. |
| **Governance** | Manual metadata tracking. | Separate catalog services. | **Integrated Ledger:** Versioned metadata. |

---

### 3. Key Concepts: The Silent Killers

#### 3.1. Column Mismatch (Structural Integrity)
A column mismatch occurs when the physical structure of the data deviates from the registered contract. 
* **The Issue:** A source database changes `user_id` (Integer) to `user_uuid` (String), or a column is dropped entirely.
* **Shovel’s Defense:** Before any task executes, Shovel performs a "Structural Sift." If the incoming schema does not perfectly align with the registered "Plot" schema, the orchestrator halts execution, preventing downstream corruption.

#### 3.2. Data Drift (Statistical Integrity)
Data Drift is a silent failure where the **schema is correct**, but the **meaning of the data** has changed.
* **The Issue:** A "Unit Price" column usually averages $50. Suddenly, due to a bug upstream, the average jumps to $5,000. The schema is still "Float," but the data is wrong.
* **Shovel’s Defense:** Shovel maintains a **Golden Baseline** (Stat Sheet). During the "Grade" phase, it performs a statistical "Swing Test." If the mean, variance, or null-count drifts beyond a software-defined threshold, the system flags it immediately.

---

### 4. The 4-Step Dataset Lifecycle

1.  **Register:** Define the location (URI/Path) of the data. This creates the initial metadata entry in the internal ledger.
2.  **Profile:** The embedded DuckDB engine surveys the data to build the "Golden Baseline" (Schema + Statistics).
3.  **Validate:** The system enforces the **Data Contract**. Every move is checked for Column Mismatches and Data Drift.
4.  **Trace:** Every transformation is recorded in a graph-based lineage map, showing exactly how "Dataset A" influenced "Dataset B."

---

### 5. Core Components

#### 5.1. The Metadata Catalog (The Brain)
* **Versioned Ledger:** Uses a Git-like architecture for metadata. You can "Fork" a metadata state to test new schemas without affecting production.
* **Bilateral Access:** High-performance Rust handles the metadata storage; Python users interact with it via an idiomatic API with zero-copy overhead.

#### 5.2. Analytical Engine (Embedded DuckDB)
* **Local OLAP:** No need for an external warehouse to check data quality. DuckDB crunches the numbers locally using vectorized kernels.
* **Geospatial Native:** Native support for the complex geometries found in GeoParquet and GeoJSON.

#### 5.3. Orchestrator (The Supervisor)
* **Contract Enforcement:** Tasks are not just scripts; they are "Contractual Obligations." If the input doesn't meet the contract, the task is denied execution.
* **Minimalist Design:** Optimized for speed and low-latency scheduling.

---

### 6. Technical Stack & Structure

* **Core:** Rust (Safety & Speed)
* **FFI:** PyO3 (The Python Bridge)
* **Memory:** Apache Arrow (Shared memory between Rust/Python)
* **Engine:** DuckDB (In-process SQL & Stats)
* **Storage:** `.shovel.db` (The persistent metadata store)

#### Project Layout
```text
shovel-rs/
├── data_catalog/          # Core Library (Rust Crate)
│   ├── src/               # Logic for Register, Profile, Validate
│   ├── registry/          # DuckDB persistence layer
│   └── python/            # Python-accessible module
├── orchestrator/          # The Execution Engine (Binary)
│   └── src/main.rs        # CLI & Task Scheduling
└── common/                # Shared utilities and Error types
