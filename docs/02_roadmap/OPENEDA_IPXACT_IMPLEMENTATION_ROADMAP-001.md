# OpenEDA IP-XACT Implementation Roadmap

| Field | Value |
|---|---|
| Document ID | OPENEDA_IPXACT_IMPLEMENTATION_ROADMAP-001 |
| Version | v1.0.0 |
| Status | APPROVED / ROADMAP LOCK |
| Project | OpenEDA IP-XACT |
| Repository | `openeda-ipxact` |
| License | Apache-2.0 |
| Approval Basis | X4 Human Architecture Gate |
| Purpose | Define implementation sequence, first vertical slice, exclusions, milestone completion criteria, and downstream integration order |

---

## 0. Roadmap Objective

Implementation proceeds by small end-to-end vertical slices.

Do not implement the full IP-XACT domain before delivering a usable parser/validator/CLI path.

Preferred pattern:

```text
Domain
→ XML
→ Validation
→ CLI
```

for a narrow semantic subset, then expand.

## 1. Roadmap Overview

```text
X4-A
Repository Foundation

X4-B
Common Domain Foundation

X4-C
Component / Port Vertical Slice

X4-D
IEEE 1685-2022 Component XML Read/Write

X4-E
Validation + CLI Vertical Slice

-------------------------------
FIRST USABLE RELEASE POINT
-------------------------------

X4-F
BusDefinition / AbstractionDefinition

X4-G
BusInterface / PortMap

X4-H
Resolution Layer

X4-I
Design / ComponentInstance

X4-J
Interconnection

X4-K
DesignConfiguration / Hierarchy

X4-L
IEEE 1685-2014 Compatibility Import

X4-M
Public C ABI

X4-N
EDA Platform Integration

X4-O
Python / Service / Agent Facades
```

## 2. X4-A — Repository Foundation

### Purpose

Create an independently buildable/testable open-source Rust workspace.

### Initial logical structure

```text
openeda-ipxact/
├─ Cargo.toml
├─ Cargo.lock
│
├─ crates/
│  ├─ ipxact-core/
│  ├─ ipxact-xml/
│  ├─ ipxact-validation/
│  └─ ipxact-operations/
│
├─ apps/
│  └─ ipxact-cli/
│
├─ tests/
│  └─ fixtures/
│
├─ docs/
│
├─ LICENSE
├─ NOTICE
├─ README.md
├─ CONTRIBUTING.md
├─ SECURITY.md
└─ CODE_OF_CONDUCT.md
```

### Completion criteria

```text
cargo build
= PASS

cargo test
= PASS

cargo fmt --check
= PASS

cargo clippy
= PASS

CI
= Linux + Windows + macOS configured
```

No IP-XACT feature is required yet.

## 3. X4-B — Common Domain Foundation

### Scope

```text
VLNV
Identifier / LocalName
DocumentIdentity
Reference primitive
VendorExtension primitive
Diagnostic primitives
DomainError / Result model
```

### VLNV requirements

```text
construction
validation
equality
hash/ordering where appropriate
string representation
parse/format round trip
```

### Forbidden dependencies

```text
XML
filesystem
CLI
Qt
Workbench
AWorldMagic
Kactus
```

## 4. X4-C — Component / Port Domain Slice

### Domain

```text
Component
└─ Port*
```

### Minimum Component

```text
identity: VLNV
displayName optional
description optional
ports
vendorExtensions
```

### Minimum Port

```text
local identity
direction
description optional
basic presence/metadata
vendorExtensions
```

### Mutation API

Examples:

```text
Component::add_port(...)
Component::remove_port(...)
Component::port(...)
Component::ports()
```

### Completion criteria

```text
duplicate port rejection
invalid identity rejection
stable lookup
structured diagnostics
unit tests
```

## 5. X4-D — IEEE 1685-2022 Component XML Slice

### Initial supported semantic content

```text
Component VLNV
Component metadata
Ports
Basic Port direction
Vendor extension preservation boundary
```

### Architecture

```text
XML
 ↓
Reader / XML DTO
 ↓
Mapping
 ↓
Domain Component
```

and:

```text
Domain Component
 ↓
Writer Mapping
 ↓
XML
```

### Completion criteria

```text
1685-2022 component fixture parse
= PASS

Domain semantic equivalence
= PASS

Domain → XML write
= PASS

XML → Domain → XML → Domain
= SEMANTIC ROUND-TRIP PASS
```

Byte-identical XML is not required.

## 6. X4-E — Validation + CLI First Vertical Slice

This stage closes the first usable product slice.

### Validation

```text
invalid VLNV
duplicate port
invalid local identifier
missing required Component identity
basic malformed state/reference
```

### CLI

```text
ipxact inspect <component.xml>

ipxact validate <component.xml>
```

### Example inspect output concept

```text
Component
vendor: acme
library: peripherals
name: uart
version: 1.0

Ports: 4
Validation: PASS
```

### Example diagnostic concept

```text
ERROR IPXACT.COMPONENT.DUPLICATE_PORT
object: ports/clk
message: duplicate port identity
```

### Definition of Done

Workbench-independent operation proves:

```text
IEEE 1685-2022 Component XML
→ canonical Domain
→ validation
→ semantic inspection
→ XML serialization
→ semantic round-trip
```

through the same core used by the CLI.

## 7. First Vertical Slice — Exact Logical Scope

### IN

```text
Repository bootstrap
Rust Cargo workspace

ipxact-core
ipxact-xml
ipxact-validation
ipxact-operations
ipxact-cli

VLNV
Identifier
DocumentIdentity
Reference primitive
VendorExtension primitive
Diagnostic / error primitives

Component
Port

Component invariant validation

IEEE 1685-2022 XML Component read
IEEE 1685-2022 XML Component write

Semantic round-trip tests

CLI:
- inspect
- validate
```

### OUT

```text
BusInterface
PortMap
BusDefinition
AbstractionDefinition
Design
ComponentInstance
Interconnection
DesignConfiguration
IEEE 1685-2014 compatibility
MemoryMap/Register
C ABI
Python
REST/gRPC
MCP/Agent
EDA Workbench integration
```

No out-of-scope capability may be pulled into the first implementation unit merely for convenience.

## 8. X4-F — BusDefinition / AbstractionDefinition

Add:

```text
BusDefinition
AbstractionDefinition
LogicalPort
BusDefinitionRef
```

Completion criteria:

```text
bus document parse/write
abstraction document parse/write
abstraction → bus definition resolution request
basic semantic validation
```

## 9. X4-G — BusInterface / PortMap

Extend Component:

```text
Component
├─ Port
└─ BusInterface
      └─ PortMap
```

Relationships:

```text
BusInterface
→ BusDefinitionRef

BusInterface
→ AbstractionDefinitionRef

PortMap
→ LogicalPortRef
→ PhysicalPortRef
```

Completion means protocol semantics, logical interface, and physical port mapping are connected.

## 10. X4-H — Resolution Layer

Introduce:

```text
DocumentRepository trait
FilesystemRepository

resolve(VLNV)

resolve_component(...)
resolve_bus_definition(...)
resolve_abstraction_definition(...)
```

Completion criteria:

```text
multiple document loading
cross-document VLNV resolution
missing-reference diagnostic
ambiguous-reference diagnostic
```

Domain remains independent from filesystem layout.

## 11. X4-I — Design / ComponentInstance

Add:

```text
Design
└─ ComponentInstance*
       └─ ComponentRef
```

Initial scope:

```text
Design identity
ComponentInstance identity
Component VLNV reference
instance lookup
reference resolution
```

Connections are intentionally deferred to X4-J.

## 12. X4-J — Interconnection

Add:

```text
Design
├─ ComponentInstance*
└─ Interconnection*
```

Endpoint semantic references:

```text
componentInstanceRef
busInterfaceRef
```

Completion criteria:

```text
two component instances resolved
BusInterfaces resolved
Interconnection created
missing endpoint rejected
incompatible bus definition diagnostic
```

This creates the minimum design graph required by downstream Schematic consumers.

## 13. X4-K — DesignConfiguration / Hierarchy

v1 subset:

```text
DesignConfiguration
DesignRef
InstanceViewSelection
basic hierarchy resolution
```

Advanced abstractor behavior remains deferred unless required by evidence.

## 14. X4-L — IEEE 1685-2014 Compatibility Import

```text
2014 XML
  ↓
Reader2014
  ↓
Compatibility Mapping
  ↓
Canonical 2022-oriented Domain
```

Conversion ambiguity/loss produces:

```text
MigrationDiagnostic
```

The canonical internal domain remains 2022-oriented.

## 15. X4-M — Public C ABI

Requirements:

```text
opaque handles
explicit ownership
structured result/error
no Rust layout exposure
```

Future integration:

```text
C++ RAII wrapper
→ C ABI
→ Rust core
```

## 16. X4-N — EDA Platform Integration

Projection examples:

```text
IP-XACT ComponentInstance
       ↓
Schematic Component Projection

IP-XACT BusInterface
       ↓
Schematic Endpoint Projection

IP-XACT Interconnection
       ↓
Schematic Connection Projection
```

AWorldMagic XML fixtures may be used as compatibility evidence, not implementation source.

## 17. X4-O — Python / Service / Agent

Recommended order:

```text
Python binding
→ optional service API
→ Agent / MCP tools
```

All facades reuse the same Operations API.

```text
Agent
 ↓
Operations
 ↓
Domain
```

## 18. Release Capability Progression

Indicative progression:

```text
0.1
Component + Port
XML read/write
Validation
CLI

0.2
BusDefinition
AbstractionDefinition
BusInterface
PortMap
Resolution

0.3
Design
ComponentInstance
Interconnection

0.4
DesignConfiguration
Hierarchy

0.5
2014 compatibility

0.6
C ABI / C++ integration

0.7
EDA Schematic integration

0.8+
Python / service / agent

1.0
stable semantic API
documented compatibility commitment
```

Exact version numbers may change, but capability ordering remains unless Human approval changes the roadmap.

## 19. Fixture Strategy

### A. Synthetic

Minimal cases created specifically for tests.

### B. Official / Reference

IEEE/Accellera-compatible examples and schema-driven fixtures.

### C. Product Compatibility

Existing AWorldMagic/Kactus-generated XML used as behavior/compatibility evidence.

Kactus implementation source is not copied.

## 20. Common Quality Gate

Each implementation unit applies the relevant subset of:

```text
Domain semantics test
Serialization test
Validation diagnostic test
Public API behavior test
CLI behavior test
Forbidden dependency check
cargo fmt --check
cargo clippy
cargo test
```

GUI testing is not part of OpenEDA IP-XACT.

## 21. First Vertical Slice Result Contract

Given:

```text
IEEE 1685-2022 Component XML
```

When:

```text
ipxact inspect component.xml
```

Then:

```text
canonical Component domain object created
VLNV displayed
Port list displayed
validation diagnostics displayed
```

And:

```text
XML
→ Domain
→ XML
→ Domain
```

must produce:

```text
semantic equivalence = PASS
```

## 22. Implementation Readiness

Already locked:

```text
Project
= OpenEDA IP-XACT

Repository
= openeda-ipxact

License
= Apache-2.0

X2
= APPROVED

X3
= APPROVED

X4
= APPROVED
```

Remaining pre-implementation work is limited to exact initial file manifest and repository bootstrap commands.

## 23. Roadmap Lock

```text
FIRST IMPLEMENTATION UNIT
= Repository Foundation
+ Common/VLNV
+ Component/Port
+ 1685-2022 Component XML read/write
+ Validation
+ CLI inspect/validate

Workbench integration
= later

Agent integration
= later facade
```

---

**End of Document**
