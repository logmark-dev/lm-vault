# Chapter 2 - The grammar

Notation is explicit meaning at typing speed. It classifies an entry without
inference, provides structure for statistics, and degrades to plain searchable
text when omitted. This chapter defines the notation a conformant reader MUST
recognize: entry types, `+routes`, and `#tags`.

The design constraints are fixed:

- **Minimal.** A small closed vocabulary, no closing delimiters, no schema.
- **Parseable without AI.** Every construct is a regular expression away.
- **Optional by design.** An entry with no notation is a valid log entry
  (section 2.1). Notation adds structure; its absence is not an error.

## 2.1 Entry types

An entry's type is determined by a **leading prefix** on the first non-empty line
of its body, or defaults to an untyped log. There are seven typed prefixes, each
with a long form and a short form, plus the implicit log.

| Long | Short | Type | Meaning |
|---|---|---|---|
| `task:` | `t:` | Task | An actionable item |
| `idea:` | `i:` | Idea | A thought worth preserving, not yet actionable |
| `block:` | `b:` | Block | Something explicitly stuck |
| `decision:` | `d:` | Decision | A choice made, stakes recorded |
| `question:` | `q:` | Question | An open inquiry worth tracking |
| `gap:` | `g:` | Gap | Known knowledge debt |
| `analogy:` | `a:` | Analogy | A structural cross-domain mapping |
| (none) | | Log | The default: an undated-type observation |

Rules a conformant reader MUST follow:

- The prefix is a form (long or short) followed immediately by a colon and then
  **at least one whitespace character**. `task: x` is a task; `task:x` and
  `tasky:` are not - they are plain log text. This whitespace guard is what keeps
  `note:` or `https://...` from being mistaken for a type.
- Matching is case-insensitive.
- The prefix MUST be at the body start. One optional leading `+route` token is
  permitted before it (`+refinery t: ship the parser`), because routing precedes
  typing in the notation. A prefix appearing later in the line, or inline (`... t:
  later`), does NOT set the type.
- When both a short and long form could match at the same position, the longest
  match wins (`idea:` over `i:`).
- An unrecognized prefix (`xyz: ...`, `taskfoo: ...`) is not a type. The entry
  degrades to a log entry with the text preserved verbatim.

**Type may also come from location.** An entry stored under a type folder
(`_tasks/`, `_ideas/`, ...) inherits that type even with no prefix (chapter 4,
section 4.3). When both a folder and a prefix are present they normally agree; if
they conflict, the reference implementation lets an explicit prefix on the line
stand for display while the folder determines the file it lives in. Producers
SHOULD keep folder and prefix consistent.

**Advanced types.** `question:`, `gap:`, and `analogy:` are full members of the
grammar - parsed everywhere - but are "advanced": tools MAY omit them from
beginner onboarding. They MUST still be parsed when present.

**Reference-implementation note (not part of the portable grammar).** The
reference implementation carries an eighth internal type, timetrack, recognized
structurally from a duration/timeline body rather than from a portable `tt:`
prefix. It is an implementation feature, not a required part of this grammar; a
conformant reader is not obligated to produce it.

## 2.2 Routes: `+project` and `+project/scope`

A `+route` token routes an entry to a project or domain. Syntax:

```
+project
+project/scope
+project/scope/subscope
```

Recognition rules a conformant reader MUST follow:

- A `+` token is recognized only at the start of a line or after whitespace (the
  boundary rule). This prevents `1+2`, `c++`, and `beans+meat` from parsing as
  routes.
- The base segment MUST start with an ASCII letter, followed by zero or more of
  `[A-Za-z0-9_-]`. **A single-letter base is valid** (`+r` is a route). There is
  no minimum length beyond the one required letter.
- Each `/scope` segment MUST likewise start with a letter, then `[A-Za-z0-9_-]*`.
- A token whose base starts with a digit (`+5`, `+18ti`) is NOT a route.

> **Disagreement with earlier prose (parser is normative).** An earlier
> LogMark notation document stated a `+project` needs "at least two characters
> total." The reference parser accepts a single-letter base (`+r`). For format
> version 1.0.0 the single-letter rule above is normative. A separate, stricter
> threshold (base of two or more characters) governs only the interactive
> "create a new route?" prompt in the reference implementation, so that typed
> text like `chmod +x` never offers to make a route; that threshold is a UX
> affordance, not a parsing rule.

Routes are extracted case-insensitively and de-duplicated. Extraction reports the
token text without the leading `+`.

## 2.3 Tags: `#tag`

A `#tag` marks a cross-cutting topic. Syntax and rules:

- A tag is `#` followed by a run of `[A-Za-z0-9_-]`, optionally continued with
  `/`- or `:`-separated segments (`#ml`, `#surface/mcp`, `#hardware:4090`).
- A tag MUST contain at least one letter (a-z or A-Z). `#2` and `#1234` are NOT
  tags (they are literal references like "issue #1234"); `#2fa` and `#v2fix`
  ARE tags, because they carry a letter.
- Tags are extracted case-insensitively and de-duplicated.

### 2.4 Faceted tags and leaf collapse

A tag containing a separator is a **faceted tag**: the segment before the first
separator is the facet key, the remainder is the value. `#surface/mcp` has key
`surface`, value `mcp`.

- A `:` separator is normalized to `/` for on-disk portability with nested-tag
  conventions (`#hardware:4090` is equivalent to `#hardware/4090`).
- **Leaf collapse (contract-worthy).** When a reader produces the flat tag index
  for an entry, each tag collapses to its **leaf** - the final segment after the
  last separator. `#surface/mcp` indexes as `mcp`; a bare `#mcp` indexes as the
  same `mcp`. The full slash-path is preserved on disk and is available to
  facet-aware consumers; only the flat index collapses. A conformant reader that
  exposes a flat tag list MUST apply leaf collapse so that faceted and flat forms
  of the same leaf reconcile.
- A tag with a trailing or doubled separator degrades gracefully: the malformed
  tail is not matched, and the valid leading portion is taken as the tag.

## 2.5 Notation inside wiki-links is inert

Obsidian-style wiki-link spans - `[[target]]`, `[[target|alias]]`,
`[[target#heading]]`, `[[target#^blockid]]` - are link syntax. A conformant
reader MUST remove wiki-link spans before extracting `#tags` and `+routes`, so
that a heading fragment (`#heading`) or block anchor (`#^b5568360`) inside a link
never pollutes the tag index. The span is treated as a boundary, not silently
concatenated with its neighbours.

## 2.6 Graceful degradation is mandatory

The whole grammar is optional and lenient by construction:

- An entry with no prefix, no routes, and no tags is a valid log entry and MUST
  be stored and searchable as plain text.
- Malformed notation (a digit-leading `+5`, a numeric `#2`, an unknown `xyz:`
  prefix) is left as literal text, never an error.
- A reader encountering notation it does not understand (for example a construct
  from a later MINOR format version) MUST preserve the text verbatim rather than
  dropping the entry.

This is the "parseable without AI, optional by design" promise made concrete: the
plain file is always readable, and structure is a bonus a reader extracts, never
a precondition it demands.
