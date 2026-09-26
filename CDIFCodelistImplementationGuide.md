---
title: CDIF Codelist Profile Classes and Properties
date: 2026-09-26
---

# Introduction

This document describes all classes and properties for the CDIF Codelist profile, which defines how classification schemes used to populate data values are represented as SKOS ConceptSchemes in JSON-LD. The profile composes the base SKOS ConceptScheme and Concept building blocks with CDIF-specific constraints: CDIF core metadata properties for the scheme, and notation (code) values and labels for all concepts. If the terms in the codelist are hierarchical, both broader and narrower relations must be asserted.

The implementation uses the [SKOS (Simple Knowledge Organization System)](https://www.w3.org/TR/skos-reference/) vocabulary with JSON-LD serialization. This profile aligns with the approach described in ['Modelling of Eurostat's Statistical Classifications in ShowVoc'](https://cros.ec.europa.eu/book-page/modeling-eurostats-statistical-classifications-showvoc).

This guide is organized by **class**: each class the profile defines gets a section, and every property the schema assigns to that class is documented under it. The authoritative source is `_sources/profiles/cdifProfile/cdifCodelist/schema.yaml` in the [metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks) register.

# Table of contents

- [Namespaces](#namespaces)
- [Classes](#classes)
  - [skos:ConceptScheme](#skosconceptscheme)
  - [CdifCodelistConcept](#cdifcodelistconcept)
  - [schema:Dataset (the catalog record)](#schemadataset-the-catalog-record)
- [Value types](#value-types)
  - [Object reference](#object-reference)
  - [schema:PropertyValue](#schemapropertyvalue)
- [Language tagging](#language-tagging)
- [Bidirectional hierarchy](#bidirectional-hierarchy)
- [Array convention](#array-convention)
- [SKOS properties beyond this profile](#skos-properties-beyond-this-profile)
- [Validation](#validation)

# Namespaces

[^ Back to TOC](#table-of-contents)

```json
"@context": {
  "skos": "http://www.w3.org/2004/02/skos/core#",
  "schema": "http://schema.org/",
  "dcterms": "http://purl.org/dc/terms/",
  "dcat": "http://www.w3.org/ns/dcat#"
}
```

Note that `schema` binds to `http://schema.org/` — the `http` form, not `https`. The two are different IRIs and only the `http` form is recognized here.

# Classes

[^ Back to TOC](#table-of-contents)

The profile defines three classes.

| class | `@type` token | role |
|---|---|---|
| [skos:ConceptScheme](#skosconceptscheme) | `skos:ConceptScheme` | The codelist itself. The document root. |
| [CdifCodelistConcept](#cdifcodelistconcept) | *not constrained — see below* | One term or code in the codelist. |
| [schema:Dataset](#schemadataset-the-catalog-record) | `schema:Dataset` + `dcat:CatalogRecord` | The catalog record that describes the codelist as CDIF metadata. The value of `schema:subjectOf`. |

Values that are structures rather than classes in their own right — object references, `schema:PropertyValue` — are described under [Value types](#value-types).

## skos:ConceptScheme

[^ Back to TOC](#table-of-contents)

The root object representing the codelist or classification scheme.

| property | cardinality | content |
|---|---|---|
| `@id` | **Required** | string (URI) |
| `skos:prefLabel` | **Required** | string |
| `schema:identifier` | **Required** | string or `schema:PropertyValue` |
| `schema:dateModified` | **Required** | string (ISO 8601) |
| `skos:hasTopConcept` | **Required**, repeatable | array of `CdifCodelistConcept`, at least one |
| `schema:license` | Required if no `schema:conditionsOfAccess` | array of string or object reference |
| `schema:conditionsOfAccess` | Required if no `schema:license` | array of string |
| `@context` | Optional | object |
| `@type` | Optional, repeatable | array of string |
| `schema:url` | Optional | string (URI) |
| `skos:definition` | Optional | string |
| `skos:note` | Optional | string |
| `schema:subjectOf` | Optional | schema:Dataset |

### @context

- **Cardinality:** Optional
- **Content:** object
- **Description:** JSON-LD context declaring the `skos` namespace prefix and any additional prefixes used in concept URIs. When present it must bind `skos` to `http://www.w3.org/2004/02/skos/core#`.

### @id

- **Cardinality:** Required
- **Content:** string (URI)
- **Description:** Globally unique, resolvable URI for the concept scheme.

### @type

- **Cardinality:** Optional, Repeatable
- **Content:** array of string
- **Description:** When present, must contain `skos:ConceptScheme`. The schema does not require the property, but omitting it leaves the scheme untyped, and the SHACL shapes target `skos:ConceptScheme` — an untyped scheme is not checked by any of them. Supply it.

### skos:prefLabel

- **Cardinality:** Required
- **Content:** string
- **Description:** Preferred lexical label for the concept scheme. Each language should appear at most once; two values with duplicate `@language` tags are non-conformant, and consumers may reject the document or arbitrarily pick one. Alternate labels (acronyms, spelling variants, obsolete names) belong on `skos:altLabel`, not here. See [Language tagging](#language-tagging) — the schema currently accepts a bare string only.

### schema:identifier

- **Cardinality:** Required
- **Content:** string, or [schema:PropertyValue](#schemapropertyvalue)
- **Description:** Primary identifier for the codelist. A plain string when the identifier is a resolvable URI; a structured `schema:PropertyValue` when it is a scheme-qualified identifier such as a DOI. Takes precedence over the equivalent `dcterms` property from the base `skos:ConceptScheme`.

### schema:dateModified

- **Cardinality:** Required
- **Content:** string (ISO 8601 date)
- **Description:** Date the codelist was last modified.

### skos:hasTopConcept

- **Cardinality:** Required, Repeatable
- **Content:** array of [CdifCodelistConcept](#cdifcodelistconcept), at least one
- **Description:** The top-level concepts of the scheme — those with no `skos:broader` within it. This is the root of the JSON-LD tree: every other concept is reached by traversing `skos:narrower` from here, so a concept in no `hasTopConcept` chain is unreachable in the document. Items are inline concept objects, not `@id` references.

### schema:license

- **Cardinality:** Required if no `schema:conditionsOfAccess`; Repeatable
- **Content:** array of string or [object reference](#object-reference)
- **Description:** Licence for the codelist. The schema requires **at least one of** `schema:license` and `schema:conditionsOfAccess`: a codelist declaring neither fails validation. Takes precedence over the equivalent `dcterms` property.

### schema:conditionsOfAccess

- **Cardinality:** Required if no `schema:license`; Repeatable
- **Content:** array of string
- **Description:** Text statement of access conditions for the codelist. See `schema:license` above for the either-or requirement.

### schema:url

- **Cardinality:** Optional
- **Content:** string (URI)
- **Description:** Web location of a page describing the codelist. The schema declares a default of `'missing'`.

### skos:definition

- **Cardinality:** Optional
- **Content:** string
- **Description:** Formal explanation of the meaning or purpose of this concept scheme.

### skos:note

- **Cardinality:** Optional
- **Content:** string
- **Description:** General note about the concept scheme.

### schema:subjectOf

- **Cardinality:** Optional
- **Content:** [schema:Dataset](#schemadataset-the-catalog-record)
- **Description:** The catalog record describing this codelist as CDIF-conformant metadata. This is the only place profile conformance is declared. Optional on the scheme, but when present its own `@type`, `schema:additionalType` and `dcterms:conformsTo` are all required — see [schema:Dataset](#schemadataset-the-catalog-record).

## CdifCodelistConcept

[^ Back to TOC](#table-of-contents)

A SKOS Concept constrained for CDIF codelist use: one term, category or code in the scheme. Because JSON-LD is open-world, any other SKOS property may also be included — see [SKOS properties beyond this profile](#skos-properties-beyond-this-profile).

| property | cardinality | content |
|---|---|---|
| `@id` | **Required** | string (URI) |
| `skos:inScheme` | **Required**, repeatable | array of object reference |
| `skos:prefLabel` | **Required** | string |
| `skos:notation` | **Required** | string |
| `skos:definition` | Optional | string |
| `skos:narrower` | Optional, repeatable | array of `CdifCodelistConcept` or object reference |
| `skos:broader` | Required if this concept is a `skos:narrower` value; repeatable | array of object reference |

The profile assigns this class **no `@type` property**. Nothing in the schema requires a concept to be typed `skos:Concept`, and the SHACL shapes target that class, so an untyped concept is checked by none of them. Type your concepts `skos:Concept` regardless.

### @id

- **Cardinality:** Required
- **Content:** string (URI)
- **Description:** Globally unique, resolvable URI for this concept.

### skos:inScheme

- **Cardinality:** Required, Repeatable
- **Content:** array of [object reference](#object-reference)
- **Description:** The concept scheme(s) this concept belongs to. Always an array, and each item is a sealed reference — `{"@id": "scheme-uri"}` and nothing else. A bare string or an inline scheme object is not accepted.

### skos:prefLabel

- **Cardinality:** Required
- **Content:** string
- **Description:** Preferred lexical label for this concept. At most one per language, enforced in SHACL by `sh:uniqueLang`. Alternate labels go on `skos:altLabel`. See [Language tagging](#language-tagging).

### skos:notation

- **Cardinality:** Required
- **Content:** string
- **Description:** The classification code for this concept within the scheme — the machine-facing value that data records carry. A single string, not an array. Codes should be unique within the scheme. This property is what distinguishes a codelist concept from a general vocabulary concept, and it is required on every concept.

### skos:definition

- **Cardinality:** Optional
- **Content:** string
- **Description:** Formal definition of this concept. Optional here, as it is in the Concept Scheme profile. Note that the canonical `skosProperties/skosConcept` building block *does* require `skos:definition` and `skos:inScheme` on a Concept, so a concept validating here will not necessarily validate against that block.

### skos:narrower

- **Cardinality:** Optional, Repeatable
- **Content:** array of [CdifCodelistConcept](#cdifcodelistconcept) or [object reference](#object-reference)
- **Description:** Narrower (child) concepts. Items are either full inline concept objects — which is how the JSON tree is built — or `{"@id": "child-uri"}` references. **An inline child must itself declare `skos:broader`**; the schema enforces this on the inline branch. See [Bidirectional hierarchy](#bidirectional-hierarchy).

### skos:broader

- **Cardinality:** Required if this concept appears as a `skos:narrower` value; Repeatable
- **Content:** array of [object reference](#object-reference)
- **Description:** Broader (parent) concepts, as sealed `{"@id": "parent-concept-uri"}` references. Any concept that is the target of another concept's `skos:narrower` must point back. Top concepts should **not** carry `skos:broader` within the scheme. See [Bidirectional hierarchy](#bidirectional-hierarchy).

## schema:Dataset (the catalog record)

[^ Back to TOC](#table-of-contents)

The value of [`schema:subjectOf`](#schemasubjectof): a catalog record describing the codelist as CDIF-conformant metadata. This is where profile conformance is declared, and it was undocumented in earlier revisions of this guide.

The record is typed `schema:Dataset` and marked as a catalog record through `schema:additionalType`. There is no class named `dcat:CatalogRecord` — `dcat:CatalogRecord` is an `schema:additionalType` value on the `Dataset` node that documents the metadata record itself.

| property | cardinality | content |
|---|---|---|
| `@type` | **Required**, repeatable | array of string containing `schema:Dataset` |
| `schema:additionalType` | **Required**, repeatable | array containing `{"@id": "dcat:CatalogRecord"}` |
| `dcterms:conformsTo` | **Required**, repeatable | array containing `{"@id": "https://w3id.org/cdif/codelist/1.1"}` |
| `@id` | Optional | string |
| `schema:about` | Optional | object reference |

```json
"schema:subjectOf": {
  "@id": "https://example.org/vocab/myCodelist/record",
  "@type": ["schema:Dataset"],
  "schema:additionalType": [{"@id": "dcat:CatalogRecord"}],
  "dcterms:conformsTo": [{"@id": "https://w3id.org/cdif/codelist/1.1"}],
  "schema:about": {"@id": "https://example.org/vocab/myCodelist"}
}
```

### schema:additionalType

- **Cardinality:** Required, Repeatable
- **Content:** array of string or [object reference](#object-reference), at least one
- **Description:** Must contain the object reference `{"@id": "dcat:CatalogRecord"}`, which is what marks this node as the metadata record rather than as the described resource. The bare string `"dcat:CatalogRecord"` satisfies the item schema but **not** the `contains` constraint, which requires the `{"@id": ...}` object form. Tooling that reads `schema:additionalType` as a string literal will not recognize the record.

### dcterms:conformsTo

- **Cardinality:** Required, Repeatable
- **Content:** array of [object reference](#object-reference), at least one
- **Description:** Must contain `{"@id": "https://w3id.org/cdif/codelist/1.1"}`. This is the record's claim to conform to this profile. Each item is a sealed `{@id}` reference; a bare string is not accepted.

### schema:about

- **Cardinality:** Optional
- **Content:** [object reference](#object-reference)
- **Description:** The resource this record describes, normally the codelist's own `@id`.

The record's `@type` is required and must contain `schema:Dataset`. Its `@id` is optional, and is distinct from the codelist's own `@id` — the record and the thing it describes are different resources.

# Value types

[^ Back to TOC](#table-of-contents)

## Object reference

[^ Back to TOC](#table-of-contents)

A reference to another node by its `@id`. Throughout this profile these are **sealed**: `@id` is required and no other property is permitted.

```json
{"@id": "https://w3id.org/isample/vocabulary/sampledfeature/anysampledfeature"}
```

## schema:PropertyValue

[^ Back to TOC](#table-of-contents)

The structured form of [`schema:identifier`](#schemaidentifier), for identifiers that are not simple resolvable URIs.

```json
{
  "@type": ["schema:PropertyValue"],
  "schema:propertyID": "https://registry.identifiers.org/registry/doi",
  "schema:value": "10.5683/SP2/TTJNIU",
  "schema:url": "https://doi.org/10.5683/SP2/TTJNIU"
}
```

# Language tagging

[^ Back to TOC](#table-of-contents)

**The schema currently types every label and note in this profile as a plain `string`.** A language-tagged value object — `{"@value": "Material", "@language": "en"}` — is *not* accepted by `skos:prefLabel`, `skos:definition` or `skos:note` here, even though the schema's own property descriptions say a language-tagged value or an array of them may be supplied.

This is an inconsistency between the schema's constraints and its prose, and a divergence from the [Concept Scheme profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-conceptscheme), whose `LanguageTaggedValue` type is accepted on the equivalent properties. It is recorded here rather than resolved silently: as the schema stands, a multilingual codelist cannot be expressed in this profile, and what validates is a bare string.

# Bidirectional hierarchy

[^ Back to TOC](#table-of-contents)

CDIF codelists require concept hierarchies to be expressed in both directions:

- **`skos:narrower`** is needed because the JSON-LD tree is rooted at `skos:hasTopConcept`. Without `skos:narrower`, child concepts cannot be reached by traversing the JSON document from the root.

- **`skos:broader`** is needed for upward navigation and for display trees in vocabulary browsers and classification tools.

Any concept that appears as a value of `skos:narrower` **must** also declare `skos:broader` pointing back to its parent. Top concepts (those in `skos:hasTopConcept`) should **not** have `skos:broader` within the scheme.

```json
{
  "@id": "sf:anysampledfeature",
  "@type": ["skos:Concept"],
  "skos:prefLabel": "Any sampled feature",
  "skos:notation": "anysampledfeature",
  "skos:definition": "Top concept",
  "skos:inScheme": [{"@id": "sf:sampledfeaturevocabulary"}],
  "skos:narrower": [
    {
      "@id": "sf:earthmaterial",
      "@type": ["skos:Concept"],
      "skos:prefLabel": "Natural Solid Material",
      "skos:notation": "earthmaterial",
      "skos:definition": "A naturally occurring solid material.",
      "skos:inScheme": [{"@id": "sf:sampledfeaturevocabulary"}],
      "skos:broader": [{"@id": "sf:anysampledfeature"}]
    }
  ]
}
```

# Array convention

[^ Back to TOC](#table-of-contents)

Unlike other CDIF profiles, the Codelist profile does **not** require every repeatable property to be serialized as an array. This recognizes standard SKOS practice, which allows either a single value or an array for literal values.

Which properties are arrays is set per property, and the [class tables](#classes) above are authoritative. In particular `skos:inScheme`, `skos:broader`, `schema:license` and `skos:hasTopConcept` **are** arrays in this profile even when they carry a single value, while `skos:prefLabel`, `skos:notation`, `skos:definition` and `skos:note` are single strings.

Consumers should test whether a value is a scalar or an array before iterating.

# SKOS properties beyond this profile

[^ Back to TOC](#table-of-contents)

JSON-LD validation here is open-world: properties the profile does not declare are permitted and are not rejected. The following are in common use on codelists and appeared in earlier revisions of this guide, but are **not** constrained by this profile's schema — no cardinality or content rule applies to them, and neither the JSON Schema nor the SHACL shapes check them:

- `skos:altLabel` — alternative labels (acronyms, abbreviations, spelling variants).
- `skos:topConceptOf` — the inverse of `skos:hasTopConcept`; the scheme(s) for which a concept is a top concept.
- `schema:creator` — author or maintainer of the vocabulary.

Use them where they help. Do not rely on validation to check them, and do not treat their presence as a conformance requirement.

# Validation

[^ Back to TOC](#table-of-contents)

- **JSON Schema** validates structure: the scheme's required properties (`@id`, `skos:prefLabel`, `schema:identifier`, `schema:dateModified`, `skos:hasTopConcept`, and at least one of `schema:license` / `schema:conditionsOfAccess`); each concept's required properties (`@id`, `skos:inScheme`, `skos:prefLabel`, `skos:notation`); the catalog record's required properties when `schema:subjectOf` is present; and bidirectional hierarchy, in that an inline `skos:narrower` concept must declare `skos:broader`.
- **SHACL** validates RDF constraints: `sh:uniqueLang` on `skos:prefLabel`, `sh:class skos:ConceptScheme` on `skos:inScheme`, `sh:class skos:Concept` on `skos:broader`, and the `narrowerImpliesBroaderShape` SPARQL-targeted rule.

The two layers do not check the same things, and neither is a superset of the other. A document that passes JSON Schema can still fail SHACL — most commonly on `sh:uniqueLang`, which JSON Schema cannot express.
