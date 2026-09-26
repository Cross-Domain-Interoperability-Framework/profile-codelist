---
title: CDIF Codelist Profile Classes and Properties
date: 2026-04-05
---

# Introduction

This document describes all classes and properties for the CDIF Codelist profile, which defines how classification schemes used to populate data values are represented as SKOS ConceptSchemes in JSON-LD. The profile composes the base SKOS ConceptScheme and Concept building blocks with CDIF-specific constraints: CDIF core metadata properties for the schema, notation (code) values and labels for all concepts. If the terme in the codelist are hierarchical, both broader and narrower relations must be asserted.

The implementation uses the [SKOS (Simple Knowledge Organization System)](https://www.w3.org/TR/skos-reference/) vocabulary with JSON-LD serialization. This profile aligns with the approach described in ['Modelling of Eurostat's Statistical Classifications in ShowVoc'](https://cros.ec.europa.eu/book-page/modeling-eurostats-statistical-classifications-showvoc).

# Table of contents

- [Namespaces](#namespaces)
- [Model](#model)
  - [CdifCodelistConcept](#cdifcodelistconcept)
  - [ConceptScheme](#conceptscheme)
  - [Data Types](#data-types)
  - [LanguageTaggedValue](#languagetaggedvalue)
  - [Object Reference](#object-reference)
  - [Optional Properties](#optional-properties)
  - [Optional Properties](#optional-properties)
  - [Required Properties](#required-properties)
- [Bidirectional Hierarchy](#bidirectional-hierarchy)
- [Array Convention](#array-convention)
- [Validation](#validation)

# Namespaces

[^ Back to TOC](#table-of-contents)

```json
"@context": {
  "skos": "http://www.w3.org/2004/02/skos/core#",
  "schema": "http://schema.org/",
  "dcterms": "http://purl.org/dc/terms/"
}
```

# Model

[^ Back to TOC](#table-of-contents)

## CdifCodelistConcept

[^ Back to TOC](#table-of-contents)

- SKOS Concept with CDIF codelist constraints. Represents a single term or category within a concept scheme.

## ConceptScheme

[^ Back to TOC](#table-of-contents)

- The root object representing the codelist or classification scheme.

### @id

- **Cardinality:** Required
- **Content:** string.uri
- **Description:** Globally unique, resolvable URI for the concept scheme.

### @type

- **Cardinality:** Required
- **Content:** array
- **Description:** Must include `skos:ConceptScheme`.

### skos:prefLabel

- **Cardinality:** Required
- **Content:** string, [LanguageTaggedValue](#languagetaggedvalue), or array of [LanguageTaggedValue](#languagetaggedvalue)
- **Description:** Preferred human-readable label for the scheme. At most one per language.

### skos:hasTopConcept

- **Cardinality:** Required, Repeatable
- **Content:** array of [CdifCodelistConcept](#cdifcodelistconcept) or [object reference](#object-reference)
- **Description:** Top-level concepts that have no `skos:broader` within this scheme. The JSON-LD hierarchy is rooted here — all child concepts are reached by traversing `skos:narrower` from these top concepts.

### schema:identifier

- **Cardinality:** Required
- **Content:** string or [PropertyValue](#propertyvalue-for-schemaidentifier)
- **Description:** Primary identifier for the codelist. CDIF core metadata property; takes precedence over `dcterms:identifier`.

### schema:dateModified

- **Cardinality:** Required
- **Content:** string, ISO 8601
- **Description:** Date when the codelist was last modified. Takes precedence over `dcterms:modified`.

- **CHOICE — at least one of:**

### schema:license

- **Cardinality:** Required if no conditionsOfAccess
- **Content:** array of string or [object reference](#object-reference)
- **Description:** License for the codelist. Takes precedence over `dcterms:license`.

### schema:conditionsOfAccess

- **Cardinality:** Required if no license
- **Content:** array of string
- **Description:** Text statement of access conditions.

## Data Types

[^ Back to TOC](#table-of-contents)

## LanguageTaggedValue

[^ Back to TOC](#table-of-contents)

- An RDF literal with a language tag, serialized as a JSON-LD value object.

### @value

- **Cardinality:** Required
- **Content:** string
- **Description:** The text content.

### @language

- **Cardinality:** Required
- **Content:** string
- **Description:** BCP 47 language tag (e.g., `en`, `fr`, `de`, `sv`).
```json
{"@value": "Sampled Feature Type vocabulary", "@language": "en"}
```

## Object Reference

[^ Back to TOC](#table-of-contents)

- A reference to another node by its `@id`, used for linking to concepts or schemes defined elsewhere in the graph or externally.
```json
{"@id": "https://w3id.org/isample/vocabulary/sampledfeature/anysampledfeature"}
```

### PropertyValue (for schema:identifier)

- When the identifier is not a simple resolvable URI, use `schema:PropertyValue`:

```json
{
  "@type": ["schema:PropertyValue"],
  "schema:propertyID": "https://registry.identifiers.org/registry/doi",
  "schema:value": "10.5683/SP2/TTJNIU",
  "schema:url": "https://doi.org/10.5683/SP2/TTJNIU"
}
```

## Optional Properties

[^ Back to TOC](#table-of-contents)

### schema:url

- **Cardinality:** Optional
- **Content:** string.uri
- **Description:** Web location of a page describing the codelist. Default: `'missing'`.

### schema:creator

- **Cardinality:** Optional
- **Content:** Person, Organization, or @list
- **Description:** Author or maintainer of the vocabulary.

### skos:definition

- **Cardinality:** Optional
- **Content:** string, [LanguageTaggedValue](#languagetaggedvalue), or array
- **Description:** Formal explanation of the meaning or purpose of the scheme.

### skos:altLabel

- **Cardinality:** Optional
- **Content:** string, [LanguageTaggedValue](#languagetaggedvalue), or array
- **Description:** Alternative labels (acronyms, abbreviations, spelling variants).

### skos:note

- **Cardinality:** Optional
- **Content:** string, [LanguageTaggedValue](#languagetaggedvalue), or array
- **Description:** General note about the scheme.

## Optional Properties

[^ Back to TOC](#table-of-contents)

### skos:inScheme

- **Cardinality:** Required
- **Content:** [object reference](#object-reference) or array of object references
- **Description:** The concept scheme(s) this concept belongs to. Each must be `{"@id": "scheme-uri"}`.

### skos:definition

- **Cardinality:** Required
- **Content:** string, [LanguageTaggedValue](#languagetaggedvalue), or array
- **Description:** Formal definition of this concept.

### skos:broader

- **Cardinality:** Required if concept appears in skos:narrower
- **Content:** array of object references
- **Description:** Broader (parent) concepts. Any concept that is the target of `skos:narrower` on another concept must declare `skos:broader` pointing back. See [Bidirectional hierarchy](#bidirectional-hierarchy) below. Each item is `{"@id": "parent-concept-uri"}`.

### skos:narrower

- **Cardinality:** Optional, Repeatable
- **Content:** array of [CdifCodelistConcept](#cdifcodelistconcept) or [object reference](#object-reference)
- **Description:** Narrower (child) concepts. If present, each inline child concept must have `skos:broader` pointing back to this concept. Items can be full inline concept objects (for building the JSON tree) or `{"@id": "child-uri"}` references.

### skos:altLabel

- **Cardinality:** Optional
- **Content:** string, [LanguageTaggedValue](#languagetaggedvalue), or array
- **Description:** Alternative labels.

### skos:note

- **Cardinality:** Optional
- **Content:** string, [LanguageTaggedValue](#languagetaggedvalue), or array
- **Description:** General note.

### skos:topConceptOf

- **Cardinality:** Optional
- **Content:** [object reference](#object-reference) or array
- **Description:** Scheme(s) for which this is a top concept.

## Required Properties

[^ Back to TOC](#table-of-contents)

### @id

- **Cardinality:** Required
- **Content:** string.uri
- **Description:** Globally unique, resolvable URI for this concept.

### @type

- **Cardinality:** Required
- **Content:** array
- **Description:** Must include `skos:Concept`.

### skos:prefLabel

- **Cardinality:** Required
- **Content:** string, [LanguageTaggedValue](#languagetaggedvalue), or array of [LanguageTaggedValue](#languagetaggedvalue)
- **Description:** Preferred label. At most one per language (enforced by SHACL `sh:uniqueLang`).

### skos:notation

- **Cardinality:** Optional, Repeatable
- **Content:** array of string
- **Description:** Classification codes. Should be unique within the scheme.

# Bidirectional Hierarchy

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
  "skos:definition": "Top concept",
  "skos:inScheme": {"@id": "sf:sampledfeaturevocabulary"},
  "skos:narrower": [
    {
      "@id": "sf:earthmaterial",
      "@type": ["skos:Concept"],
      "skos:prefLabel": "Natural Solid Material",
      "skos:definition": "A naturally occurring solid material.",
      "skos:inScheme": {"@id": "sf:sampledfeaturevocabulary"},
      "skos:broader": [{"@id": "sf:anysampledfeature"}]
    }
  ]
}
```

# Array Convention

[^ Back to TOC](#table-of-contents)

Unlike other CDIF profiles, the Codelist profile does **not** require repeatable properties to always be serialized as arrays. This recognizes standard SKOS practice that allows either a single string or an array for literal values. For example, both of these are valid:

```json
"skos:prefLabel": "Material"
```

```json
"skos:prefLabel": [
  {"@value": "Material", "@language": "en"},
  {"@value": "Matériau", "@language": "fr"}
]
```

Consumers of CDIF codelist documents should test whether a value is a string or an array before iterating.

# Validation

[^ Back to TOC](#table-of-contents)

- **JSON Schema** validates structure: codelist required properties (`@id`, `skos:prefLabel`, `skos:hasTopConcept`, `schema:identifier`, `schema:dateModified`, license/access), concept requirements (`@id`, `skos:notation`, `skos:prefLabel`), and bidirectional hierarchy (inline narrower concepts must have `skos:broader`)
- **SHACL** validates RDF constraints: `sh:uniqueLang` on `skos:prefLabel`, `sh:class skos:ConceptScheme` on `skos:inScheme`, `sh:class skos:Concept` on `skos:broader`, and the `narrowerImpliesBroaderShape` SPARQL-targeted rule
