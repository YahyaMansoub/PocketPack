# Project Plan

A milestone-by-milestone roadmap for building PocketPack while learning Rust at the same time.

The guiding principle: **learn just in time**. Don't finish a Rust course before writing a line of code. Learn each concept exactly when you need it to move the project forward.

---

## Phase 0 — Rust learning track (before writing project code)

Learn only what you immediately need. Skip the rest for now.

### Concepts to cover

| Concept | Why you need it |
|---|---|
| Ownership & borrowing | everywhere in Rust — you can't avoid it |
| Structs & enums | to model header, entries, errors |
| `Result` & `Option` | all I/O returns these |
| Traits | `Read`, `Write`, `Display`, compression abstraction |
| Modules & crates | project layout, `Cargo.toml` |
| File I/O (`std::fs`) | core of the whole tool |
| `Vec<u8>` & slices | byte buffers for binary data |
| Testing (`cargo test`) | needed from Milestone 2 onward |
| CLI args (`clap`) | needed for the `pack`/`list`/`unpack` commands |

### Suggested mini exercises (in order)

1. Read a file into a `Vec<u8>` and print its length.
2. Write a `Vec<u8>` to a new file.
3. Walk a directory recursively and print every path.
4. Manually write a small fixed-size header to a file (magic + version as raw bytes).
5. Parse those bytes back into a struct and assert the values match.

These five exercises directly prepare you for Milestone 2.

---

## Milestone 1 — Project skeleton

**Goal:** a compilable Rust project with the right module structure and no logic yet.

### Tasks

- [ ] `cargo new pocketpack` (or rename existing)
- [ ] Add dependencies to `Cargo.toml`: `clap`, `anyhow`, `walkdir`
- [ ] Create all source files listed in `STRUCTURE.md` with empty module stubs
- [ ] Wire up `main.rs` → `cli.rs` so `cargo run -- --help` prints usage
- [ ] Add a `tests/` directory

### Done when

`cargo build` succeeds and `pocketpack --help` prints the three subcommands.

---

## Milestone 2 — Custom archive format (read + write)

**Goal:** write a header and one file entry to disk, then read it back.

### Tasks

- [ ] Implement `format/header.rs`: `Header` struct with `magic`, `version`, `compression`, `entry_count`
- [ ] Implement `format/entry.rs`: `Entry` struct and `EntryType` enum
- [ ] Implement `format/writer.rs`: serialize `Header` + `Vec<Entry>` to a `Write` sink
- [ ] Implement `format/reader.rs`: parse bytes from a `Read` source into `Header` + `Vec<Entry>`
- [ ] Write `tests/format_roundtrip.rs`: write then read, assert byte-for-byte equality

### Concepts you will need

- `std::io::{Read, Write}`
- endianness: use `u32::to_le_bytes()` / `u32::from_le_bytes()`
- `impl Trait` patterns

### Done when

The roundtrip test passes for a header + one file entry + one directory entry.

---

## Milestone 3 — Pack and unpack

**Goal:** `pocketpack pack <dir> -o out.fpk` and `pocketpack unpack out.fpk -d out/` work end-to-end.

### Tasks

- [ ] Implement `fs/walk.rs`: return `Vec<PathBuf>` for all files under a directory
- [ ] Implement `fs/paths.rs`: strip the root prefix so paths inside the archive are relative
- [ ] Implement `archive/pack.rs`: walk directory → build `Vec<Entry>` → write archive
- [ ] Implement `archive/unpack.rs`: read archive → create dirs → write files
- [ ] Implement `archive/list.rs`: read archive → print table (path, type, size)
- [ ] Connect commands in `cli.rs` / `main.rs`
- [ ] Write `tests/pack_unpack.rs`: pack a temp directory, unpack it, assert identical trees

### Done when

```
pocketpack pack ./samples -o samples.fpk
pocketpack list samples.fpk
pocketpack unpack samples.fpk -d ./out
diff -r ./samples ./out   # ← should report no differences
```

---

## Milestone 4 — Test coverage

**Goal:** edge cases are covered before touching compression.

### Test cases to add

- [ ] Single file (tiny, <1 KB)
- [ ] Single empty file
- [ ] Deeply nested directory structure
- [ ] Binary file (PNG, ELF, etc.)
- [ ] Archive with only directories (no files)
- [ ] Path with spaces and Unicode characters

### Done when

`cargo test` is green for all the above.

---

## Milestone 5 — Compression abstraction

**Goal:** add a `Compression` enum so the rest of the code never touches compression directly.

### Tasks

- [ ] Define `Compression` enum in `compression/mod.rs`: `None`, `Gzip`, `Rle`
- [ ] Implement `compression/none.rs`: identity pass-through
- [ ] Implement `compression/gzip.rs`: wrap `flate2` (add `flate2` to `Cargo.toml`)
- [ ] Wire compression into `archive/pack.rs` (compress entry data before writing)
- [ ] Wire decompression into `archive/unpack.rs` (decompress after reading)
- [ ] Expose `--compression` flag in CLI (default: `none`)
- [ ] Regression test: pack with gzip, unpack, compare to source

### Done when

`pocketpack pack ./samples -o samples.fpk --compression gzip` and subsequent unpack produce identical output to the no-compression path.

---

## Milestone 6 — Hand-rolled RLE compressor (learning exercise)

**Goal:** implement a toy compressor from scratch to understand how compression works internally.

### Tasks

- [ ] Implement `compression/rle.rs`: simple run-length encoding over `Vec<u8>`
- [ ] Write unit tests for RLE: compress then decompress, assert roundtrip
- [ ] Plug `Compression::Rle` into the archive pipeline
- [ ] Compare compressed sizes with `Compression::None` and `Compression::Gzip`

### Notes

RLE is not efficient for most real-world data. That is fine — the goal is to understand encoding/decoding logic, not to compete with zstd.

---

## Milestone 7 — Polish and hardening

**Goal:** make the tool reliable and user-friendly enough to show off.

### Tasks

- [ ] Add per-entry CRC32 checksum (verify on unpack, error on mismatch)
- [ ] Detect and reject truncated / corrupted archives with a clear error message
- [ ] Add `--verbose` / `-v` flag: print each file as it is packed or unpacked
- [ ] Add progress output for large archives
- [ ] Store and restore Unix file permissions
- [ ] Store and restore modification timestamps

---

## Tech choices

| Need | Choice | Notes |
|---|---|---|
| CLI parsing | `clap` (derive API) | easiest to use for beginners |
| Error handling | `anyhow` | good for application code |
| Error types | `thiserror` | good for library-style error enums |
| Directory walking | `walkdir` | handles symlinks and cross-platform paths |
| Byte order helpers | manual `u32::to_le_bytes()` first | learn the concept before reaching for a crate |
| Gzip compression | `flate2` | well maintained, simple API |
| Future compression | `zstd` or `lz4` | if you want better ratios after Milestone 6 |
| Testing | built-in `cargo test` + `tempfile` crate | no extra framework needed |

---

## Learning checkpoints

After each milestone you should be able to answer:

| After | Question |
|---|---|
| Phase 0 | What does the borrow checker actually prevent? |
| Milestone 2 | How do you write a `u64` in little-endian to a file? |
| Milestone 3 | Why do you strip path prefixes when packing? |
| Milestone 4 | Why should you test with binary files, not just text? |
| Milestone 5 | What does a compression abstraction boundary buy you? |
| Milestone 6 | At what kind of data does RLE perform well vs. poorly? |
| Milestone 7 | What does a checksum protect against? |

---

## Honest scope warning

**Do not start with:**
- full tar/gzip compatibility
- Windows-compatible path handling
- async I/O
- parallel compression

**Start with:**
> "I will make a minimal custom archive format that can pack, list, and extract files."

Everything else is a later milestone.
