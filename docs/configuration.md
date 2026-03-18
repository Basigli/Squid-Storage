# Configuration

All three Squid Storage executables accept runtime configuration via command-line arguments. No configuration files are required.

## Server

```bash
./SquidStorageServer [port] [replicationFactor] [timeoutSeconds]
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `port` | integer | `12345` | TCP port the server listens on |
| `replicationFactor` | integer | `2` | Number of DataNode replicas maintained per file. A higher value improves fault tolerance but increases storage usage. |
| `timeoutSeconds` | integer | `60` | Socket operation timeout in seconds. Connections that do not respond within this window are considered failed. |

There are also two compile-time constants in `server/src/server.hpp`:

| Constant | Default | Description |
|----------|---------|-------------|
| `DEFAULT_LOCK_INTERVAL` | `5` (minutes) | How frequently stale file locks are checked and released |
| `DEFAULT_PATH` | `./test_txt/test_server` | Default working directory for the server's own file store |
| `BUFFER_SIZE` | `1024` bytes | Read buffer size for socket I/O |

## DataNode

```bash
./DataNode [serverIP] [serverPort]
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `serverIP` | string | `127.0.0.1` | IP address of the Server to connect to |
| `serverPort` | integer | `12345` | TCP port of the Server |

The DataNode uses its **current working directory** as the storage root. Set this by running the process from the desired directory.

## Client

```bash
./SquidStorage [serverIP] [serverPort]
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `serverIP` | string | `127.0.0.1` | IP address of the Server to connect to |
| `serverPort` | integer | `12345` | TCP port of the Server |

## Choosing the Replication Factor

| Replication Factor | Meaning | Minimum DataNodes Required |
|--------------------|---------|---------------------------|
| `1` | No redundancy — file exists on exactly one DataNode | 1 |
| `2` | Can survive the loss of one DataNode (default) | 2 |
| `3` | Can survive the simultaneous loss of two DataNodes | 3 |
| `N` | Can survive the loss of N−1 DataNodes | N |

If the number of available DataNodes is less than the replication factor, the Server replicates to as many nodes as are available and will automatically increase replicas as new nodes join.
