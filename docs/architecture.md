# Architecture

Squid Storage is composed of three independent components that communicate over TCP using the [Squid Protocol](protocol.md).

```
┌─────────────────────────────────────────────────────────────────┐
│                           CLIENT (GUI)                          │
│   SquidStorage  ←──────────────────────────────→  SquidStorage  │
│   (primary socket)                            (secondary socket) │
└────────────────────────────┬────────────────────────────────────┘
                             │  Squid Protocol (TCP)
                             ▼
┌────────────────────────────────────────────────────────────────┐
│                           SERVER                               │
│   - Metadata (dataNodeReplicationMap)                          │
│   - File lock management                                       │
│   - Heartbeat monitoring                                       │
│   - select()-based async I/O                                   │
└───────────┬────────────────────────────────────┬───────────────┘
            │  Squid Protocol (TCP)              │  Squid Protocol (TCP)
            ▼                                    ▼
┌───────────────────────┐            ┌───────────────────────┐
│       DATANODE 1      │            │       DATANODE 2      │
│   (file replicas)     │            │   (file replicas)     │
└───────────────────────┘            └───────────────────────┘
```

## Components

### Server (Central Coordinator)

The **Server** is the single source of truth for metadata. It:

- Accepts connections from Clients and DataNodes.
- Maintains the `dataNodeReplicationMap` — a mapping of each file to the DataNodes that hold a replica.
- Manages distributed **file locks** so that at most one client writes a file at a time.
- Forwards file operations (create, update, delete) to the relevant DataNodes and broadcasts changes to all connected Clients.
- Sends periodic **heartbeats** to DataNodes to detect failures.
- Calls `rebalanceFileReplication()` when a DataNode goes offline to restore the desired replication level.
- Uses the `select()` system call so it can monitor all sockets concurrently in a single thread.

More details: [Server Component](components/server.md)

### DataNode (Storage Node)

Each **DataNode** is a lightweight storage process. It:

- Stores file replicas in its working directory.
- Responds to heartbeat pings from the Server.
- Receives file content from the Server when a replication or update is needed.
- Sends file content to the Server when a Client requests a read.

More details: [DataNode Component](components/datanode.md)

### Client (GUI Application)

The **Client** provides a graphical interface backed by Dear ImGui + SDL2 + OpenGL. It:

- Opens **two TCP connections** to the Server at startup — a *primary* socket for sending commands and a *secondary* socket for receiving asynchronous updates pushed by the Server.
- Lets users create, open (read), edit (update), and delete files.
- Acquires a distributed lock before writing and releases it when done.
- Switches to **read-only mode** when disconnected to prevent inconsistencies.

More details: [Client Component](components/client.md)

## Common Shared Code

The `common/` module contains code shared by all three components:

| Module | Path | Description |
|--------|------|-------------|
| `SquidProtocol` | `common/src/squidprotocol/` | Message serialisation/deserialisation and send/receive helpers |
| `FileManager` | `common/src/filesystem/` | File CRUD operations and version tracking |
| `FileLock` | `common/src/filesystem/` | Distributed lock data structure |
| `FileTransfer` | `common/src/filesystem/` | Chunked file transfer over a socket |
| `Peer` | `common/src/peer/` | Base class with socket management and reconnection logic |

## Data Flow — File Update

```
Client                       Server                      DataNodes
  │                             │                             │
  │── AcquireLock(file) ───────▶│                             │
  │◀─ Response(isLocked) ───────│                             │
  │                             │                             │
  │── UpdateFile(file) ────────▶│                             │
  │                             │── UpdateFile(file) ────────▶│ (replica 1)
  │                             │── UpdateFile(file) ────────▶│ (replica 2)
  │◀─ ACK ──────────────────────│                             │
  │                             │                             │
  │── ReleaseLock(file) ───────▶│                             │
  │◀─ ACK ──────────────────────│                             │
```

## Replication and Fault Tolerance

- Each file is replicated across `replicationFactor` DataNodes (default: 2).
- The Server tracks which DataNodes hold each file in `dataNodeReplicationMap`.
- When a DataNode goes offline (missed heartbeat), the Server removes it from the map and calls `rebalanceFileReplication()` to assign new replicas.
- Clients automatically attempt to reconnect in a loop when they lose the server connection.
- While disconnected, files are shown in **read-only mode** to preserve consistency.

## Consistency Model

Squid Storage prioritises **consistency over availability**:

- A Client can only write when it holds a distributed lock **and** is connected to the Server.
- All writes are propagated synchronously to all DataNodes before the Server acknowledges the Client.
- If a component is partitioned, it cannot modify shared state; it can only read its local copy.
