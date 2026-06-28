# craft-rust-maintainer Architecture

This cartridge packages Rust maintenance judgment as a CRAFT harness. It is intentionally small: the manifest points at a system prompt, memory schema, tool expectations, and validator checks that can be installed, inspected, and composed with other cartridges.

## Cartridge Shape

```mermaid
flowchart TB
    Manifest["craft.toml<br/>name, version, model, file refs"] --> Prompt["prompts/system.md<br/>review stance and operating rules"]
    Manifest --> Memory["memory/schema.toml<br/>facts and events to retain"]
    Manifest --> Tools["tools/mcp.toml<br/>expected local capabilities"]
    Manifest --> Validators["validators/checks.tdd<br/>required and forbidden output patterns"]

    Prompt --> Review["Rust maintenance review"]
    Memory --> Review
    Tools --> Review
    Validators --> Review

    Review --> Findings["file:line findings"]
    Review --> Fixes["small scoped fixes"]
    Review --> Verification["cargo fmt, clippy, test, doc"]
```

## Review Loop

```mermaid
sequenceDiagram
    participant User as User
    participant Harness as rust-maintainer
    participant Repo as Rust repo
    participant Checks as Cargo checks

    User->>Harness: Review or fix a scoped Rust task
    Harness->>Repo: Read nearby code and existing patterns
    Harness->>Repo: Identify highest-risk issue first
    Harness->>Repo: Apply minimal change or produce finding
    Harness->>Checks: Run fmt, clippy, tests, docs when available
    Checks-->>Harness: Verification output
    Harness-->>User: Findings, changes, risks, next actions
```

The harness prioritizes correctness over broad cleanup. It should lead with concrete bugs, soundness risks, panic paths, missing tests, and release hazards before style suggestions.

## Composition

`rust-maintainer` composes well with:

| Cartridge | Use Together For |
|---|---|
| `tdd-architect` | Refactors that need acceptance criteria and test-first sequencing |
| `godot-designer` | Rust tooling around Godot asset pipelines or game support services |

```sh
craft compose rust-maintainer tdd-architect -o craft.compose.toml
craft run craft.compose.toml --prompt "Refactor parser errors with tests first"
```

When composed, the Rust cartridge should still own Rust-specific safety, dependency, MSRV, and verification decisions.

## Verification Contract

Preferred verification for a changed Rust workspace:

```sh
cargo fmt --check
cargo clippy --all-targets -- -D warnings
cargo test --all-targets
cargo test --doc
cargo doc --no-deps
```

If a command cannot run, the harness should say why and keep the claim scoped to what was actually verified.
