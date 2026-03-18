# Squid Protocol

The **Squid Protocol** is a lightweight, text-based application protocol used for all communication between Clients, the Server, and DataNodes.

## Message Format

Every message follows the pattern:

```
Keyword<ArgName1:Arg1, ArgName2:Arg2, ..., ArgNameN:ArgN>
```

Arguments are comma-separated key-value pairs enclosed in angle brackets. Messages with no arguments still include the angle brackets:

```
Heartbeat<>
```

### Example Messages

```
CreateFile<filePath:/squidstorage/notes.txt>
UpdateFile<filePath:/squidstorage/notes.txt>
AcquireLock<filePath:/squidstorage/notes.txt>
Response<ACK>
Response<isLocked:true>
Identify<>
Response<nodeType:DATANODE, processName:dn-1>
```

## Message Reference

| Keyword | Direction | Arguments | Response |
|---------|-----------|-----------|----------|
| `CreateFile` | Client → Server, Server → DataNode | `filePath` | `ACK` |
| `TransferFile` | Server → DataNode | `filePath` | `ACK` |
| `ReadFile` | Client → Server | `filePath` | `ACK` + file content |
| `UpdateFile` | Client → Server, Server → DataNode | `filePath` | `ACK` |
| `DeleteFile` | Client → Server, Server → DataNode | `filePath` | `ACK` |
| `AcquireLock` | Client → Server | `filePath` | `isLocked` (true/false) |
| `ReleaseLock` | Client → Server | `filePath` | `ACK` |
| `Heartbeat` | Server → DataNode | — | `ACK` |
| `SyncStatus` | Client/DataNode → Server | — | `ACK` + file version map |
| `Identify` | Server → peer | — | `nodeType`, `processName` |
| `Response` | any | `ACK` / `nodeType,processName` / `isLocked` | — |
| `Close` | any | — | `ACK` |

## Argument Details

### `filePath`

The path of the file inside the shared storage namespace. Always an absolute-style path beginning with `/`, for example `/squidstorage/report.txt`.

### `isLocked`

Boolean string returned by `AcquireLock`. `true` means the lock was successfully granted; `false` means another client currently holds it.

### `nodeType`

Returned in an `Identify` response. One of:

- `CLIENT`
- `DATANODE`

### `processName`

A human-readable identifier for the connecting process (e.g. a DataNode instance name or client window title). Used by the Server to populate its endpoint maps.

## File Transfer

File content is transmitted immediately after the protocol message using a **length-prefixed** binary transfer:

1. The sender transmits the message (e.g. `CreateFile<filePath:...>`).
2. The sender then transmits the file length as a 4-byte big-endian integer.
3. The sender transmits the raw file bytes.
4. The receiver sends a `Response<ACK>`.

The `FileTransfer` class in `common/src/filesystem/filetransfer.cpp` implements both sides.

## Versioned Operations

`CreateFile` and `UpdateFile` can optionally carry a `version` argument when the Server needs to push a specific version to a DataNode during rebalancing:

```
CreateFile<filePath:/squidstorage/notes.txt, version:3>
```

## Implementation

The protocol is implemented in:

| File | Description |
|------|-------------|
| `common/src/squidprotocol/squidprotocol.hpp/.cpp` | `SquidProtocol` class — send/receive helpers for every message type |
| `common/src/squidprotocol/squidProtocolFormatter.hpp/.cpp` | Serialises and deserialises `Message` objects to/from the text wire format |
| `server/src/squidProtocolServer.cpp` | Server-side request dispatcher |
