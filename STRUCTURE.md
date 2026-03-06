# Project Structure

This document describes the planned source layout for PocketPack. No code files have been created yet — this is the design blueprint.

```
PocketPack/
├── Cargo.toml                  # workspace manifest, dependencies (clap, anyhow, walkdir, flate2…)
├── README.md
├── STRUCTURE.md                ← this file
├── PROJECT_PLAN.md
│
├── src/
│   ├── main.rs                 # entry point — parse CLI args and dispatch commands
│   ├── cli.rs                  # clap command definitions (pack, list, unpack)
│   ├── error.rs                # unified error type (thiserror / anyhow)
│   │
│   ├── format/                 # "what bytes mean" — pure binary format logic
│   │   ├── mod.rs
│   │   ├── header.rs           # magic bytes, version, compression flag, entry count
│   │   ├── entry.rs            # entry types: File / Dir; path, sizes, data
│   │   ├── writer.rs           # serialize header + entries to bytes
│   │   └── reader.rs           # parse bytes back into header + entries
│   │
│   ├── archive/                # "what to do with the format" — pack/unpack logic
│   │   ├── mod.rs
│   │   ├── pack.rs             # walk filesystem → build entries → write archive
│   │   ├── unpack.rs           # read archive → recreate files/dirs on disk
│   │   └── list.rs             # read archive → print table of contents
│   │
│   ├── compression/            # compress/decompress byte buffers
│   │   ├── mod.rs              # Compression enum: None, Gzip, Rle
│   │   ├── none.rs             # pass-through (no compression)
│   │   ├── gzip.rs             # flate2-based gzip wrapper
│   │   └── rle.rs              # hand-rolled run-length encoding (learning exercise)
│   │
│   ├── fs/                     # filesystem helpers
│   │   ├── mod.rs
│   │   ├── walk.rs             # recursive directory walking (uses walkdir)
│   │   └── paths.rs            # path normalisation, stripping prefixes
│   │
│   └── util/
│       ├── mod.rs
│       └── io.rs               # read_exact helpers, write_u64_le, etc.
│
├── tests/                      # integration tests
│   ├── pack_unpack.rs          # pack a folder, unpack it, assert identical content
│   ├── format_roundtrip.rs     # write a header+entries to bytes, parse them back
│   └── cli.rs                  # invoke binary via Command, check stdout/files
│
└── examples/                   # small runnable demos
    └── basic_pack.rs           # hardcoded demo: pack "examples/sample/" → out.fpk
```

## Module responsibilities at a glance

| Module | Responsibility |
|---|---|
| `cli` | User-facing commands and argument parsing only |
| `format` | Binary layout: how bytes are structured on disk |
| `archive` | Orchestration: drives `format` + `fs` + `compression` together |
| `compression` | Transform raw byte buffers (compress / decompress) |
| `fs` | Walk directories, normalise paths |
| `util` | Low-level byte I/O helpers shared across modules |
| `error` | Single error type used everywhere |

## Archive file format (`.fpk`)

```
┌──────────────────────────────────────┐
│ HEADER                               │
│   magic        4 bytes  "FPKG"       │
│   version      1 byte   e.g. 1       │
│   compression  1 byte   0=none 1=gz  │
│   entry_count  4 bytes  u32 LE       │
├──────────────────────────────────────┤
│ ENTRY  (repeated entry_count times)  │
│   entry_type   1 byte   0=file 1=dir │
│   path_len     2 bytes  u16 LE       │
│   path_bytes   path_len bytes        │
│   orig_size    8 bytes  u64 LE       │
│   stored_size  8 bytes  u64 LE       │
│   data         stored_size bytes     │
└──────────────────────────────────────┘
```

Fields planned for later milestones:
- `checksum` — CRC32 or xxHash per entry
- `permissions` — Unix mode bits
- `mtime` — modification timestamp
