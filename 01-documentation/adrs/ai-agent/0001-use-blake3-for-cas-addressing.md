# ADR-0001: Use Blake3 for CAS Content Addressing

## Status

Accepted

## Context

The CAS Data Lake needs a content-addressing hash function. Candidates: SHA-256, SHA-512, Blake2b, Blake3.

Requirements:
- Fast hashing for large blobs (100MB+)
- Incremental hashing for streaming writes
- Strong collision resistance
- Wide language support (Rust, Go, Python, C++, Scala, C#)

## Decision

Use **Blake3**.

- 10x faster than SHA-256 on large inputs
- Supports incremental hashing natively
- Trivially parallelizable (SIMD)
- Available in all tier languages via native bindings
- 256-bit output (same security margin as SHA-256)

## Consequences

- Deterministic addressing: same content = same hash = same `cas://` URI
- Append-only storage: content never changes, so hashes never collide
- Fast verification: rehash on read to detect corruption

## Alternatives Considered

| Alternative | Reason Against |
| :--- | :--- |
| SHA-256 | 3-10x slower, no incremental mode |
| Blake2b | Slower than Blake3, less ecosystem support |
| xxHash | Not cryptographically secure |
