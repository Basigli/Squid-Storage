# Getting Started

This page explains how to build Squid Storage from source and run the three components — Server, DataNode, and Client.

## Prerequisites

| Dependency | Version | Notes |
|------------|---------|-------|
| C++ compiler | C++17 or later | GCC or Clang |
| CMake | 3.10+ | Build system |
| SDL2 | any recent | GUI windowing |
| OpenGL | system | GUI rendering |

### Install dependencies on Ubuntu

```bash
sudo apt-get update
sudo apt-get install -y cmake libsdl2-dev libglu1-mesa-dev freeglut3-dev mesa-common-dev
```

### Install dependencies on macOS

```bash
brew install cmake sdl2
```

OpenGL is provided by the system framework on macOS and does not require a separate install.

## Building

Clone the repository and build with CMake:

```bash
git clone https://github.com/Basigli/Squid-Storage.git
cd Squid-Storage

cmake -B build
cmake --build build
```

Three executables will be produced inside the `build/` directory:

| Executable | Description |
|------------|-------------|
| `SquidStorageServer` | Central coordinator server |
| `DataNode` | Storage node |
| `SquidStorage` | GUI client |

## Running

### 1. Start the Server

The server must be started first. All other components connect to it.

```bash
./build/SquidStorageServer [port] [replicationFactor] [timeoutSeconds]
```

**Example — default settings:**

```bash
./build/SquidStorageServer
# Starting server on port: 12345, replication factor: 2, timeout: 60s
```

**Example — custom settings:**

```bash
./build/SquidStorageServer 8080 3 120
# Starting server on port: 8080, replication factor: 3, timeout: 120s
```

See [Configuration](configuration.md) for a full description of each parameter.

### 2. Start one or more DataNodes

Each DataNode connects to the running server. Start each instance from its own directory so that files are stored separately.

```bash
mkdir -p datanode1 && cd datanode1
../build/DataNode [serverIP] [serverPort]
```

**Example:**

```bash
mkdir -p datanode1 && (cd datanode1 && ../build/DataNode 127.0.0.1 12345)
mkdir -p datanode2 && (cd datanode2 && ../build/DataNode 127.0.0.1 12345)
```

The DataNode uses the current working directory as its storage root.

### 3. Start the Client

```bash
./build/SquidStorage [serverIP] [serverPort]
```

**Example:**

```bash
./build/SquidStorage 127.0.0.1 12345
```

The GUI window will open. You can create, open, edit, and delete files that are automatically replicated across all connected DataNodes.

## Full Local Cluster Example

The following sequence starts a complete local cluster with one server, two DataNodes, and one client:

```bash
# Terminal 1 — server
./build/SquidStorageServer 12345 2 60

# Terminal 2 — datanode 1
mkdir -p /tmp/dn1 && (cd /tmp/dn1 && /path/to/build/DataNode 127.0.0.1 12345)

# Terminal 3 — datanode 2
mkdir -p /tmp/dn2 && (cd /tmp/dn2 && /path/to/build/DataNode 127.0.0.1 12345)

# Terminal 4 — client
./build/SquidStorage 127.0.0.1 12345
```

The replication factor is set to `2`, so each file will be stored on both DataNodes.

## Test Scripts

The `test/` directory contains shell scripts that automate a multi-component test using **tmux**:

```bash
test/testMultiple.sh       # server + datanodes + clients
test/testMultiClient.sh    # multiple clients only
test/testMultiDataNode.sh  # multiple datanodes only
```

These scripts require `tmux` to be installed.
