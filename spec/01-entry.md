# Chapter 1 - The unit

A vault stores memories. A memory is a **timestamped, typed, attributable markdown claim**. It is not a document, not a chat transcript, not a database row. This chapter defines the unit; chapters 2 through 4 define its notation, its provenance, and where it lives.

## 1.1 What a memory is

A memory is one dated statement a person or an agent chose to keep. Concretely, an entry is:

- **A markdown fragment**, not a file. Many entries share one file (section 1.3). The unit of meaning is the entry, not the document that contains it.
- **Timestamped** to the minute. Every entry carries a `## YYYY-MM-DD HH:mm` header that is both its delimiter and its identity anchor (section 1.2).
- **Typed** by a leading prefix, or untyped by default (chapter 2). The type is a small, closed vocabulary that says what kind of claim this is - a task, an idea, a decision - without prescribing a schema for its body.
- **Attributable**. Absence of a source means a human wrote it; a `source:` trailer names the agent that did (chapter 3).
- **A claim**, meaning it stands on its own. An entry is legible without its neighbours. It is append-first: you add the next dated claim rather than editing yesterday's.

## 1.2 The entry header is the unit boundary

An entry begins at a level-2 markdown header of the form:

```
## YYYY-MM-DD HH:mm
```

optionally followed by a context suffix:

```
## YYYY-MM-DD HH:mm | context
```

A conformant reader MUST treat this header as the start of a new entry and MUST end the previous entry at the line before it. The timestamp is minute-precise; seconds are not represented. Everything from the header up to the next header (or end of file) is that entry's body and trailing metadata.

- The date component is `YYYY-MM-DD` and the time component is `HH:mm`, each a fixed-width run of digits. Leading whitespace before `##` is permitted.
- The optional `| context` records where the entry was captured (an application name, a window title, a terminal). It is free text.
- **Leniency:** a header whose digits form an out-of-range date or time (for example `## 2026-13-45 25:99`) is not a valid timestamp. The reference reader does not error; it drops the entry silently and continues with the next header. A conformant reader SHOULD either parse a header's timestamp or drop that one entry, and MUST NOT abort the file.

Text that appears in an entry file **before the first header** is not part of any entry. A reader MUST ignore it for the purpose of entry extraction. A file with no header at all yields zero entries (it is still valid markdown; it simply contains no memories in the formal sense).

## 1.3 Many entries per file

Entries are grouped into files by type and route and year (chapter 4), so a single file holds a run of entries:

```markdown

---

## 2026-07-20 08:00

Reviewed the overnight digest. Nothing on fire.

## 2026-07-20 18:30 | terminal

Wrapped for the day. #wrap
```

Two headers, two entries, one file. The `---` thematic breaks between entries are cosmetic separators; a reader MUST NOT treat them as entry boundaries (only the `## ` header is a boundary) and MUST tolerate their presence or absence.

Because timestamps are minute-precise, two entries in one file MAY share the exact same header. They remain distinct entries; a reader that needs stable per-entry identity beyond the timestamp uses the entry id (chapter 3, section 3.4).

## 1.4 The body

The body is the markdown between the header's metadata lines and the next entry. It is ordinary markdown: prose, lists, links, code. Two things about the body are part of the contract:

- The **leading type prefix** (if any) is part of the body text, not stripped from it. `task: ship the parser` stores the literal string `task: ship the parser`; the type is derived from it (chapter 2) but the text is preserved so the raw file reads naturally.
- **Notation inside the body** - `+routes` and `#tags` - is extracted as structure (chapter 2) while remaining literally present in the text. Nothing is rewritten away; the plain file is always the source of truth.

Wiki-links (`[[target]]`, `[[target|alias]]`, `[[target#heading]]`) MAY appear in the body. Their internal `#` and `+` characters are link syntax and MUST NOT be extracted as tags or routes (chapter 2, section 2.5).

## 1.5 Why not a document or a row

- **Not a document.** A document has one author, one topic, and is edited in place. A memory is one dated claim among many; its value compounds by accumulation, not revision. Storing memories as documents loses the timeline and invites destructive edits.
- **Not a chat log.** A transcript is a stream of turns bound to a conversation. A memory is decoupled from the conversation that produced it - it is what you chose to keep, attributed to who kept it, retrievable long after the chat is gone.
- **Not a database row.** A row needs a schema, a migration to change shape, and a query engine to read. A memory is plain markdown: parseable by a regex today, training data for a model tomorrow, and readable by a human with no tool at all.

This is what "light" means. The unit is deliberately small and unstructured so that capture is cheap and the store outlives any tool that reads it.
