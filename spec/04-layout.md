# Chapter 4 - The layout

The layout is how entries, routes, and system files are arranged on disk. It is folder-based and human-navigable: the file system is the index. This chapter defines the content root, the `vault.md` manifest, folder-as-route semantics, the type folders, and the `_system/` locations.

Everything in this chapter is a durable open-format fact. It says nothing about any product's private features; a vault is just markdown files in named folders.

## 4.1 The content root

All LogMark content lives under a `logmark/` directory. Given a vault path, the content root is resolved as:

- `<vault>/logmark` if that directory exists (a **wrapped** vault, sharing its top directory with other tools), otherwise
- `<vault>` itself (a **flat** vault).

A conformant reader MUST apply this rule so that a vault can either stand alone or nest inside a larger workspace. Throughout this chapter, paths are written relative to the vault path and therefore begin with `logmark/`.

## 4.2 The `vault.md` manifest

The vault's constitution is a single markdown file, `vault.md`, at the content root (`logmark/vault.md` in a wrapped vault). It is the anchor artifact, and it is where a reader learns what the vault is for.

- A reader MUST prefer `vault.md`. For backward compatibility it MAY fall back to a legacy `constitution.md` at the same location if `vault.md` is absent.
- The manifest is OPTIONAL. A vault with no manifest is valid; its constitution is simply empty.

The manifest is parsed as markdown with optional YAML frontmatter. The recognized structure is:

| Element | Meaning |
|---|---|
| The first `#` H1 | The vault name |
| The first paragraph after the H1 (or, when a heading comes first, the first paragraph of a `## Purpose` section) | The vault purpose |
| `## Scope` | Scope prose with optional `INCLUDE:` / `EXCLUDE:` lines naming `+routes` and `#tags` |
| `## How agents should help` | Guidance for agents operating on the vault |
| `## Frozen vault-level decisions` | Standing decisions that hold vault-wide |
| `## Don't surface` | Things a reader should keep quiet about |
| `## Conventions` | House rules for the vault |

- Section matching is by H2 title, case-insensitively.
- **Leniency:** YAML frontmatter is lexically delimited by `---` fences and is not required to be valid YAML; a reader MUST NOT abort manifest parsing because frontmatter is malformed. The manifest body (name, purpose, sections) is still extracted. The `schema:` field declares the format version the vault targets (`SPEC-VERSION.md`); a vault SHOULD carry it and a writer MUST preserve it. Other frontmatter fields are advisory and MUST be preserved by a writer that rewrites the manifest.
- A manifest with none of the recognized elements parses to an empty constitution, which is valid.

## 4.3 Type folders and folder-as-route

Entries are stored in files named by year, inside a **type folder**, inside a **route**.

### Type folders

Each entry type has an underscore-prefixed folder:

| Type | Folder |
|---|---|
| Task | `_tasks` |
| Idea | `_ideas` |
| Block | `_blocks` |
| Decision | `_decisions` |
| Question | `_questions` |
| Gap | `_gaps` |
| Analogy | `_analogies` |
| Log | `_diary` |

The folder determines the type of entries within it: an entry under `_tasks/` is a task even with no `task:` prefix on its line (chapter 2, section 2.1). A reader MUST derive the type from the type folder when no prefix is present, and SHOULD treat folder and prefix as agreeing when both are present.

### Files are years

Within a type folder, entries are collected into per-year files: `2026.md`, `2027.md`. A conformant reader locates entries by walking type folders and reading these year files; the filename's year is the coarse temporal bucket, and each entry's own `## header` carries its exact date.

### Folder-as-route

The path segments between the content root and the type folder are the entry's **route**:

```
logmark/<route>/_<type>/<year>.md
```

- `logmark/projects/refinery/_tasks/2026.md` - tasks routed to `projects/refinery`.
- `logmark/domains/travel/_ideas/2026.md` - ideas routed to `domains/travel`.

The route of a stored entry is its folder location, full stop. An inline `+route` token (chapter 2) is a routing *instruction* consulted at capture time to choose the destination folder; once written, the entry's route is where it lives, and a reader MUST read the route from the path, not re-derive it from body text. This is the meaning of "folder-as-route": the directory tree is the routing index.

### The inbox: unrouted entries

Entries with no route live under a reserved `_inbox` segment:

```
logmark/_inbox/_<type>/<year>.md
```

`logmark/_inbox/_diary/2026.md` holds unrouted log entries. A reader MUST treat `_inbox` as "no route," not as a route named "inbox."

## 4.4 The `_system/` folder

Vault-level configuration and indexes live under a reserved `_system/` folder at the content root. The locations a conformant reader relies on:

| Path | Contents |
|---|---|
| `logmark/_system/routes.md` | The route registry (section 4.5) |
| `logmark/_system/tags.md` | The tag registry: known tags and optional clusters |
| `logmark/_system/config.md` | Vault configuration |
| `logmark/_system/log.md` | A human-readable capture audit trail |

`_system` and `_inbox` are reserved segments: a reader MUST NOT interpret either as a user route or a type folder. Additional files a tool keeps under `_system/` (indexes, caches, palettes) are implementation detail and outside this contract; a reader MUST tolerate their presence and MUST NOT treat unknown `_system/` entries as content. This includes any version stamp an implementation keeps for its own on-disk structure (the reference implementation's is `_system/vault.json`): such a stamp is private to that implementation and is not the format version. The only format version declaration is the manifest's `schema:` field.

## 4.5 The route registry

`logmark/_system/routes.md` declares the vault's routes. It is markdown organized by H2 sections (conventionally `Projects`, `Domains`, `Archived`), with each route an H3 heading followed by `- key: value` property lines:

```markdown
# Routes

## Projects

### refinery
- path: projects/refinery
- description: The knowledge refinery
- status: active
- tags: #ml #knowledge
```

Recognized route properties:

| Property | Meaning |
|---|---|
| `path` | The vault-relative route root; a route needs a `path` to receive entries |
| `description` | Human-readable description |
| `status` | `active`, `building`, `paused`, `archived` |
| `tags` | Space-separated `#tags` associated with the route |
| `scopes` | Named sub-scopes, as an indented `- name: description` block |

- The route **type** is inferred from its section header, not a field.
- A route with no `path` is a reference-only entry (it names a route without a storage location).
- Prose before the first `## section` is read as vault-level notes; prose after a section header, before its first route, is that section's notes.
- The registry is OPTIONAL. Routes can exist as folders without a registry entry; the registry adds description, status, and grouping.

## 4.6 What the layout guarantees

Because the file system is the index:

- A person can navigate the whole vault with a file browser and read any entry with no tool.
- A route is a folder you can move, archive, or back up as a unit.
- The layout anchors - the `logmark/` content root, `_system/`, `_inbox`, folder-as-route, the `vault.md` manifest - are stable within format MAJOR version 1 (`SPEC-VERSION.md`). A tool can rely on them without a schema migration.
