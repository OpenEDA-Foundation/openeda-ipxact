# OpenEDA IP-XACT Implementation Architecture

| Field | Value |
|---|---|
| Document ID | OPENEDA_IPXACT_IMPLEMENTATION_ARCHITECTURE-001 |
| Version | v1.0.0 |
| Status | APPROVED / IMPLEMENTATION ARCHITECTURE LOCK |
| Project | OpenEDA IP-XACT |
| Repository | `openeda-ipxact` |
| License | Apache-2.0 |
| Approval Basis | X3 Human Architecture Gate |
| Purpose | Define the implementation technology, repository structure, dependency policy, API boundaries, testing, CI, release, and interoperability strategy |

---

## 0. Technology Decision

```text
Implementation Language
= Rust

Rust Edition
= 2024

Toolchain
= Stable only

Build / Package
= Cargo

Repository Model
= Single Git repository
+ Cargo workspace
```

No nightly-only language feature or dependency is required by policy.

## 1. Why Rust

The project requirements include:

```text
independent open-source library
CLI
semantic domain model
XML parser/writer
validation
resolution
cross-platform support
future C ABI
future Python bindings
future service/API
future agent tooling
```

Rust is selected because it fits a memory-safe long-lived domain core, explicit ownership/reference discipline, Cargo workspace/package management, CLI delivery, cross-platform builds, FFI, Python bindings, and later service expansion.

This decision does not require the EDA Workbench UI to be implemented in Rust.

## 2. Workbench Integration Boundary

```text
Rust OpenEDA IP-XACT
        │
        ├─ Rust native API
        │
        └─ FFI facade
               │
               ▼
             C ABI
               │
               ▼
           C++ facade
               │
               ▼
        EDA Workbench / Qt
```

The C++ Workbench shall not depend on Rust internal layout or compiler ABI.

## 3. Repository Architecture

Target repository:

```text
openeda-ipxact/
```

Recommended mature logical shape:

```text
openeda-ipxact/
├─ Cargo.toml
├─ Cargo.lock
│
├─ crates/
│  ├─ ipxact-core/
│  ├─ ipxact-xml/
│  ├─ ipxact-validation/
│  ├─ ipxact-resolution/
│  ├─ ipxact-operations/
│  ├─ ipxact-api/
│  └─ ipxact-ffi/
│
├─ apps/
│  └─ ipxact-cli/
│
├─ bindings/
│  └─ python/
│
├─ tests/
│  ├─ fixtures/
│  ├─ conformance/
│  └─ compatibility/
│
├─ schemas/
├─ docs/
├─ examples/
├─ LICENSE
├─ NOTICE
├─ README.md
├─ CONTRIBUTING.md
├─ SECURITY.md
└─ CODE_OF_CONDUCT.md
```

This is a target logical architecture, not a requirement to create every crate immediately.

## 4. Initial Physical Package Set

Start with:

```text
ipxact-core
ipxact-xml
ipxact-validation
ipxact-operations
ipxact-cli
```

Further physical separation occurs only when ownership/dependency pressure justifies it. Logical module boundaries do not need to map one-to-one to crates from day one.

## 5. Core Dependency Rule

`ipxact-core` is the foundation.

Allowed:

```text
Rust standard library
small value-oriented dependencies when justified
```

Forbidden:

```text
XML parser
CLI framework
HTTP server
Python runtime
Qt
Workbench headers
database implementation
filesystem repository implementation
AI/LLM SDK
Kactus code
AWorldMagic code
```

Dependency direction:

```text
ipxact-xml
ipxact-validation
ipxact-operations
ipxact-ffi
ipxact-cli
      ↓
ipxact-core
```

Reverse dependencies are forbidden.

## 6. XML Implementation Strategy

XML serialization is isolated in `ipxact-xml`.

A candidate parser/writer ecosystem may include `quick-xml`, subject to implementation-time review.

The canonical domain shall not be bound directly to XML through blanket serialization derives.

Preferred mapping:

```text
XML
 ↓
XML parser / DTO
 ↓
mapping
 ↓
Domain
```

and:

```text
Domain
 ↓
writer mapping
 ↓
XML
```

This isolates namespace handling, version differences, vendor extensions, preservation, and ordering concerns.

## 7. CLI Architecture

The CLI is a first-class product.

Conceptual executable:

```text
ipxact
```

Initial commands:

```text
inspect
validate
```

Later command families:

```text
query
convert
format
component
design
```

Required call path:

```text
CLI
 ↓
Operations
 ↓
Domain
```

CLI must not directly mutate XML nodes.

## 8. Library API

The primary native API is Rust.

API principles:

```text
semantic concepts
controlled mutation
structured result
structured diagnostics
stable identities
```

Prefer private fields plus constructors, read/query methods, and semantic mutation methods. Avoid exposing all internal struct fields publicly.

## 9. Public C ABI

The C ABI is deferred until the semantic model is stable enough.

Requirements:

```text
opaque handles
explicit ownership
explicit result/error objects
versioned functions where required
```

Rust object layout is not exposed.

A future C++ RAII facade wraps the C ABI.

## 10. Python Binding

Python is a later consumer, not a v1 blocker.

Candidate stack:

```text
PyO3
maturin
```

Bindings shall call the same semantic core used by CLI and other consumers.

## 11. Service API

REST/gRPC/server infrastructure is deferred.

Architecture:

```text
Service
 ↓
Operations API
 ↓
Domain
```

No service framework belongs in the domain.

## 12. Agent API

Agent/MCP/tool interfaces are deferred facades.

```text
Agent / MCP
    ↓
Tool facade
    ↓
Operations API
    ↓
Domain
```

Core must not contain prompt logic, LLM SDKs, or provider-specific AI logic.

## 13. Versioning Policy

Use Semantic Versioning.

```text
0.x
= development/evolution

1.0.0+
= stable public semantic API commitment
```

Breaking public API changes must be deliberate.

## 14. Rust Version Policy

Use stable Rust only.

Repository shall declare an explicit minimum supported Rust version once concrete dependency requirements are known.

```text
rust-version
= explicit

nightly
= forbidden by default
```

## 15. Dependency Policy

```text
minimize core dependencies
isolate specialized dependencies by crate
avoid wildcard dependency ranges
avoid Git-only release dependencies
avoid nightly-only dependencies
review license compatibility
```

Placement:

```text
XML dependency
→ ipxact-xml

CLI dependency
→ ipxact-cli

FFI dependency
→ ipxact-ffi

Python dependency
→ binding package
```

## 16. Test Architecture

### Unit
`VLNV`, identifiers, references, aggregate invariants.

### Domain
Component operations, Design operations, semantic validation.

### Conformance
Official/reference IEEE/Accellera-compatible XML fixtures.

### Compatibility
Existing AWorldMagic/Kactus-generated XML as behavior/compatibility evidence.

Kactus source is not copied.

## 17. Round-Trip Policy

Primary requirement:

```text
XML
→ Domain
→ XML
→ Domain
```

Success criterion:

```text
semantic equivalence
```

Byte-identical XML is not required by default.

Additional preservation tests may cover unknown vendor extensions, namespaces, and ordering where materially required.

## 18. CI

Target platforms:

```text
Linux
Windows
macOS
```

Minimum gates:

```text
cargo fmt --check
cargo clippy
cargo test
cargo doc
```

Workbench build is not part of OpenEDA IP-XACT CI.

## 19. Release Artifacts

Long-term releases may include:

```text
Rust crates
CLI binaries
C API headers/libraries
C++ facade package
Python wheels
documentation
```

Initial releases need not provide all surfaces.

## 20. Git Policy

```text
main
= releasable / CI-passing

feature branches
= pull requests

tags
= releases
```

No heavy GitFlow is required.

## 21. Documentation Policy

Minimum durable documentation:

```text
README
Product Definition
Domain Architecture
Implementation Architecture
Implementation Roadmap
CLI Reference
API Reference
Conformance / Compatibility
Contributing
Security
```

Avoid documentation proliferation.

## 22. Open-Source License

Approved:

```text
Apache-2.0
```

Repository shall preserve clear provenance and contribution boundaries.

## 23. Independence Test

This question must always return YES:

> Can OpenEDA IP-XACT be built, tested, released, and used without the EDA Workbench repository?

If any of these become mandatory, architecture has regressed:

```text
Qt
AWorldMagic
Workbench headers
Kactus runtime
GUI session
```

## 24. Implementation Architecture Lock

```text
LANGUAGE
= Rust

EDITION
= 2024

TOOLCHAIN
= stable

BUILD
= Cargo

REPOSITORY
= single repository / Cargo workspace

CORE MODEL
= IEEE 1685-2022-oriented semantic domain

XML
= separate serialization layer

CLI
= first-class

NATIVE API
= Rust library API

C++ INTEGRATION
= future stable C ABI + C++ facade

PYTHON
= future binding

SERVICE
= future facade

AGENT
= future facade over semantic operations

QT
= forbidden

WORKBENCH DEPENDENCY
= forbidden

AWORLDMAGIC DEPENDENCY
= forbidden

KACTUS IMPLEMENTATION DEPENDENCY
= forbidden

LICENSE
= Apache-2.0
```

---

**End of Document**
