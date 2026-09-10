# CDIF Codelist Profile

A CDIF profile for controlled vocabulary codelists implemented as a [SKOS ConceptScheme](https://www.w3.org/TR/skos-reference/) in JSON-LD. Defines how classification schemes, thesauri, and enumerated value domains are represented with machine-enforceable constraints.

## Specification

- **[Implementation Guide](CDIFCodelistImplementationGuide.md)** — Complete classes and properties documentation
- **[CDIFCodelistProfileStructuredSchema.json](CDIFCodelistProfileStructuredSchema.json)** — JSON Schema for validation (generated from metadataBuildingBlocks)
- **[rules.shacl](rules.shacl)** — SHACL validation shapes (synced from metadataBuildingBlocks)

## ConceptScheme requirements

- Must have a globally unique, resolvable `@id` URI
- Must have at least one `skos:prefLabel`
- Must declare top concepts via `skos:hasTopConcept`
- Must have `schema:identifier`, `schema:dateModified`, and either `schema:license` or `schema:conditionsOfAccess` (CDIF core metadata properties)

## Concept requirements

- Must have a globally unique, resolvable `@id` URI
- Must have `skos:inScheme` linking to the containing ConceptScheme
- Must have at least one `skos:prefLabel` (at most one per language)
- Must have at least one `skos:definition`
- `skos:notation` is optional but must be unique within the scheme if used
- Hierarchical concepts must declare both `skos:narrower` and `skos:broader` (see below)

## Bidirectional hierarchy

CDIF codelists require concept hierarchies to be expressed in **both directions**:

- **`skos:narrower`** — needed for JSON-LD tree traversal from `skos:hasTopConcept` root
- **`skos:broader`** — needed for upward navigation and display trees in applications

Any concept in `skos:narrower` **must** also have `skos:broader` pointing back. Top concepts should not have `skos:broader` within the scheme.

## Array convention

Unlike other CDIF profiles, the Codelist profile does **not** require repeatable properties to always be serialized as arrays. This recognizes standard SKOS practice that allows either a single string or an array for literal values. Consumers should test whether a value is a string or an array before iterating.

## Examples

| File | Description |
|------|-------------|
| `examples/exampleCdifCodelist.json` | iSamples Sampled Feature Type vocabulary (full, with hierarchy and history notes) |
| `examples/exampleCDIFCodelistMinimal.json` | iSamples Materials vocabulary (minimal, with hierarchy) |

## JSON-LD Framing and Validation

**`FrameAndValidate.py`** frames a SKOS ConceptScheme JSON-LD document against the Codelist profile schema and optionally validates it:

```bash
# Frame and validate
python FrameAndValidate.py examples/exampleCDIFCodelist.json --validate

# Frame and save output
python FrameAndValidate.py examples/exampleCDIFCodelistMinimal.json -o framed.json
```

The script uses **`CDIFCodelist-frame.jsonld`** to frame JSON-LD documents. The frame targets `skos:ConceptScheme` (not `schema:Dataset`) and handles recursive concept hierarchies via `skos:narrower`/`skos:broader`. Context prefixes from the input document are automatically merged into the frame.

**Requirements:** `pyld`, `jsonschema` (`pip install pyld jsonschema`)

## SHACL Validation

**`rules.shacl`** contains SHACL shapes for validating CDIF Codelist profile instances. Source shapes come from [`metadataBuildingBlocks/_sources/profiles/cdifProfiles/CDIFCodelistProfile/rules.shacl`](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks/blob/main/_sources/profiles/cdifProfiles/CDIFCodelistProfile/rules.shacl) and should be updated whenever the source changes.

## Related repositories

- **[metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks)** — Source building block schemas (skosConceptScheme, skosConcept, CDIFCodelistProfile)
- **[cdif-core](https://github.com/Cross-Domain-Interoperability-Framework/core)** — CDIF Core profile
- **[validation](https://github.com/Cross-Domain-Interoperability-Framework/validation)** — Validation tools

## Reference

This profile aligns with the approach described in ['Modelling of Eurostat's Statistical Classifications in ShowVoc'](https://cros.ec.europa.eu/book-page/modeling-eurostats-statistical-classifications-showvoc).

## Changelog — v1.1.0

Released 2026-09-10 as `v1.1.0`. Content synced from the CDIF
**metadataBuildingBlocks** source; see the
[release](../../releases/tag/v1.1.0) for the tagged snapshot and
`git log v1.1.0` for the per-commit history:

- **Populated from metadataBuildingBlocks** — `*StructuredSchema.json`, merged SHACL,
  JSON-LD frame, examples, and the normative `FrameAndValidate.py` generated from the
  building-block source; `Examples/` renamed to `examples/`.
- **CDIF v1.1** — profile conformance URIs migrated `/1.0` → `/1.1`.
- **License** standardized on CC-BY-4.0.
- **`@id`-reference tightening** — bare `{@id}` reference slots sealed
  (`additionalProperties: false` + `required: ['@id']`); a canonical `objectReference`
  building block introduced as the strict node reference.
- **`prov:used` wrapper reconciliation** — the base `generatedBy.prov:used` accepts
  role-keyed wrappers (`schema:instrument` / `bios:computationalTool` / `prov:reagent`)
  alongside string / `{@id}` / inline `prov:Entity`; profiles pin a wrapper's shape via
  a constraint-only `if/then` (never a narrowed `anyOf`).
- **`skos:notation` → single string** at concept level (consistent with the codelist
  single-notation design).
- **`FrameAndValidate.py`** (normative, drift-checked against
  `Cross-Domain-Interoperability-Framework/validation`) — two-frame root-`@type`
  selection, context-aware `schema:about`, `--conformance` detection, `cdif:`-`@id`
  re-expansion, and (2026-08) reference-collapse on all document types + blank-node
  dedupe + agent `schema:identifier` unwrap, so `@embed:@always`-framed documents
  validate against the tightened schemas.
- **Examples** conformed to the tightened schemas throughout (PrimaryKey →
  `cdi:ComponentPosition`, reference slots → `{@id}`, CVE `hasIntendedDataType` →
  string, `skos:notation` → string, `schema:additionalType` URI → `{@id}`).


## Branches

`main` is the **current release** — GitHub Pages serves it, so the published
URLs always show the newest release. It is protected: changes reach it only by
pull request, which means the merge *is* the release.

New work goes on the **`updates`** branch and is merged to `main` when a release
is cut, then tagged `v1.1.n`. The former `reviewRevision202606` branch is retained
as **`archive202609`**.


## License

See [LICENSE](LICENSE).
