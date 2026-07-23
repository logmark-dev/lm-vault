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

A vault MAY declare the format version it targets in its `vault.md` manifest
frontmatter:

```yaml
---
schema: lm-vault/1.0.0
---
```

This declaration is OPTIONAL. The reference implementation does not currently
read or enforce it: an undeclared vault is treated as current, and a declared
version is advisory. This is deliberate forward-lenient behavior - a vault
written against a later MINOR version still opens. Tools that need strict version
gating SHOULD read the declaration themselves; they MUST NOT refuse a vault
merely because the declaration is absent.

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
