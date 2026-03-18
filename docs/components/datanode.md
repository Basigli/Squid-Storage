# DataNode

A **DataNode** (`DataNode` executable) is a storage process that holds file replicas and serves them to the Server on request. You can run as many DataNode instances as you like; the Server distributes replicas across all available nodes.

## Responsibilities

- Store file replicas in its working directory.
- Receive and persist files sent by the Server (`CreateFile`, `UpdateFile`).
- Delete files on `DeleteFile` requests from the Server.
- Respond to `Heartbeat` pings to signal availability.
- Serve file content to the Server when a Client reads a file (`ReadFile`).

## Starting a DataNode

```bash
./DataNode [serverIP] [serverPort]
```

Both arguments are optional:

| Argument | Default | Description |
|----------|---------|-------------|
| `serverIP` | `127.0.0.1` | IP address of the running Server |
| `serverPort` | `12345` | TCP port the Server is listening on |

The DataNode uses its **current working directory** as the storage root. Run each instance from a different directory so that their file stores do not overlap:

```bash
mkdir -p /var/storage/dn1 && (cd /var/storage/dn1 && /path/to/DataNode 192.168.1.10 12345)
mkdir -p /var/storage/dn2 && (cd /var/storage/dn2 && /path/to/DataNode 192.168.1.10 12345)
```

## Lifecycle

1. **Connect** — the DataNode establishes a TCP connection to the Server.
2. **Identify** — the Server sends an `Identify` request; the DataNode responds with `nodeType=DATANODE` and its `processName`.
3. **Receive files** — the Server pushes file replicas as `CreateFile` / `UpdateFile` / `DeleteFile` messages.
4. **Heartbeat** — the Server periodically sends `Heartbeat` messages; the DataNode replies with `ACK`.
5. **Reconnect** — if the connection drops, the DataNode continuously retries the connection in a loop.

## Storage Layout

Files are stored relative to the DataNode's working directory, mirroring the path used in the Squid Protocol messages. For example, a file with `filePath:/squidstorage/report.txt` is stored as `./squidstorage/report.txt` inside the DataNode's directory.

The `FileManager` component (from `common/`) tracks file versions using a `.fileVersion.txt` metadata file so that stale replicas can be detected and updated.

## Version Tracking

Each file has an integer version number. When the Server rebalances replicas, it includes the current version so the receiving DataNode can verify it has the latest content. The `SyncStatus` message allows the Server to compare file version maps across DataNodes.
