# OpenEDA IP-XACT Domain Architecture

| Field | Value |
|---|---|
| Document ID | OPENEDA_IPXACT_DOMAIN_ARCHITECTURE-001 |
| Version | v1.0.0 |
| Status | APPROVED / ARCHITECTURE LOCK |
| Project | OpenEDA IP-XACT |
| Repository | `openeda-ipxact` |
| License | Apache-2.0 |
| Canonical Standard | IEEE 1685-2022 |
| Approval Basis | X2 Human Architecture Gate |
| Purpose | Define the canonical semantic domain model, ownership, identity, reference, mutation, validation, extension, and dependency rules |

---

## 0. Architecture Objective

The OpenEDA IP-XACT domain is a semantic model, not an XML DOM model and not a Workbench model.

```text
Domain
≠ XML object tree

Domain
≠ Workbench state

Domain
≠ Kactus clone
```

The domain must remain usable by CLI, services, agents, validators, schematic engines, and future EDA applications.

## 1. High-Level Architecture

```text
                 IP-XACT Domain
                       │
        ┌──────────────┼───────────────┐
        ▼              ▼               ▼
   Validation      Resolution      Operations
        │              │               │
        └──────────────┼───────────────┘
                       │
             Serialization Adapters
                       │
                  IP-XACT XML
```

External consumers:

```text
CLI
Library API
Agent facade
EDA Validation
EDA Schematic
EDA Workbench
```

## 2. Identity Model

### Document Identity

Document-level roots use VLNV.

```text
VLNV
{
    vendor
    library
    name
    version
}
```

VLNV is immutable.

Document aggregate roots:

```text
Component
BusDefinition
AbstractionDefinition
Design
DesignConfiguration
```

### Local Identity

Owned entities use aggregate-scoped identifiers:

```text
Component.Port
Component.BusInterface
Design.ComponentInstance
Design.Interconnection
AbstractionDefinition.LogicalPort
```

They do not receive global VLNVs.

## 3. Reference Model

References are explicit domain concepts. Persistent identity must not be raw object pointers.

Conceptual categories:

```text
VlnvReference<T>
PortReference
BusInterfaceReference
ComponentInstanceReference
LogicalPortReference
```

Resolution is separate:

```text
Reference
   │
   ▼
Resolver
   │
   ▼
Resolved Domain Object
```

This permits filesystem, Workbench, remote, service-backed, and in-memory repositories.

## 4. Aggregate Roots

```text
Component
BusDefinition
AbstractionDefinition
Design
DesignConfiguration
```

Aggregate boundaries define ownership and mutation. Cross-aggregate relationships use references.

## 5. Component Aggregate

```text
Component
{
    identity: VLNV
    metadata
    ports
    busInterfaces
    views
    designInstantiations
    bounded parameters
    vendorExtensions
}
```

Ownership:

```text
Component
 ├─ owns Port
 ├─ owns BusInterface
 ├─ owns View
 └─ owns DesignInstantiation
```

External relationships remain references.

## 6. Port

A Port is a physical signal endpoint owned by a Component.

```text
Port
{
    localIdentity
    direction
    width/shape information
    presence/basic metadata
    type/wire information
    vendorExtensions
}
```

Canonical identity:

```text
Component VLNV
+
Port local identity
```

A Port is not a schematic graphics object.

## 7. BusInterface

```text
BusInterface
{
    localIdentity
    BusDefinitionRef
    AbstractionDefinitionRef*
    interfaceMode / role
    portMaps
    vendorExtensions
}
```

Relationships:

```text
BusInterface
→ BusDefinition

BusInterface
→ AbstractionDefinition

PortMap
→ Component.Port
→ AbstractionDefinition logical port
```

## 8. PortMap

`PortMap` is a relationship object.

```text
PortMap
{
    logicalPortRef
    physicalPortRef
    mapping information
}
```

It owns neither endpoint.

## 9. BusDefinition

```text
BusDefinition
{
    VLNV
    protocol properties
    compatibility properties
    addressing capability
    vendorExtensions
}
```

Role:

```text
BusDefinition
= protocol semantics
```

## 10. AbstractionDefinition

```text
AbstractionDefinition
{
    VLNV
    BusDefinitionRef
    logicalPorts
    wire/transactional abstraction information
    vendorExtensions
}
```

Role:

```text
AbstractionDefinition
= representation semantics
```

## 11. Design Aggregate

```text
Design
{
    VLNV
    componentInstances
    interconnections
    optional adHocConnections
    vendorExtensions
}
```

Design does not own schematic paint, path, selection, or canvas state.

## 12. ComponentInstance

```text
ComponentInstance
{
    localInstanceId
    ComponentRef
    parameter/configuration overrides
    vendorExtensions
}
```

Identity:

```text
Design identity
+
instanceId
```

Component reference and instance identity are distinct.

## 13. Interconnection

```text
Interconnection
{
    localConnectionId
    endpoints
    metadata
    vendorExtensions
}
```

Conceptual endpoint:

```text
InterfaceEndpoint
{
    componentInstanceRef
    busInterfaceRef
}
```

Hierarchy endpoints may use separate typed references.

The canonical Interconnection must not hold `QGraphicsItem`, `SchematicWireItem`, raw `Port*`, or raw `BusInterface*`.

## 14. DesignConfiguration

The v1 subset supports hierarchy resolution.

```text
DesignConfiguration
{
    VLNV
    DesignRef
    instanceViewSelections
    vendorExtensions
}
```

Advanced abstractor/configuration behavior is deferred unless required by proven v1 needs.

## 15. Mutability Policy

### Immutable Value Objects

```text
VLNV
Identifiers
References
```

### Controlled Mutable Aggregate Roots

```text
Component
BusDefinition
AbstractionDefinition
Design
DesignConfiguration
```

### Owned Entities

```text
Port
BusInterface
PortMap
ComponentInstance
Interconnection
```

Prefer aggregate-controlled APIs such as:

```text
component.addPort(...)
design.addInstance(...)
design.connect(...)
```

Avoid unrestricted public mutable collections.

## 16. Resolution Architecture

Domain entities do not know about filesystem, Workbench, database, or network topology.

Conceptual contract:

```text
DocumentRepository
resolve(VLNV)
find(type, VLNV)
```

Possible implementations:

```text
FilesystemRepository
WorkspaceRepository
RemoteRepository
InMemoryRepository
```

## 17. Validation Architecture

### L1 — Object Invariants

```text
invalid identifier
duplicate local identity
invalid VLNV construction
```

### L2 — Document Semantic Validation

```text
PortMap references missing physical port
duplicate BusInterface
invalid Component-local relationship
```

### L3 — Cross-Document Validation

```text
missing BusDefinition
missing AbstractionDefinition
missing Component
unresolved Design
incompatible interconnection endpoints
```

## 18. Diagnostic Model

Validation returns structured diagnostics, not only booleans.

```text
Diagnostic
{
    code
    severity
    documentIdentity
    objectPath
    message
    relatedReferences
    optionalRemediationMetadata
}
```

Example codes:

```text
IPXACT.COMPONENT.DUPLICATE_PORT
IPXACT.PORTMAP.PHYSICAL_PORT_NOT_FOUND
IPXACT.INTERCONNECTION.INTERFACE_NOT_FOUND
IPXACT.BUS.INTERFACE_INCOMPATIBLE
```

Diagnostics are shared by CLI, IDE, Workbench, CI, and Agent consumers.

## 19. Error Model

Conceptual categories:

```text
INVALID_ARGUMENT
NOT_FOUND
DUPLICATE_IDENTITY
UNRESOLVED_REFERENCE
SEMANTIC_CONFLICT
VALIDATION_FAILED
UNSUPPORTED_STANDARD_FEATURE
```

Concrete representation is implementation-language specific.

## 20. Vendor Extensions

Vendor extensions are a first-class generic capability.

```text
VendorExtension
{
    namespace
    identity
    payload
}
```

Optional extension infrastructure may include:

```text
ExtensionCodec
ExtensionRegistry
```

No AWorldMagic or Workbench extension is hard-coded in the core domain.

## 21. Serialization Boundary

Forbidden in the canonical domain:

```text
XML element types
QDomNode
QXmlStreamWriter
XPath state
parser callbacks
```

Architecture:

```text
XML
 ↓
Reader / XML DTO
 ↓
Mapping
 ↓
Canonical Domain
```

and:

```text
Canonical Domain
 ↓
Writer Mapping
 ↓
XML
```

## 22. Version Boundary

Canonical semantics are IEEE 1685-2022-oriented.

```text
Reader2014
     ↓
compatibility mapping
     ↓
Canonical Domain
     ↑
Reader2022
```

Conversion loss or ambiguity produces structured migration diagnostics.

## 23. Operations Layer

Semantic operation examples:

```text
CreateComponent
AddPort
AddBusInterface
BindPortMap
InstantiateComponent
ConnectInterfaces
DisconnectInterfaces
RenameInstance
```

Generic flow:

```text
request
→ resolve
→ validate
→ mutate
→ result
```

This layer is shared by CLI, API, Workbench, and future agents.

## 24. Agent-Ready Architecture

Agent readiness comes from:

```text
stable identifiers
structured diagnostics
semantic queries
bounded mutation operations
deterministic validation
machine-readable results
dry-run support
diff/result reporting
```

No LLM-specific SDK or prompt logic belongs in the domain.

## 25. Dependency on Schematic

Allowed:

```text
EDA/Schematic
→ OpenEDA IP-XACT
```

Forbidden:

```text
OpenEDA IP-XACT
→ EDA/Schematic
```

OpenEDA IP-XACT must not know about wire, route, canvas, selection, paint, or graphics scene state.

## 26. IP-XACT Validation vs EDA Validation

```text
OpenEDA IP-XACT Validation
= standard/domain semantic consistency

EDA Validation
= schematic/product/tool policy validation
```

These remain separate capabilities.

## 27. v1 Object Graph

```text
Component
├── Port*
├── BusInterface*
│      ├── BusDefinitionRef
│      ├── AbstractionDefinitionRef*
│      └── PortMap*
│             ├── LogicalPortRef
│             └── PhysicalPortRef
├── View*
└── DesignInstantiation*
       └── DesignRef

BusDefinition
└── protocol semantics

AbstractionDefinition
├── BusDefinitionRef
└── LogicalPort*

Design
├── ComponentInstance*
│      └── ComponentRef
└── Interconnection*
       └── InterfaceEndpoint*

DesignConfiguration
├── DesignRef
└── InstanceViewSelection*
```

## 28. Architecture Constraints

### REQUIRED

```text
semantic domain independent from XML
explicit reference model
aggregate-controlled mutation
structured diagnostics
cross-document resolution boundary
generic vendor-extension mechanism
```

### FORBIDDEN

```text
Qt Widgets dependency
QGraphics dependency
Workbench dependency
AWorldMagic dependency
Kactus implementation dependency
XML DOM as canonical domain
Schematic routing state inside IP-XACT domain
```

## 29. Architecture Lock

```text
CANONICAL STANDARD
= IEEE 1685-2022

DOMAIN
= semantic

IDENTITY
= VLNV for documents
= local scoped identity for owned entities

RELATIONSHIPS
= explicit references

RESOLUTION
= separate capability

MUTATION
= aggregate-controlled

VALIDATION
= invariant + document + cross-document

SERIALIZATION
= adapter

VENDOR EXTENSIONS
= generic first-class mechanism

SCHEMATIC
= downstream consumer

WORKBENCH
= downstream consumer

KACTUS
= reference only
```

---

**End of Document**
