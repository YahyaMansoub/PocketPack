# PocketPack

> A custom archive and compression tool built in Rust from scratch to learn systems programming, binary formats, file I/O, and compression design.

## What is this?

PocketPack is a learning project with two goals running in parallel:

1. **Learn Rust** — ownership, borrowing, traits, error handling, CLI, file I/O
2. **Build a real archiver** — a custom binary archive format (like a minimal `tar`), with optional compression

The archive format uses the `.fpk` extension. The tool supports packing files/folders into a single archive, listing its contents, and unpacking it back.

## Quick overview

```
pocketpack pack <folder> -o archive.fpk
pocketpack list archive.fpk
pocketpack unpack archive.fpk -d out/
```

## Documentation

- [`STRUCTURE.md`](./STRUCTURE.md) — planned source layout and module responsibilities
- [`PROJECT_PLAN.md`](./PROJECT_PLAN.md) — milestone-by-milestone development roadmap

## Status

🚧 Project in planning phase — see [`PROJECT_PLAN.md`](./PROJECT_PLAN.md) for the roadmap.
