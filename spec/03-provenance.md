# Chapter 3 - The provenance contract

Provenance is the auditable "prove what it remembers" property. For every entry a
vault can answer: who wrote this, from what source, in what work session, related
to what, and where it stands in its lifecycle. This is what makes a shared
human-and-agent memory trustworthy - you can see which claims an agent added and
trace each back to its origin.

## 3.0 A note on the word "frontmatter"

Design discussion sometimes calls provenance "the frontmatter contract." In the
reference implementation, entry-level provenance is NOT carried in YAML
frontmatter. It is carried in **trailer metadata lines** - `key: value` lines
that follow the entry body, inside the entry's `## header` block. This is
deliberate: the lines render as plain text in a markdown viewer (no broken YAML
blocks between every dated entry), and they are extracted by a line-oriented
reader. YAML frontmatter is used at the file level for the `vault.md` manifest
(chapter 4), not per entry. Where this document says "provenance," read
"trailer metadata lines," and take that as normative for format version 1.0.0.

## 3.1 The provenance fields

The following trailer lines are recognized. Each appears on its own line within
the entry block, conventionally after the body.

| Line | Meaning | Example |
|---|---|---|
| `source:` | Author attribution. Absent = human. Present = the agent that wrote it. | `source: agent:claude-code` |
| `session:` | Work-session grouping id. | `session: 2026-07-20-a1b2c3` |
| `refs:` | Cross-entry references, comma-separated `type:id`. | `refs: decision:9f8e7d6c, task:b123bbe1` |
| `_id:` | Stable entry id for non-task entries (section 3.4). | `_id: c4d5e6f7` |
| `_task:` | Task metadata: deadline, postpone count, id (section 3.5). | `_task: 2026-07-25, 17:00 \| postponed: 1 \| b123bbe1` |
| `resolved:` | Resolution note for a completed task or a cleared block. | `resolved: raised the bucket ceiling to 200/s` |

A conformant reader MUST recognize these lines and MUST NOT surface them as body
text. A reader MAY encounter other `key: value` lines after the first body line;
those are retained as **extended metadata** (section 3.6), never dropped.

## 3.2 Author: the human-or-agent contract

The single most important provenance rule:

- **An entry with no `source:` line was written by a human.** Absence is the
  signal; there is no `source: human` sentinel.
- **An entry with a `source:` line was written by that source.** The
  conventional value is `agent:<name>` (`agent:claude-code`, `agent:codex`), but
  the field is free text after the colon; a reader MUST store it verbatim and
  MUST NOT require the `agent:` prefix.

This one bit - present or absent - is what lets a reader show at a glance which
memories an agent contributed, and it is the anchor of the trust story: a person
can audit every agent-written claim in their own vault.

**Leniency:** a `source:` line with an empty value (`source:` with nothing after
it) is retained as an empty attribution, not treated as human and not an error. A
conformant reader SHOULD preserve the empty value rather than discard the line.

## 3.3 Session and references

- `session:` groups entries written in one work session. The conventional shape
  is `YYYY-MM-DD-<hash>`, but the field is opaque to the format; a reader stores
  it as given.
- `refs:` is a comma-separated list of `type:id` tokens linking this entry to
  others (`decision:9f8e7d6c`). A reader MUST parse each comma-separated token as
  a `type:id` pair, lower-casing the type. **Leniency:** a token that is not a
  well-formed `type:id` pair (an empty id like `task:`, or a bare word) is
  skipped; the well-formed tokens on the same line are still collected. A reader
  MUST NOT let one malformed reference discard the others.

## 3.4 Entry identity

Timestamps are minute-precise and therefore not unique. For stable identity:

- Non-task entries MAY carry an `_id:` line with an 8-character hex id
  (`_id: c4d5e6f7`), optionally followed by an Obsidian block anchor
  (`^c4d5e6f7`) for deep linking.
- Task entries carry their id inside the `_task:` line (section 3.5).
- An entry with no id is still valid; a reader that needs identity falls back to
  file-path-plus-line. Ids are RECOMMENDED for any entry an agent will later
  reference.

## 3.5 Lifecycle status

Lifecycle - where a claim stands over time - is expressed by a combination of
signals, not a single status field. A conformant reader MUST recognize each:

- **Completion by strikethrough.** A body or task-metadata line wrapped in
  `~~ ... ~~` marks the entry completed. This is how a done task reads as done in
  plain markdown.
- **Resolution note.** A `resolved:` trailer records how a task was completed or a
  block was cleared. For a block - which has no task metadata - this is the only
  place the resolution surfaces, so a reader MUST attach `resolved:` to any entry
  type that carries it, not only tasks.
- **Task metadata** (`_task:`) carries a deadline (an ISO date, optionally with a
  time, or the literal `none`), a `postponed:` count, and the task id, in the form
  `_task: <deadline> | postponed: <n> | <id>`. A reader MUST tolerate the `none`
  deadline and MUST parse the postpone count as an integer.
- **Status tags.** The conventional lifecycle tags `#frozen`, `#active`,
  `#revisit`, `#obsolete` are ordinary `#tags` (chapter 2) used by convention to
  mark decision and entry state. They are not a distinct field; a reader surfaces
  them through the normal tag index. Their meaning is a convention this spec
  documents but does not enforce.

Deadline extraction from prose (turning "by friday" into a date) is a capture-time
convenience of the reference implementation and is NOT part of the storage
contract: on disk, a task's deadline lives in its `_task:` line or nowhere. A
reader parsing a stored vault MUST NOT be required to infer dates from body prose.

## 3.6 Extended metadata is retained, not dropped

Any `key: value` line appearing after the first body line, whose key is lowercase
and not one of the recognized fields above, is retained as **extended metadata**
(for example `origin: import`, `rating: 4`). A conformant reader MUST preserve
these key/value pairs rather than discarding them. This is the forward-compatibility
hinge: a provenance field added in a later MINOR format version lands here in an
older reader and survives a read/write round-trip instead of being lost.

## 3.7 Why provenance is a contract, not a convenience

Retrieval systems can be swapped; a vault outlives them. What must not be lost is
the chain of custody on each claim: a memory whose author, source, and lifecycle
are recorded can be trusted, audited, and corrected. A memory without that chain
is just text. The provenance contract is the difference between a store you can
hand to an agent and a store you cannot.
