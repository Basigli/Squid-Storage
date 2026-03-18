# Server

The **Server** (`SquidStorageServer`) is the central coordinator of Squid Storage. It manages all metadata, routes file operations between Clients and DataNodes, and monitors the health of every connected node.

## Responsibilities

- Accept incoming TCP connections from Clients and DataNodes.
- Route and propagate file operations (create, read, update, delete).
- Maintain the `dataNodeReplicationMap` — which DataNodes hold replicas of each file.
- Manage distributed **file locks** and check for lock expiry.
- Send periodic **heartbeats** to DataNodes; remove unresponsive nodes from the replication map.
- Rebalance file replicas when the actual replication count falls below the configured factor.
- Use `select()` to handle all sockets concurrently without spawning per-connection threads.

## Starting the Server

```bash
./SquidStorageServer [port] [replicationFactor] [timeoutSeconds]
```

All arguments are optional and fall back to defaults:

| Argument | Default | Description |
|----------|---------|-------------|
| `port` | `12345` | TCP port to listen on |
| `replicationFactor` | `2` | Number of DataNode replicas per file |
| `timeoutSeconds` | `60` | Socket operation timeout in seconds |

## Internal State

### `dataNodeReplicationMap`

```
map<string /*filePath*/, map<string /*datanodeName*/, SquidProtocol>>
```

Tracks, for every file, which DataNodes currently hold a replica and the corresponding protocol endpoint. When a DataNode disconnects, it is erased from this map and `rebalanceFileReplication()` is triggered.

### `clientEndpointMap`

```
map<string /*clientName*/, pair<SquidProtocol /*primary*/, SquidProtocol /*secondary*/>>
```

Each Client opens two connections to the Server. The Server stores both so it can push asynchronous updates through the secondary socket.

### `dataNodeEndpointMap`

```
map<string /*datanodeName*/, SquidProtocol>
```

Active DataNode connections indexed by name.

### `fileLockMap`

```
map<string /*filePath*/, FileLock>
```

Stores the current lock holder and lock timestamp for every file. `checkFileLockExpiration()` periodically releases stale locks (default interval: 5 minutes).

## Key Methods

| Method | Description |
|--------|-------------|
| `run()` | Main event loop — calls `select()` and dispatches incoming messages |
| `handleAccept()` | Handles a new incoming connection; performs the handshake |
| `handleConnection()` | Processes a received message from a Client or DataNode |
| `propagateCreateFile()` | Sends a `CreateFile` message to the selected DataNodes |
| `propagateUpdateFile()` | Sends an `UpdateFile` message to all DataNodes holding the file |
| `propagateDeleteFile()` | Sends a `DeleteFile` message to all DataNodes holding the file |
| `getFileFromDataNode()` | Retrieves a file from a DataNode and forwards it to a Client |
| `sendHeartbeats()` | Pings all DataNodes; removes those that do not respond |
| `rebalanceFileReplication()` | Assigns new DataNodes to a file whose replica count is below the threshold |
| `acquireLock()` / `releaseLock()` | Grants or releases a distributed write lock for a file |
| `checkFileLockExpiration()` | Periodically releases locks that have exceeded the timeout |

## Connection Handshake

When a new socket connects, the Server sends an `Identify` request. The peer responds with its `nodeType` (`CLIENT` or `DATANODE`) and its `processName`. Based on the type the Server:

- **Client**: records both the primary socket (for commands) and waits for a second connection to be registered as the secondary (update) socket.
- **DataNode**: registers the endpoint in `dataNodeEndpointMap` and starts including it in replication decisions.

## Replication Flow

1. A Client calls `CreateFile` or `UpdateFile`.
2. The Server selects `replicationFactor` DataNodes from `dataNodeEndpointMap` using round-robin.
3. The Server forwards the file to each selected DataNode via `propagateCreateFile()` / `propagateUpdateFile()`.
4. The Server updates `dataNodeReplicationMap` to record the new assignments.
5. The Server broadcasts the change to all other connected Clients.

## Fault Tolerance

- The `sendHeartbeats()` loop runs in a background thread.
- Any DataNode that fails to respond is removed from `dataNodeEndpointMap` and `dataNodeReplicationMap`.
- `rebalanceFileReplication()` is immediately invoked for every file that lost a replica, pushing a fresh copy to a healthy DataNode.
