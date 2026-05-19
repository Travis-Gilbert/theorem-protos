# theorem-protos

Canonical gRPC protocol definitions for **Theorem's Harness**, **RustyRed-GraphDB**, and **Theseus search**.

This repo is the **single source of truth** for inter-service contracts across the Theseus ecosystem. Each product has its own proto package; they version independently; consumers depend on this repo as a git submodule, subtree, or Cargo/Python dep so updates to a contract flow into every consumer through one canonical path.

## Three products, three contracts

| Product | Proto package | What it is | Implementations |
|---|---|---|---|
| **RustyRed-GraphDB** | `rustyred.v1` | Standalone graph + vector database (OSS). Graph ops, vector search, fulltext, spatial, algorithms. | [`Travis-Gilbert/RustyRed-Graph-Database`](https://github.com/Travis-Gilbert/RustyRed-Graph-Database) |
| **theorems-harness** | `theorems_harness.v1` | The full harness for agents: sessions, cross-agent coordination, presence, agent-memory. Theseus-customized fork of RustyRed Core with THG protocol native support. | `rustyredcore_THG/` inside `Travis-Gilbert/Theseus` |
| **theseus-search** | `theseus_search.v1` | Gap-driven search orchestrator. Sits on top of theorems-harness or rustyred. Search, gap-walk, source-pair, provenance. | TBD — implemented inside `Travis-Gilbert/Theseus` once the orchestrator is built |

The three contracts are **sibling products**, not parent-child. theorems-harness and rustyred.v1 will have similar database-operation surfaces because both products share the RedCore storage substrate, but the duplication is intentional — each consumer should only need to depend on the contract for the product it's using.

## Layout

```
theorem-protos/
  rustyred/v1/rustyred.proto              # RustyRed-GraphDB database surface
  theorems_harness/v1/harness.proto       # theorems-harness sessions/coordination/presence/agent-memory
  theseus_search/v1/search.proto          # Search orchestrator (search/gapWalk/sourcePair/provenance)
  README.md
  LICENSE                                  # MIT
```

## Consumption

### Rust (tonic)

```toml
# Cargo.toml of any consumer
[build-dependencies]
tonic-build = "0.12"
prost-build = "0.13"
```

Add this repo as a git submodule or as a path dependency in a workspace, then in `build.rs`:

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    tonic_build::configure()
        .build_server(true)
        .build_client(true)
        .compile_protos(
            &[
                "theorem-protos/rustyred/v1/rustyred.proto",
                "theorem-protos/theorems_harness/v1/harness.proto",
                "theorem-protos/theseus_search/v1/search.proto",
            ],
            &["theorem-protos"],
        )?;
    Ok(())
}
```

### Python (grpcio)

```bash
pip install grpcio-tools
python -m grpc_tools.protoc \
  -I theorem-protos \
  --python_out=. \
  --grpc_python_out=. \
  theorem-protos/rustyred/v1/rustyred.proto
```

### TypeScript (ts-proto / @bufbuild/protoc-gen-es)

See your client framework's docs — these protos are syntax = "proto3" and use no exotic options.

## Versioning policy

- Each package is independently versioned (`rustyred.v1`, `theorems_harness.v1`, `theseus_search.v1`).
- Major versions land in new packages (`v2`, `v3`) — never break a published `v1`.
- Within a version, additions (new methods, new optional fields) are backward-compatible; removals and changes are not.
- Tagged releases of this repo bump a top-level `theorem-protos` version that consumers can pin.

## Status

This repo is the contract authoring substrate. Implementations are in progress:

- `rustyred.v1` — being implemented in `rustyred-server` of the RustyRed-Graph-Database repo
- `theorems_harness.v1` — Sessions/Coordination/Presence/AgentMemory are scaffolded per the [Cross-Agent Coordination Spec](https://github.com/Travis-Gilbert/Theseus/blob/main/Theseus/Feature%20Spec%20Cross-Agent%20Coordination%2C%20Persistent%20Agent%20Identity%2C%20and%20gRPC%20Harness%20API.md). The non-coordination methods (database ops) are reserved for the implementation that follows.
- `theseus_search.v1` — being implemented inside `apps/notebook/search/` of the Theseus repo

## License

MIT (see LICENSE).
