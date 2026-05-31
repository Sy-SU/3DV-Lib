## Literature Management Rules

This project is an Obsidian-based literature knowledge base.

### Directory conventions

- `文献列表.md` is the original user-maintained source list. Never delete or overwrite it.
- `Literature/Papers/` contains one canonical Markdown file per paper.
- `Literature/Topics/` contains topic navigation files.
- `Literature/Surveys/` contains cross-paper synthesis notes.
- `Literature/Literature Index.md` is the global navigation index.
- `Literature/unresolved.md` records ambiguous or conflicting items.
- `.base` files are Obsidian Bases view configurations. Do not delete or rename them without explicit user approval.

### Canonical paper rule

- Each paper must have exactly one canonical file in `Literature/Papers/`.
- A paper may appear in multiple topic indexes, but all links must target the same canonical file.
- Before creating a paper file, search existing files by title, short name, URL, DOI, arXiv ID, and normalized title.
- Never create duplicate paper notes.

### Status fields

Allowed `status` values:

- `unread`
- `reading`
- `skimmed`
- `needs-review`
- `fully-summarized`
- `reference-only`

Allowed `summary_level` values:

- `none`
- `abstract`
- `full`

Allowed `source_level` values:

- `none`
- `abstract`
- `fulltext`
- `fulltext-and-code`

Allowed `priority` values:

- `high`
- `medium`
- `low`

### Source discipline

- Do not infer full-paper details from an abstract.
- If only an abstract is available, set:

```yaml
source_level: abstract
summary_level: abstract
```

- If the full paper has been obtained and read, set:

```yaml
source_level: fulltext
```

- Set:

```yaml
summary_level: full
status: fully-summarized
```

only after a structured full-paper summary has actually been written.
- Never invent authors, venues, DOI values, URLs, experimental numbers, equations, conclusions, limitations, or BibTeX entries.
- Every experimental number must be traceable to the source paper.
- Separate paper claims from your own analysis.
- When uncertain, record the issue under `## 待确认问题` and in `Literature/unresolved.md`.

### User-authored content

- Preserve all existing user notes.
- Do not rewrite or delete `## 用户原始备注`.
- If new content conflicts with existing user notes, record the discrepancy instead of silently replacing content.

### Markdown and Obsidian conventions

- In Markdown tables, use escaped Obsidian aliases:

```markdown
[[Papers/DDPM\|DDPM]]
```

- Outside Markdown tables, normal Obsidian links are allowed.
- Do not introduce unnecessary alignment spaces inside Obsidian links.
- Keep YAML Front Matter valid.
- Keep filenames stable unless explicit approval is given.

### Network access

- Do not access the network unless the user explicitly starts a literature retrieval task.
- Before downloading any paper, state the intended sources and processing batch.
- Prefer official paper pages, arXiv, conference pages, journal pages, and author project pages.
- If the full text is unavailable, do not pretend that an abstract is a full paper.
- Process papers in small batches.
- Update indexes and status fields after each batch.

### Completion checks

After each literature-processing batch:

1. Confirm that no duplicate paper file was created.
2. Confirm that index links remain valid.
3. Update `status`, `summary_level`, and `source_level`.
4. Record unresolved issues.
5. Report which sources were accessed.
6. Report which files were created or modified.

### Mathematical notation in Obsidian

- Use Obsidian-compatible MathJax delimiters.
- Use `$...$` for short inline formulas.
- Use separate `$$` lines for display formulas.
- Do not use `\(...\)` or `\[...\]` in Markdown notes.
- Prefer display formulas for equations in `## 关键公式`.
- Use `\mid` for conditional probability inside formulas instead of a raw `|`.
- Use `\mathcal{N}` for a Gaussian distribution when appropriate.
- Use `\mathbb{E}` for mathematical expectation when appropriate.
- Wrap multi-letter textual subscripts with `\mathrm{}`, for example `L_{\mathrm{simple}}`.
- Never modify formulas inside fenced code blocks or inline code.
- Never confuse mathematical `|` with Markdown table separators or Obsidian link aliases.

### Graph hygiene and management views

- `Literature/Literature Index.md` and `Literature/Workflow/Retrieval Queue.md` are management notes, not paper knowledge nodes.
- Do not place bulk `[[Papers/...]]` links in management Markdown files.
- Use Obsidian Bases for large filtered tables and queue views.
- Keep semantic links between paper notes when they represent citations, prerequisites, extensions, comparisons, or conceptual relationships.
- Keep curated topic navigation links in `Literature/Topics/`.
- Do not generate a Markdown table containing links to every paper unless the user explicitly requests it.

### Retrieval queue selection

- Do not parse a bulk Markdown table to select the next literature batch.
- Select the next batch by scanning YAML Front Matter in `Literature/Papers/*.md`.
- Skip papers where:

```yaml
summary_level: full
```

- Sort candidates by:
  1. `priority`
  2. `status`
  3. stable filename
- Process at most 5 papers per batch.
- Update each paper's YAML Properties after processing.
- Use `Literature/Workflow/Retrieval Log.md` for historical records.
- Use `Literature/Workflow/Retrieval Queue.base` as the visual queue in Obsidian.


### Backup file conventions

- Backup files must not remain as `.md` files inside the Obsidian Vault if they contain bulk links.
- Store migration backups under `Literature/Workflow/Backups/`.
- Use the `.md.bak` suffix for Markdown backups, for example:

```text
Literature Index.before-base-migration.md.bak
```

- Do not create `.md` backup copies that Obsidian may index as knowledge nodes.
- Do not delete backups unless the user explicitly requests deletion.
- Before creating a backup, check for filename conflicts and use an incrementing suffix when needed.

## 3DV-Lib Repository Rules

### Repository boundary

- The repository root is the current `3DV-Lib/` directory.
- All literature-related operations must remain inside this repository.
- Do not read, modify, move, delete, or initialize Git in any parent directory.
- Do not modify files outside `3DV-Lib/`.
- Do not treat the parent Obsidian Vault as part of this project.

### Obsidian configuration

- `.obsidian/` is part of this repository.
- Preserve and commit the complete `.obsidian/` directory.
- Do not ignore `.obsidian/workspace.json`.
- Do not ignore `.obsidian/workspace-mobile.json`.
- Do not silently rewrite Obsidian configuration files.
- Before committing `.obsidian/`, check for files that may contain credentials or private tokens.

### PDF policy

- `Literature/PDFs/` is local-only.
- Never stage, commit, or push files under `Literature/PDFs/`.
- Preserve local PDFs during literature processing.
- Keep `pdf_path` in paper YAML as a repository-relative local path.
- Keep `retrieved_from` URLs so PDFs can be downloaded again on another machine.

### Backup policy

- Store local backups under `Literature/Workflow/Backups/`.
- Do not commit local backups.
- Do not create `.md` backups that may pollute the Obsidian graph.
- Use `.md.bak` for Markdown backup copies.

### Git preparation

- Do not initialize Git unless the user explicitly requests it.
- Do not add a GitHub remote unless the user explicitly requests it.
- Do not commit or push unless the user explicitly starts the Git synchronization step.
- Never store credentials, personal access tokens, passwords, cookies, or SSH private keys in project files.

## 3DV-Lib Path and Graph Hygiene Rules

### Current repository root

- The current working directory is:

```text
/Users/susenyang/Documents/Obsidian Vault/3DV-Lib
```

- All literature operations must remain inside this directory.
- Do not modify the parent Vault:

```text
/Users/susenyang/Documents/Obsidian Vault
```

- Prefer repository-relative paths in YAML and Markdown.
- Do not store absolute local paths unless strictly necessary.
- Use:

```yaml
pdf_path: "Literature/PDFs/<stable-filename>.pdf"
```

  instead of an absolute PDF path.

### Graph hygiene

- `Literature/Literature Index.md` is a navigation page.
- Do not add bulk `[[Papers/...]]` links to `Literature Index.md`.
- `Literature/Workflow/Retrieval Queue.md` is a workflow instruction page.
- Do not add bulk `[[Papers/...]]` links to `Retrieval Queue.md`.
- `Literature/Workflow/Retrieval Log.md` is a historical log.
- In `Retrieval Log.md`, write paper names as plain text, not Wikilinks.
- Use `.base` files for dynamic management tables.
- Keep semantic Wikilinks in `Papers/`, `Topics/`, and `Surveys/`.
- Do not create `.md` backup files containing bulk links.
- Use `.md.bak` for local Markdown backups.

### Retrieval workflow

- Select the next literature batch by scanning YAML Front Matter in:

```text
Literature/Papers/*.md
```

- Do not parse a bulk Markdown queue table.
- Do not rewrite `Literature Index.md` or `Retrieval Queue.md` with all papers.
- After each batch, update paper YAML Properties.
- Append a plain-text paper name to `Retrieval Log.md`.
- Do not create new management-type high-degree graph nodes.
