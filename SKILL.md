---
name: obsidian-operations
description: Use when working inside an Obsidian vault and the task may create, edit, rename, move, delete, inspect, or reorganize notes, attachments, canvases, folders, or vault metadata. Enforces Obsidian-aware operations so links, trash behavior, history, and vault features are preserved.
---

# Obsidian Operations

Work inside Obsidian vaults without bypassing Obsidian's vault-aware behavior.

## Core Rule

Use normal file edits for precise content changes inside existing files. Use the native `obsidian` CLI for structural vault operations so Obsidian can preserve links, trash behavior, history, and vault state.

Structural operations include creating, renaming, moving, deleting, opening, templating, and inspecting relationships between notes. Do not use raw filesystem operations for those unless the user explicitly asks to bypass Obsidian behavior, because direct file operations can silently skip Obsidian's safeguards.

## Detect Vault Context

Treat a directory as an Obsidian vault when it or an ancestor contains `.obsidian/`.

Before structural changes:

1. Identify the vault root.
2. Prefer vault-relative paths for Obsidian CLI `path=` and `to=` arguments to keep operations scoped to the vault.
3. Check for ambiguous filenames before using name-based operations to avoid changing the wrong note.
4. If the vault is in Git, check status before broad changes.

Avoid editing `.obsidian/`, `.trash/`, plugin state, sync metadata, and generated caches unless the user specifically requests it, because these files store app state, recovery data, and plugin-managed data rather than ordinary note content.

## Operation Rules

### Edit Existing Content

For targeted edits inside an existing note, canvas, or metadata file, use standard read and patch tools. Obsidian vaults are plain files and Obsidian monitors external changes.

Before large edits, read the current file and preserve unrelated user changes to avoid overwriting concurrent edits from Obsidian or another agent.

### Create Notes

Prefer:

```bash
obsidian create path="Folder/Note.md" content="..."
obsidian create path="Folder/Note.md" template="Template Name"
```

Use direct file creation only for simple files that do not need templates or Obsidian indexing during the operation.

### Rename Notes

Use:

```bash
obsidian rename path="Folder/Old.md" name="New"
```

Do not use `mv` for Markdown notes, canvases, or linked attachments, because Obsidian CLI rename can update internal links when the vault setting to update internal links is enabled.

### Move Notes And Attachments

Use:

```bash
obsidian move path="Folder/Old.md" to="Archive/Old.md"
```

Do not use raw filesystem move operations for files that may be linked, embedded, or tracked by Obsidian, because Obsidian-aware moves can preserve link targets and vault metadata.

### Delete Files

Use:

```bash
obsidian delete path="Folder/Note.md"
```

Never use `rm` for vault file deletion, because deletions should go through Obsidian's trash and recovery behavior. Never pass the `permanent` flag unless the user explicitly asks for irreversible deletion after being warned.

`obsidian delete` is trash-first by default. If the user specifically requires Obsidian's `.trash/` folder rather than system trash, verify behavior in the current vault and platform before deleting.

### Append Or Prepend

Prefer Obsidian CLI commands for append/prepend workflows, especially daily notes:

```bash
obsidian append path="Folder/Note.md" content="..."
obsidian prepend path="Folder/Note.md" content="..."
obsidian daily:append content="..."
obsidian daily:prepend content="..."
```

Use direct patches when the edit needs exact placement or multi-line restructuring to avoid forcing content edits through command arguments.

### Inspect Links, Backlinks, And Vault State

Prefer Obsidian's index over raw text search for vault relationships:

```bash
obsidian links path="Folder/Note.md"
obsidian backlinks path="Folder/Note.md"
obsidian unresolved
obsidian orphans
obsidian tags
obsidian tasks
```

Raw content search is useful for finding text, but do not treat text matches as a complete link graph because Obsidian resolves links, embeds, aliases, headings, and unresolved references through its own index.

### History And Recovery

When a task involves recovery, previous versions, or deleted files, prefer Obsidian history and Sync commands before direct filesystem assumptions:

```bash
obsidian diff path="Folder/Note.md"
obsidian history path="Folder/Note.md"
obsidian history:read path="Folder/Note.md" version=1
obsidian history:restore path="Folder/Note.md" version="<version>"
obsidian sync:history path="Folder/Note.md"
obsidian sync:read path="Folder/Note.md" version=1
obsidian sync:restore path="Folder/Note.md" version="<version>"
```

Obsidian File Recovery is local and path-sensitive; Obsidian Sync history is separate. Avoid raw moves/deletes when history continuity matters because path changes can make recovery context harder to preserve.

## CLI Availability

Use the native `obsidian` command when available. If it is missing, Obsidian is not running, or the installed version does not support the needed command:

1. Explain the limitation briefly.
2. Stop and ask the user for guidance before proceeding because the requested preservation behavior may not be available.
3. Do not perform structural vault operations with raw filesystem commands as a fallback, because that bypasses the protections this Skill exists to enforce.
4. For content edits, proceed only if the task is clearly limited to patching existing file contents and does not rely on Obsidian CLI behavior.

## Path Discipline

For Obsidian CLI commands, prefer vault-relative `path="Folder/Note.md"` and `to="Folder/Note.md"` arguments to keep commands portable and scoped inside the vault.

Use `file="Note name"` only when intentionally relying on Obsidian's wikilink-style resolution and after checking for ambiguity to avoid targeting the wrong note.

Quote paths so the CLI receives the exact Obsidian filename as one literal argument. Preserve exact capitalization. Do not normalize spaces or punctuation in filenames unless the user asks.

## Bulk Change Checklist

Before bulk rename, move, or delete operations:

1. List planned target files.
2. Confirm no name or path ambiguity.
3. Inspect backlinks or outgoing links when relationships matter.
4. Check Git status if applicable.
5. Use Obsidian CLI structural commands.
6. Verify the expected files and links after the change.

## When Direct Filesystem Operations Are Acceptable

Direct file operations are acceptable for:

- Reading files.
- Searching file content.
- Applying precise patches to existing notes.
- Creating non-linked scratch files when Obsidian behavior is irrelevant.
- Running formatters or validators that only change note content.

Direct filesystem operations are not acceptable by default for:

- Deleting vault files.
- Renaming notes.
- Moving notes or attachments.
- Bulk reorganizing folders.
- Touching vault settings, plugin state, trash, or sync metadata.
