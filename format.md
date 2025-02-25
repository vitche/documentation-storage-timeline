## Storage.Timeline Format Documentation

This document describes the internal format of the Storage.Timeline database system, including both its hierarchical folder structure and the binary format used to store time-series data.

### Hierarchical Database Structure

Storage.Timeline implements a three-level hierarchical structure that organizes time-series data:

```
storage_root/
├── schema_1/
│   ├── timeline_1
│   ├── timeline_2
│   └── ...
├── schema_2/
│   ├── timeline_3
│   ├── timeline_4
│   └── ...
└── ...
```

#### Level 1: Storage

The top level of the hierarchy is the **Storage** directory, which serves as the root container for all data. This directory is specified when initializing a Storage instance:

```javascript
const storage = new Storage('/path/to/storage_root');
```

#### Level 2: Schema

Each **Schema** is represented as a subdirectory within the Storage directory. Schemas function as logical groupings of related timelines (similar to tables in a relational database):

```javascript
storage.create('schema_name', callback);
```

#### Level 3: Timeline

Each **Timeline** is represented as a file within a Schema directory. A timeline contains a sequence of time-ordered events, stored in binary format:

```javascript
schema.create('timeline_name', callback);
```

### Binary Format of Timeline Files

Timeline files store time-series data in a binary format optimized for sequential reading. Each record in a timeline follows this structure:

#### Record Structure

Each record consists of:

1. **Payload Size** (8 bytes, Int64): Total size in bytes of the following data (timestamp + value)
2. **Timestamp** (8 bytes, Int64): Unix timestamp in milliseconds
3. **Value** (variable length): The actual data payload

```
┌─────────────┬────────────┬───────────────────┐
│ Payload Size│  Timestamp │       Value       │
│   (8 bytes) │  (8 bytes) │ (variable length) │
└─────────────┴────────────┴───────────────────┘
```

#### Data Type Encoding

The system supports several data types, each encoded differently:

##### String Values
```
┌─────────────┬────────────┬───────────────────┐
│ 8 + str.len │  Timestamp │  UTF-8 String     │
│   (8 bytes) │  (8 bytes) │ (variable length) │
└─────────────┴────────────┴───────────────────┘
```

##### Number Values
```
┌─────────────┬────────────┬───────────┐
│      16     │  Timestamp │   Value   │
│   (8 bytes) │  (8 bytes) │ (8 bytes) │
└─────────────┴────────────┴───────────┘
```

##### Buffer Values
```
┌─────────────┬────────────┬───────────────────┐
│ 8 + buf.len │  Timestamp │    Buffer Data    │
│   (8 bytes) │  (8 bytes) │ (variable length) │
└─────────────┴────────────┴───────────────────┘
```

##### Object/Array Values
Objects and arrays are first converted to JSON strings, then stored using the string format.

#### Reading Strategy

The Timeline's reading mechanism uses a stateful cursor approach:

1. The timeline file is read into a buffer on the first `next()` call
2. An internal offset pointer tracks the current position in the buffer
3. Each `next()`, `nextString()`, or `nextNumber()` call advances this pointer
4. The `reset()` method resets the pointer to the beginning of the file

#### Example Record

For a numeric value `42` recorded at timestamp `1634567890123`:

```
┌─────────────┬────────────┬───────────┐
│      16     │ 1634567890 │     42    │
│             │     123    │           │
└─────────────┴────────────┴───────────┘
```

### File Manipulation Operations

The system provides several file-level operations for timelines:

- **copy**: Creates a complete duplicate of a timeline file
- **concatenate**: Appends the contents of one timeline to another
- **rename**: Changes the name of a timeline file
- **truncate**: Empties a timeline file but keeps it in place
- **unlink**: Completely removes a timeline file

These operations allow for flexible management of time-series data without having to manipulate the binary format directly.

