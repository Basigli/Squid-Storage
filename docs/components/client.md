# Client

The **Client** (`SquidStorage` executable) provides a graphical interface for users to interact with the distributed storage system. It is built with **Dear ImGui**, **SDL2**, and **OpenGL 2**.

## Starting the Client

```bash
./SquidStorage [serverIP] [serverPort]
```

| Argument | Default | Description |
|----------|---------|-------------|
| `serverIP` | `127.0.0.1` | IP address of the running Server |
| `serverPort` | `12345` | TCP port the Server is listening on |

## GUI Overview

The client window shows:

- **File list panel** — displays all files currently tracked by the distributed storage.
- **New File** button — creates a new empty file.
- **Open** button — reads and displays the selected file's content in an editable text area.
- **Delete** button — deletes the selected file from the distributed storage.

Editing a file automatically acquires a distributed lock before the write and releases it after the update is committed.

## Dual-Socket Connection

When the Client starts it establishes **two TCP connections** to the Server:

| Socket | Purpose |
|--------|---------|
| **Primary** | Sending commands (CreateFile, UpdateFile, DeleteFile, AcquireLock, …) |
| **Secondary** | Receiving asynchronous push notifications (file updates from other clients) |

This separation prevents command responses and asynchronous updates from interleaving on the same socket.

## Supported Operations

| Operation | Protocol Message | Description |
|-----------|-----------------|-------------|
| Create file | `CreateFile` | Sends the file content to the Server for replication |
| Read file | `ReadFile` | Retrieves a file from the Server (which fetches it from a DataNode) |
| Update file | `UpdateFile` | Sends updated content; Server propagates to DataNodes |
| Delete file | `DeleteFile` | Removes the file from all DataNodes |
| Acquire lock | `AcquireLock` | Requests an exclusive write lock from the Server |
| Release lock | `ReleaseLock` | Frees the write lock after writing |
| Sync status | `SyncStatus` | Retrieves the full list of files and their versions |

## Disconnected (Read-Only) Mode

If the Client loses its connection to the Server, it:

1. Attempts to reconnect in a loop.
2. Switches all files to **read-only** mode until reconnection succeeds.

This ensures that a partitioned client cannot write stale data and create inconsistencies.

## Technology Stack

| Library | Version | Role |
|---------|---------|------|
| **Dear ImGui** | bundled | Immediate-mode GUI rendering |
| **SDL2** | system | Window creation, input handling |
| **OpenGL 2** | system | GPU-accelerated rendering backend |
