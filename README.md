# lm-vault - the light memory vault format

A memory is a timestamped, typed, attributable markdown claim. This specification
defines how such memories are written, named, routed, and attributed on disk, so
that a person and their agents can share one durable, inspectable store. It
standardizes four things and nothing more: the unit (chapter 1), the notation
grammar (chapter 2), the provenance contract (chapter 3), and the vault layout
(chapter 4). **Out of scope, on purpose:** retrieval and ranking - embeddings,
graphs, cue anchors, policy retrievers - are not specified here; any retrieval
system MAY sit on top of a compliant vault, and doing so does not change the
format. Also out of scope is the block envelope, a separate rendering contract
versioned on its own cadence (see "Relationship to other contracts"). A vault is
a place you keep what you decided to keep; how anything recalls it later is
someone else's layer.

"Light" is the load-bearing word. Entries are atomic plain-text claims that
parse without a model and degrade to searchable prose when notation is omitted.
The intelligence - salience, synthesis, retrieval - is machinery that sits above
the vault, never inside the format.

## Requirement levels (RFC 2119)

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT,
RECOMMENDED, MAY, and OPTIONAL in this specification are to be interpreted as
described in RFC 2119. In this document they are written in uppercase. A
requirement stated as MUST is a conformance condition; a tool that violates one
does not implement the lm-vault format. SHOULD-level guidance may be departed
from with reason; MAY-level items are genuinely optional.

Where this text and the reference implementation disagree, the disagreement is a
defect in one of them; for format version 1.0.0 the reference implementation's
observed behavior is normative and this prose has been written to match it.
Sections that describe deliberate leniency (inputs a strict reader might reject
but a compliant reader accepts) are marked "Leniency".

## Reference implementation

LogMark is the reference implementation of this format. Its open kernel,
[logmark](https://github.com/logmark-dev/logmark), parses, routes, stores,
searches, and lints vaults exactly as described here, and the conformance
fixtures in `fixtures/` are generated from that kernel's parser. Nothing in
this document is specific to LogMark the product; the format is the contract,
and any tool may read or write it.

## What this document contains

| File | Chapter | Standardizes |
|---|---|---|
| `spec/01-entry.md` | The unit | A memory as a timestamped, typed, attributable claim - not a document, chat log, or database row |
| `spec/02-notation.md` | The grammar | Entry types, `+routes`, `#tags` - minimal, parseable without AI, optional by design |
| `spec/03-provenance.md` | The provenance contract | Author, source, session, references, lifecycle - the auditable "prove what it remembers" property |
| `spec/04-layout.md` | The layout | The `vault.md` manifest, folder-as-route semantics, system-file locations |
| `SPEC-VERSION.md` | Versioning | The vault format version (v1.0.0) and its compatibility promise |
| `fixtures/` | Conformance | Four sample vaults with parser-generated `expected.json` ground truth |

## Relationship to other contracts

Two version numbers are contracts in this ecosystem. This document defines the
first; it does not define the second.

1. **Vault format version** (this spec): notation, provenance, layout. See
   `SPEC-VERSION.md`. Current: 1.0.0.
2. **Block envelope schema version** (elsewhere): the typed render envelope a
   producer emits for a viewer to display. It is a rendering contract, not a
   storage contract, and is versioned separately so a renderer and a vault can
   evolve independently. It is deliberately not part of this specification.

## Not a linked-data graph

This format uses frontmatter and trailer metadata as a provenance contract -
who wrote a claim, from what source, when, and where it stands in its lifecycle.
It does not convert prose into RDF triples or require a linked-data context
file. Adjacent efforts that turn markdown vaults into linked-data graphs solve a
different problem; an lm-vault vault is legible to a person reading the raw
markdown first, and to machines second.

## Positioning

Agent-memory systems standardize how agents recall; lm-vault standardizes what
humans keep.
