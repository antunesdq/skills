---
name: knowledge
description: Read from and write to the durable knowledge store at ~/projects/knowledge — architecture, decisions, project state, ownership, open questions — using Graphify to query it. Use when a decision or architecture fact lands in conversation and should outlive the session, when asked to "remember this", "add this to the graph", "graphify this", or when answering a question about how a system works, why something was chosen, who owns something, or what the current state of a project is.
---

# Knowledge store

`~/projects/knowledge` is the durable store. Notes are markdown; their frontmatter compiles to
a Graphify graph via `build.py`. `~/projects/knowledge/CONVENTIONS.md` is the schema contract —
read it before writing your first note in a session.

`graphify` lives at `~/.local/bin/graphify`. If it is not on PATH:
`export PATH="$HOME/.local/bin:$PATH"`.

## Read before you answer

When the question is about how a system works, why something was chosen, who owns something,
or where a project stands, query the store before reasoning from scratch:

```bash
cd ~/projects/knowledge
graphify query "<the user's question>" --budget 1500
graphify explain "<node label>"          # one node + its neighbours
graphify path "A" "B" --undirected       # how two things connect
graphify affected "<node>" --relation DEPENDS_ON --relation RELATES_TO   # what depends on this
#   ^ the --relation flags are REQUIRED: `affected` defaults to code relations
#     (calls, imports, inherits...) and finds nothing in this graph without them
graphify god-nodes --top 10              # the hubs
cat INDEX.md                             # everything, by type
```

Cite what you found as `file:line` from the `src=`/`loc=` fields — the note is the source, the
graph is only the finder. Read the note itself before relying on it; the graph carries labels
and edges, not the reasoning in the body.

Treat `OpenQuestion` nodes and anything marked `unconfirmed` as **not established**. Never
quote them back as fact.

## Write as it happens

Do not wait for the end of the session and do not ask permission for each note — writing a
note is the standing instruction (`dec:2026-09-02-capture-as-we-go`).

**Write a note when:**

- A decision is made, or an earlier one is reversed → `decisions/`
- How a system works gets established or corrected, especially a non-obvious behaviour or a
  gotcha someone would otherwise rediscover → `architecture/`
- A project's goal, shape or status changes → `projects/` (edit the existing note in place;
  project state is current-state, not a log)
- Ownership, or who to ask about something → `people/`
- An external doc, paper, ticket or dashboard turns out to matter → `reference/`, or
  `graphify add <url>` to fetch it into `raw/`
- Something load-bearing turns out to be unknown → `questions/`

**Do not write:** transient debugging narration, task-level todos, anything already recorded
in a repo or its git history, or a restatement of a note that already exists — extend that
note instead.

### How to write one

1. `cat ~/projects/knowledge/CONVENTIONS.md` for the frontmatter schema and id prefixes.
2. Check for an existing note first: `grep -rl "<topic>" ~/projects/knowledge --include="*.md"`
   or `graphify query "<topic>"`. Extend rather than duplicate.
3. Write the file. `label` must be a full searchable claim, not a title — `graphify query`
   string-matches labels to pick traversal start nodes, so "Taxonomy DAG is serial across days
   because depends_on_past is set" beats "DAG notes".
4. Link it: `relates_to` / `depends_on` / `supersedes` / `owner`. A relationship stated only in
   prose is invisible to the graph.
5. Set `confidence` honestly: `verified` only if read off a repo, a run or a primary source.
6. Build and commit:

```bash
cd ~/projects/knowledge && python3 build.py && git add -A && git commit -m "<what was learned>"
```

`build.py` is deterministic and offline. It exits non-zero on schema errors and on links to
ids that do not exist, and still writes the graph — a dangling target becomes a `MISSING: <id>`
stub, which is a to-do, not a failure. Fix real errors before moving on.

## Never overwrite a decision

Superseding a decision means a **new** note with `supersedes: [dec:old]`, and setting the old
note's `status: superseded`. The graph should show how the thinking changed, not just where it
landed.

## Never point graphify's own extractor at this repo

Do not run `/graphify`, `graphify extract`, or `graphify hook install` inside
`~/projects/knowledge`.

- `/graphify .` and `graphify extract .` overwrite `graphify-out/graph.json` with a code-only
  AST extraction. Markdown extraction needs an LLM backend key this machine lacks, so the
  result is near-empty. **Recovery: `python3 build.py`** — it rebuilds byte-identically from
  the notes, so a clobber costs one command.
- `graphify hook install` would replace this repo's `post-commit` hook with one that runs
  `graphify update`, i.e. the clobbering path on every commit.

Graphify's own skill is installed user-wide (`~/.claude/skills/graphify/`); `/graphify` is the
right tool pointed at the *code* repos, e.g. `~/projects/data-refinenet-models`.
