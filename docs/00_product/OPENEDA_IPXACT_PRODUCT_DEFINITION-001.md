# OpenEDA IP-XACT Product Definition

| Field | Value |
|---|---|
| Document ID | OPENEDA_IPXACT_PRODUCT_DEFINITION-001 |
| Version | v1.0.0 |
| Status | APPROVED BASELINE |
| Project | OpenEDA IP-XACT |
| Repository | `openeda-ipxact` |
| License | Apache-2.0 |
| Canonical Standard | IEEE 1685-2022 |
| Purpose | Define the product identity, scope, consumers, principles, and external boundaries of the OpenEDA IP-XACT platform |

---

## 0. Purpose

OpenEDA IP-XACT is an independent open-source EDA domain platform for IEEE 1685 IP-XACT metadata.

The project exists to provide a reusable semantic foundation for command-line tools, application/library APIs, CI and automation pipelines, EDA Workbenches, code/documentation generators, service APIs, and future AI/agent tool interfaces.

OpenEDA IP-XACT is not an internal library of a specific Workbench. The EDA Workbench is a consumer of OpenEDA IP-XACT.

## 1. Product Definition

OpenEDA IP-XACT shall provide a standards-oriented semantic platform capable of:

```text
parse
model
validate
query
modify
resolve
serialize
convert
automate
```

The public programming model shall be semantic.

```text
XML
≠ public programming model

IP-XACT Domain
= public semantic model
```

XML is a serialization format and adapter boundary.

## 2. Project Identity

```text
Project Name:
OpenEDA IP-XACT

Repository:
openeda-ipxact

License:
Apache-2.0

Primary Standard:
IEEE 1685-2022
```

The project must remain independently buildable, testable, releasable, and usable without AWorldMagic, the new EDA Workbench, Qt, Kactus, or any graphical runtime.

## 3. Strategic Position

```text
OpenEDA IP-XACT
      │
      ├── EDA Validation
      ├── EDA Schematic
      └── other EDA capabilities
               │
               ▼
          EDA Workbench
```

Dependency direction:

```text
EDA Workbench
→ OpenEDA IP-XACT

OpenEDA IP-XACT
→ EDA Workbench
= FORBIDDEN
```

## 4. Kactus Relationship

Kactus/Kactus2 and its `IPXACTmodels` are not implementation dependencies.

They may be used only as:

```text
reference implementation
coverage reference
behavior reference
compatibility fixture source
migration evidence
```

OpenEDA IP-XACT shall not copy Kactus implementation code, depend on KactusAPI, depend on ProjectLibraryHandler, depend on Kactus-specific UI/application lifecycle, or mirror Kactus architecture merely for compatibility.

Implementation authority:

```text
IEEE 1685-2022
Accellera schemas/supporting material
OpenEDA product requirements
OpenEDA domain architecture
independent compatibility fixtures
```

## 5. Primary Consumers

### CLI
First-class consumer. Long-term command families include `inspect`, `validate`, `query`, `convert`, `format`, `component`, and `design`.

### Library API
Stable semantic API for domain objects and operations.

### EDA Workbench
Consumer of OpenEDA IP-XACT; must not embed a second IP-XACT model.

### CI / Automation
Deterministic non-GUI use in build, validation, packaging, and design automation pipelines.

### Agent Tooling

```text
Agent
  ↓
Tool Facade
  ↓
Semantic Operations API
  ↓
IP-XACT Domain
  ↓
Validation / Serialization
```

Agents must not manipulate XML directly.

## 6. In Scope

```text
IEEE 1685-oriented domain model
IP-XACT XML reader/writer
structural validation
semantic validation
document/reference resolution
query API
mutation API
version compatibility
conversion/migration
CLI
programmatic API
vendor-extension framework
agent-compatible semantic operations
```

## 7. Out of Scope

```text
schematic rendering
wire routing
QGraphicsScene
Qt Widgets
canvas interaction
place-and-route
HDL simulation
synthesis
AWorldMagic-specific UI
EDA Workbench-specific state
product-specific UI policy
```

## 8. Canonical Standard

```text
IEEE 1685-2022
= canonical semantic model

IEEE 1685-2014
= compatibility/import concern

future revisions
= version adapters / migration paths
```

Do not maintain two complete parallel canonical models for 2014 and 2022 unless a future requirement proves it necessary.

## 9. Product Principles

1. **Semantic API First** — users operate on Component, Port, BusInterface, BusDefinition, AbstractionDefinition, Design, ComponentInstance, and Interconnection.
2. **XML Is an Adapter** — parsing/writing is separate from the canonical domain.
3. **Workbench Independence** — no GUI is required.
4. **Deterministic Diagnostics** — validation results are structured and machine-readable.
5. **Controlled Mutation** — mutation occurs through semantic operations or aggregate-controlled APIs.
6. **Vendor Extensions Are First-Class** — generic extension support exists from the start, while product-specific extensions stay outside the core.
7. **Agent Readiness Without LLM Coupling** — stable identifiers, queries, operations, diagnostics, dry-run, and structured results; no LLM SDK/prompt logic in the core.

## 10. v1 Product Scope

### Domain

```text
Common
- VLNV
- identifiers
- references
- vendor extensions

Component
- Component
- Port
- BusInterface
- PortMap
- View
- DesignInstantiation

Bus
- BusDefinition
- AbstractionDefinition

Design
- Design
- ComponentInstance
- Interconnection
- ActiveInterface
- HierInterface

DesignConfiguration
- minimal hierarchy support
```

### Infrastructure

```text
IEEE 1685-2022 XML read/write
structural validation
essential semantic validation
filesystem document resolution
query API
mutation API
CLI
```

### Compatibility

Selected IEEE 1685-2014 import compatibility may be added after the canonical 2022 model is stable.

## 11. Deferred Scope

```text
MemoryMap / AddressBlock / Register / Field
advanced remap/banking
full FileSet/build flow
Generator execution
CPU model
all configurable-element semantics
complete expression language
all 2014 edge cases
all vendor extensions
TGI server
Workbench UI
Schematic routing
HDL parser
code generation
```

## 12. Success Criteria

The platform is successful when it can independently:

1. read an IEEE 1685-2022 component,
2. construct a canonical semantic domain object,
3. query ports and interfaces,
4. resolve referenced definitions,
5. validate semantic consistency,
6. modify the domain through controlled operations,
7. serialize conforming XML,
8. expose the same semantics to CLI and library consumers,
9. support downstream EDA capabilities without embedding Workbench concerns.

## 13. Open-Source Policy

Approved license:

```text
Apache-2.0
```

Recommended repository-level open-source files:

```text
LICENSE
NOTICE
README.md
CONTRIBUTING.md
SECURITY.md
CODE_OF_CONDUCT.md
```

## 14. Final Definition

OpenEDA IP-XACT is an independent Apache-2.0 open-source EDA domain platform for IEEE 1685 IP-XACT metadata, providing canonical semantic modeling, validation, resolution, query/mutation operations, XML serialization, CLI access, and future API/agent integration.

---

**End of Document**
