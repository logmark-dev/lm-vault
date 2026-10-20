# Vault format version

**Version: 1.0.0**

This is the vault format version - the first of the two compatibility contracts
in the LogMark ecosystem (the second, the block envelope schema, is versioned
separately and is out of scope for this specification). It governs the notation
grammar, the provenance contract, and the vault layout defined in `spec/`.

## Semantics

The version is `MAJOR.MINOR.PATCH`.

- **MAJOR** increments on a breaking change: an input that a 1.x reader parsed
  one way that a 2.x reader parses differently, or a previously-valid vault that
  a 2.x reader would reject. A conformant reader MUST NOT silently reinterpret a
  vault that declares a MAJOR version it does not implement.
- **MINOR** increments when the format gains an optional capability that older
  readers ignore safely (a new entry type, a new provenance field). A 1.0 reader
  encountering a 1.1 vault MUST degrade gracefully: unknown notation is preserved
  as plain text (chapter 2), and unknown metadata lines are retained (chapter 3),
  never dropped.
- **PATCH** increments on clarifications that do not change how any valid vault
  parses.

## Declaring a version

A vault SHOULD declare the format version it targets in its `vault.md` manifest
frontmatter:

```yaml
---
schema: lm-vault/1.0.0
---
```

The version is how an amendment to this specification reaches a vault already on
disk: without it a reader cannot tell which revision a vault was written against.

- Tools that create a vault SHOULD write the declaration, naming the version they
  implement.
- A writer that rewrites the manifest MUST preserve an existing declaration
  unless it is deliberately changing it. A tool that exposes the manifest as
  parsed sections, without the frontmatter, MUST carry the existing frontmatter
  over when it writes back.
- A reader MUST NOT refuse a vault because the declaration is absent; an
  undeclared vault is treated as the current version.
- A reader encountering a later MINOR version opens the vault and degrades as
  described above.

The reference implementation writes the declaration at vault creation, keeps it
across constitution updates, and surfaces the declared value to agents. When a
vault declares a later MAJOR it still opens, but it warns at startup and in
`get_constitution` rather than reading the vault silently.

## Compatibility promise

Within MAJOR version 1, the guarantees are:

- Every entry that a 1.0 reader parses, a later 1.x reader parses identically.
- The seven typed prefixes and the untyped log (chapter 2) never change meaning.
- The provenance fields (chapter 3) never change meaning; new fields may be
  added and older readers retain them verbatim.
- The layout anchors (chapter 4) - the `logmark/` content root, the `_system/`
  folder, folder-as-route, the `vault.md` manifest - never move within MAJOR 1.

## History

| Version | Date | Change |
|---|---|---|
| 1.0.0 | 2026-07-23 | Initial published format: entry unit, notation grammar, provenance contract, vault layout. |
