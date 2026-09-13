# OpenEDA IP-XACT Standards Reference & Source Authority

Document ID: OPENEDA_IPXACT_STANDARDS_REFERENCE-001
Version: v1.0.0
Status: PROPOSED — SOURCE AUTHORITY BASELINE

## 1. Purpose

This document defines the standards and official reference material used
to design, implement, validate, and review OpenEDA IP-XACT.

It also identifies which implementation decisions come directly from
the governing standard and which are OpenEDA architecture decisions.

## 2. Canonical Standard

The canonical standard for OpenEDA IP-XACT is:

IEEE 1685-2022
IEEE Standard for IP-XACT, Standard Structure for Packaging,
Integrating, and Reusing IP within Tool Flows.

OpenEDA IP-XACT uses IEEE 1685-2022 as its primary semantic baseline.

## 3. Official Machine-Readable Reference

The official Accellera IEEE 1685-2022 XML Schema set is the primary
machine-readable reference for IP-XACT XML representation.

The schema material is used to determine:

- XML vocabulary
- XML structure
- datatypes
- occurrence constraints
- schema-valid document structure

The OpenEDA semantic domain model remains an architectural representation
of IP-XACT concepts rather than an XML document model.

## 4. Official Supplemental Reference Material

The following official material may be consulted when implementing
and reviewing OpenEDA IP-XACT:

- IEEE 1685-2022 Release Notes
- Accellera IP-XACT User Guide
- Accellera Recommended Vendor Extensions
- Accellera published issue material
- Accellera official examples
- applicable Accellera TGI material

These materials provide additional implementation and usage context
for the IEEE 1685-2022 standard.

## 5. OpenEDA Internal Architecture Authority

The following OpenEDA documents define the product and software architecture:

1. OPENEDA_IPXACT_PRODUCT_DEFINITION-001
2. OPENEDA_IPXACT_DOMAIN_ARCHITECTURE-001
3. OPENEDA_IPXACT_IMPLEMENTATION_ARCHITECTURE-001
4. OPENEDA_IPXACT_IMPLEMENTATION_ROADMAP-001
5. OPENEDA_IPXACT_STANDARDS_REFERENCE-001

These documents define OpenEDA choices such as:

- domain boundaries
- aggregate structure
- API shape
- module boundaries
- crate responsibilities
- diagnostics representation
- implementation sequencing

## 6. Source Authority Order

When implementing a standards-sensitive concept, use the following order:

1. IEEE 1685-2022
2. official Accellera IEEE 1685-2022 schema material
3. official Accellera supplemental material
4. approved OpenEDA architecture documents

This order provides the reference path for design and implementation review.

## 7. Evidence Classification

Standards-sensitive implementation decisions should be identifiable as one
of the following:

### NORMATIVE

Derived directly from IEEE 1685-2022.

### SCHEMA-DERIVED

Derived from the official IEEE 1685-2022 XML schema material.

### OFFICIAL-GUIDANCE

Derived from official Accellera supplemental material.

### OPENEDA-DESIGN

An OpenEDA architecture or API decision made within the implementation space
provided by the standard.

### UNRESOLVED

A standards interpretation or implementation decision that requires further
review before it is fixed in the implementation.

## 8. Schema Version Tracking

When official schema material is introduced into the repository or used as
a versioned implementation input, the following information should be
recorded:

- upstream source
- schema version or publication identifier
- retrieval date
- integrity information where applicable

This allows implementation results to be traced to the exact official
schema material used.

## 9. Current Roadmap Application

This source-authority baseline applies beginning with:

X4-B — Common Domain Foundation.

X4-B design and implementation should identify whether each material
semantic decision comes from:

- IEEE 1685-2022
- official schema material
- official supplemental material
- OpenEDA architecture

This applies to common domain concepts including:

- Identifier
- VLNV
- DocumentIdentity
- scoped and local identities
- reference primitives
- VendorExtension
- diagnostics
- errors
- common invariants

## 10. Decision

OpenEDA IP-XACT uses IEEE 1685-2022 as its canonical standard baseline.

Official Accellera material provides the machine-readable and supplemental
reference set.

Approved OpenEDA architecture documents define the software architecture
used to implement those standards concepts.

Implementation reviews should preserve a clear distinction between
standards-derived semantics and OpenEDA design decisions.