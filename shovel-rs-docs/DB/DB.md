-- Enable UUID extension for dataset_id generation
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- ==========================================
-- 1. BASE DATASET TABLE
-- ==========================================
CREATE TABLE datasets (
    id SERIAL PRIMARY KEY,
    dataset_id UUID NOT NULL UNIQUE DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

-- ==========================================
-- 2. METADATA HISTORY (Immutable Log)
-- ==========================================
CREATE TABLE metadata_history (
    id SERIAL PRIMARY KEY,
    dataset_id INTEGER NOT NULL REFERENCES datasets(id) ON DELETE CASCADE,
    metadata JSONB NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    archived_at TIMESTAMP WITH TIME ZONE
);

-- Index to fast-query the active metadata state
CREATE INDEX idx_metadata_active 
ON metadata_history(dataset_id) 
WHERE archived_at IS NULL;


-- ==========================================
-- 3. DATASET VERSION (The Commit Tree)
-- ==========================================
CREATE TABLE dataset_versions (
    id SERIAL PRIMARY KEY,
    dataset_id INTEGER NOT NULL REFERENCES datasets(id) ON DELETE CASCADE,
    version_number INTEGER NOT NULL,
    parent_version_id INTEGER REFERENCES dataset_versions(id) ON DELETE SET NULL,
    commit_hash VARCHAR(64) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    
    -- Ensures version numbers are sequential/unique per dataset entity
    CONSTRAINT unique_dataset_version UNIQUE (dataset_id, version_number)
);

-- Index for graph traversal via parent pointers
CREATE INDEX idx_dataset_versions_parent ON dataset_versions(parent_version_id);


-- ==========================================
-- 4. COLUMNS TABLE (Flat Schema Design)
-- ==========================================
CREATE TABLE columns (
    id SERIAL PRIMARY KEY,
    dataset_version_id INTEGER NOT NULL REFERENCES dataset_versions(id) ON DELETE CASCADE,
    column_name VARCHAR(255) NOT NULL,
    data_type VARCHAR(100) NOT NULL,
    is_nullable BOOLEAN NOT NULL DEFAULT TRUE,
    ordinal_position INTEGER NOT NULL,
    description TEXT,
    
    -- Avoid duplicate column definitions within the same schema commit
    CONSTRAINT unique_version_column_name UNIQUE (dataset_version_id, column_name)
);

-- Index for fetching schemas instantly when scanning a version
CREATE INDEX idx_columns_version_id ON columns(dataset_version_id);


-- ==========================================
-- 5. STORAGE REGISTRY (Append-Only Data Paths)
-- ==========================================
CREATE TABLE dataset_storage (
    id SERIAL PRIMARY KEY,
    dataset_version_id INTEGER NOT NULL REFERENCES dataset_versions(id) ON DELETE CASCADE,
    storage_type VARCHAR(50) NOT NULL, -- 's3', 'local', etc.
    file_format VARCHAR(50) NOT NULL,  -- 'geoparquet', 'csv', etc.
    path TEXT NOT NULL,
    written_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL
);

-- Index for gathering all files belonging to a specific data contract version
CREATE INDEX idx_storage_version_id ON dataset_storage(dataset_version_id);


-- ==========================================
-- 6. BRANCH POINTER TABLE (Git HEADs)
-- ==========================================
CREATE TABLE dataset_branches (
    id SERIAL PRIMARY KEY,
    dataset_id INTEGER NOT NULL REFERENCES datasets(id) ON DELETE CASCADE,
    branch_name VARCHAR(100) NOT NULL,
    current_version_id INTEGER NOT NULL REFERENCES dataset_versions(id),
    is_master BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    
    -- Branch names must be unique within a single dataset ecosystem
    CONSTRAINT unique_dataset_branch_name UNIQUE (dataset_id, branch_name)
);

-- Index to quickly pull branch tracking definitions
CREATE INDEX idx_branches_dataset_id ON dataset_branches(dataset_id);