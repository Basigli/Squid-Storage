# Squid Storage

[![CMake Build (Ubuntu + macOS)](https://github.com/MHS-20/Squid-Storage/actions/workflows/cmake-multi-platform.yml/badge.svg)](https://github.com/MHS-20/Squid-Storage/actions/workflows/cmake-multi-platform.yml)

**Squid Storage** is a distributed storage system designed to manage file replication and consistency of a specific folder across multiple nodes.

It provides:

- **Automatic replication** — files are replicated across multiple DataNodes for fault tolerance.
- **Distributed file locking** — prevents concurrent writes and data corruption.
- **Graphical client interface** — users can create, read, update, and delete files through a GUI.
- **Partition tolerance** — every component keeps running and attempts reconnection if disconnected.
- **Real-time propagation** — changes are immediately broadcast to all connected clients and DataNodes.

## Key Components

| Component | Role |
|-----------|------|
| **Server** | Central coordinator — manages file metadata, client/DataNode connections, and replication |
| **DataNode** | Storage node — stores file replicas and responds to server requests |
| **Client** | GUI application — lets users interact with the distributed storage system |

## Quick Links

- [Getting Started](getting-started.md) — build and run Squid Storage
- [Architecture](architecture.md) — how the system is designed
- [Squid Protocol](protocol.md) — the custom wire protocol
- [Configuration](configuration.md) — runtime parameters
- [Contributing](contributing.md) — how to contribute

## Platform Support

Squid Storage is continuously built and tested on **Ubuntu** and **macOS** via GitHub Actions.
