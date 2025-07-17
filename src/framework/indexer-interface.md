# Indexer Interface

Dojo enables efficient on-chain data storage with indexing through Torii. The Dojo standard defines tables using models.

A proposed structure of the events emitted to the indexer:

```rust
pub enum DatabaseEvents {
    DeclareTable: DeclareTable,
    DeclareTableWithSchema: DeclareTableWithSchema,
    DeclareFields: DeclareFields,
    DeclareField: DeclareField,
    DeclareSchema: DeclareSchema,
    UpdateValue: UpdateValue,
    UpdateRecordFields: UpdateRecordFields,
    UpdateFieldRecords: UpdateFieldRecords,
    UpdateRecordsFields: UpdateRecordsFields,
    UpdateRecordFromSchema: UpdateRecordFromSchema,
    UpdateRecordsFromSchema: UpdateRecordsFromSchema,
}

/// Emitted when a new table is declared with inline field definitions.
///
/// Fields:
/// - `id`: Unique identifier of the table (e.g., hash of the name or a custom ID).
/// - `name`: Human-readable name of the table.
/// - `fields`: List of field layouts (name + layout) associated with this table.

pub struct DeclareTable {
    #[key]
    pub id: felt252,
    pub name: ByteArray,
    pub fields: Span<FieldLayout>,
}

/// Emitted when a table is undeclared.
///
/// Fields:
/// - `id`: Unique identifier of the table (e.g., hash of the name or a custom ID).

pub struct UndeclareTable {
    #[key]
    pub id: felt252,
}

/// Declares a table using a pre-defined schema.
///
/// Fields:
/// - `id`: Unique identifier for the table.
/// - `name`: Human-readable name of the table.
/// - `schema`: The schema declared via `DeclareSchema`.

pub struct DeclareTableWithSchema {
    #[key]
    pub id: felt252,
    pub name: ByteArray,
    pub schema: felt252,
}

/// Declares multiple fields for an existing table.
///
/// Fields:
/// - `table`: Table ID the fields belong to.
/// - `fields`: Array of field definitions (name + layout).

pub struct DeclareTableFields {
    #[key]
    pub table: felt252,
    pub fields: Span<FieldLayout>,
}

/// Declares a single field within a given table.
///
/// Fields:
/// - `table`: Table ID this field belongs to.
/// - `selector`: Field selector (usually feltified name).
/// - `layout`: Layout of the field (e.g., fixed, array, struct).
pub struct DeclareTableField {
    #[key]
    pub table: felt252,
    #[key]
    pub selector: felt252,
    pub layout: Layout,
}


/// Declares a reusable schema layout.
///
/// Fields:
/// - `id`: Deterministic schema ID (e.g., hash of fields).
/// - `fields`: Field layouts (selector + layout), without names.

pub struct DeclareTableSchema {
    #[key]
    pub id: felt252,
    pub fields: Span<FieldLayout>,
}

/// Updates a single field for a single record.
///
/// Fields:
/// - `table`: Table ID.
/// - `record`: Record/entity ID.
/// - `field`: Field selector.
/// - `value`: Value to set.

pub struct UpdateValue {
    #[key]
    pub table: felt252,
    #[key]
    pub record: felt252,
    #[key]
    pub field: felt252,
    pub value: Span<felt252>,
}


/// Updates multiple fields for a single record.
///
/// Fields:
/// - `table`: Table ID.
/// - `record`: Record ID.
/// - `fields`: List of field selectors.
/// - `values`: Concatenated field values (decoded per layout).

pub struct UpdateRecordFields {
    #[key]
    pub table: felt252,
    #[key]
    pub record: felt252,
    pub fields: Span<felt252>,
    pub values: Span<felt252>,
}


/// Updates a single field across multiple records.
///
/// Fields:
/// - `table`: Table ID.
/// - `field`: Field selector.
/// - `records`: List of record IDs.
/// - `values`: Corresponding values for each record.

pub struct UpdateFieldRecords {
    #[key]
    pub table: felt252,
    #[key]
    pub field: felt252,
    pub records: Span<felt252>,
    pub values: Span<felt252>,
}

/// Updates multiple fields across multiple records (row-major order).
///
/// Fields:
/// - `table`: Table ID.
/// - `records`: List of record IDs (rows).
/// - `fields`: List of field selectors (columns).
/// - `data`: Flattened data buffer [record][field] values.

pub struct UpdateRecordsFields {
    #[key]
    pub table: felt252,
    pub records: Span<felt252>,
    pub fields: Span<felt252>,
    pub data: Span<felt252>,
}


/// Updates a record using a predefined schema layout.
///
/// Fields:
/// - `table`: Table ID.
/// - `record`: Record ID.
/// - `schema`: Schema ID (defines field order).
/// - `data`: Field values matching the schema.

pub struct UpdateRecordFromSchema {
    #[key]
    pub table: felt252,
    #[key]
    pub record: felt252,
    #[key]
    pub schema: felt252,
    pub data: Span<felt252>,
}

/// Updates multiple records using a predefined schema layout.
///
/// Fields:
/// - `table`: Table ID.
/// - `schema`: Schema ID.
/// - `records`: List of record IDs.
/// - `data`: Flattened record values following the schema order.

pub struct UpdateRecordsFromSchema {
    #[key]
    pub table: felt252,
    #[key]
    pub schema: felt252,
    pub records: Span<felt252>,
    pub data: Span<felt252>,
}
```

This structure is designed to mimic database operations and be as generic as possible, while still being efficient in gas usage and keeping with databasing convention.
