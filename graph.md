## `Storage.Timeline.Graph` Format Documentation

This document describes the Graph Timeline format - an inner layer of the Storage.Timeline database system that stores time-bound graph data.

### Graph Data Structure Overview

Within the Timeline layer, graph data is stored as a series of JSON encoded objects, each representing an edge in a graph at a specific point in time. This approach creates a temporal graph where the structure can evolve over time.

#### Conceptual Representation

```
storage_root/
├── schema_1/
│   ├── timeline_1  <-- Contains graph edges as JSON objects
│   │                   Each object bound to a timestamp
```

### Edge-Encoded Graph Format

Each entry in a timeline represents a single edge in the graph, encoded as a JSON object and bound to a specific timestamp. This edge-centric representation allows for efficient storage and retrieval of evolving graph structures.

#### Edge Object Structure

Each edge is stored as a JSON object with the following required fields:

```json
{
  "s": "sourceNodeId",  // Source node identifier
  "t": "targetNodeId",  // Target node identifier
  "w": 1.5             // Weight of the edge (numeric value)
}
```

| Field | Description                                                                                                                 |
|-------|-----------------------------------------------------------------------------------------------------------------------------|
| `s`   | Source node identifier. Can be a string or numeric ID.                                                                      |
| `t`   | Target node identifier. Can be a string or numeric ID.                                                                      |
| `w`   | Weight of the edge. Numeric value representing the strength, distance, or other quantifiable property of the relationship.  |

#### Binary Storage Format

The JSON edge object is stored in the Timeline's binary format as a string value:

```
┌─────────────┬────────────┬───────────────────────────────┐
│ Payload Size│  Timestamp │ JSON Edge Object as UTF-8     │
│   (8 bytes) │  (8 bytes) │ {"s":"A","t":"B","w":1.5}     │
└─────────────┴────────────┴───────────────────────────────┘
```

### Temporal Graph Representation

The Timeline structure inherently adds a temporal dimension to the graph data:

1. Each edge is associated with a specific timestamp
2. The graph's structure can be reconstructed for any point in time
3. Edges can be added, modified, or implicitly removed over time

#### Example: Graph Evolution

Consider a timeline with the following entries:

```
Timestamp: 1633042800000 (2021-10-01 00:00:00)
Value: {"s":"A","t":"B","w":1.0}

Timestamp: 1633046400000 (2021-10-01 01:00:00)
Value: {"s":"B","t":"C","w":2.0}

Timestamp: 1633050000000 (2021-10-01 02:00:00)
Value: {"s":"A","t":"B","w":1.5}  // Updated weight
```

This represents a graph that:
- Started with a single edge A→B with weight 1.0
- Added an edge B→C with weight 2.0 an hour later
- Updated the weight of edge A→B to 1.5 another hour later

### Querying the Graph Timeline

The edge-encoded graph can be queried using the Timeline's standard reading methods:

```javascript
timeline.nextString(function(error, item) {
  if (error || !item) return;
  
  // Parse the JSON string into an edge object
  const edge = JSON.parse(item.value);
  
  console.log(`At time ${new Date(item.time).toISOString()}`);
  console.log(`Edge: ${edge.s} → ${edge.t} (weight: ${edge.w})`);
});
```

### Extended Edge Properties

While `s`, `t`, and `w` are the required fields, the edge object can be extended with additional properties to represent complex graph structures:

```json
{
  "s": "A",
  "t": "B",
  "w": 1.5,
  "type": "friendship",
  "directed": true,
  "metadata": {
    "created_by": "user_123",
    "visibility": "public"
  }
}
```

### Reconstructing the Complete Graph

To reconstruct the complete graph at a specific point in time:

1. Reset the timeline cursor: `timeline.reset()`
2. Iterate through all edges up to the desired timestamp
3. For duplicate edges (same source and target), use the most recent version
4. Build an adjacency list or matrix representing the graph state

### Use Cases

This Graph Timeline format is particularly useful for:

- Social network evolution analysis
- Network traffic monitoring over time
- Financial transaction networks
- Infrastructure and dependency changes
- Any system where relationships between entities evolve over time

### Implementation Example

```javascript
// Add a new edge to the graph timeline
const edge = {
  s: "nodeA",
  t: "nodeB",
  w: 2.5
};

timeline.add(JSON.stringify(edge), function(error) {
  if (error) {
    console.error("Failed to add edge:", error);
    return;
  }
  console.log("Edge added successfully");
});
```

