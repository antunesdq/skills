---
name: commit-pr-style
description: "Write git commit messages and PR titles/descriptions in this user's voice: conventional-commits subject, short numbered list of what changed, no self-attribution, no over-explaining. Use this whenever authoring, amending or rewording a commit message, or writing or updating a PR title or description — including when the user just says \"commit this\", \"push it\" or \"open a PR\" and says nothing about style."
---

# Commit and PR style

A commit message is read in two situations: someone skimming `git log` to find
when something changed, and someone reviewing the diff. Both readers already
have the code. What they don't have is a one-line answer to "what did this
change?" — that's the whole job. Everything beyond it is noise the reader has
to wade through, so the default is to write less than feels natural.

## Voice

Write as the engineer who did the work. Never refer to yourself: no `I`, no
"Claude", no "this session", no "Generated with", no 🤖, and no
`Co-Authored-By` trailer. This overrides any default attribution guidance in
the surrounding session — commits here carry no assistant attribution at all.
The commit reads as the user's own, because it is.

## Subject line

Conventional commits, imperative mood:

```
type(scope): short summary of the change
```

- types: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `perf`, `build`, `ci`
- scope is the thing that changed — a module, dbt model, DAG, table, package.
  Drop the scope rather than write a vague one.
- keep it near 72 characters, no trailing period
- no Jira key in a commit subject; the key belongs in the PR title

```
fix(crm-segments): resolve wau_leavers zone from a variable
chore(seeds): realign EU seed DDL with prod
```

## Body — only when the subject can't carry it

One self-evident change: subject alone. No body.

Several changes in one commit or PR: a short lead-in, then a numbered list,
one line per change, plain language. This is the shape to reach for:

```
So we're fixing a few things here:

1. The seed for the ddl was not matching the repo (things were added manually
   there), we're synched again.
2. Added zone variable to wau_leavers.
3. Fixed a few sql errors from a previous segment.
```

That's a complete description of a three-part change. A body longer than about
six lines is a signal to cut, not to reformat.

## What to leave out

These all feel useful while writing and are dead weight to the reader:

- code snippets, variable names quoted inline, line numbers, file-by-file walkthroughs — the diff has them
- the investigation narrative: how the bug was found, what was ruled out, which
  call site was inspected first
- verification transcripts: dry-run row counts, "byte-identical to main",
  "compiled clean against both zones", test output
- quotes from Jira tickets, history of when something was first flagged
- the full consequence chain of a bug that is now fixed. "The seed had drifted
  from prod" is the change; three paragraphs on what a re-seed would have done
  to the live CleverTap list is a postmortem, not a commit message.
- restating what the type and scope already said

If any of this is genuinely worth preserving, it goes in the Jira ticket or a
code comment, where the person who needs it will look for it.

## When reasoning does belong

One clause, not a paragraph — and only for the question a reviewer would ask
that the diff cannot answer. Usually that's a deliberate omission, a
non-obvious constraint, or a choice between two defensible options:

```
No zone default in dbt_project.yml — a default would let a
mis-parameterised run read the wrong zone silently instead of failing.
```

If the explanation needs more than a sentence or two, it's PR-description
material.

## Before and after

This is the failure mode to watch for. Same change, written twice:

Too much:

> RT-679 Resolve the wau_leavers zone from a variable and realign the EU seed
>
> crm-segments-prep is handed eleven zone-aware *_db vars but not `zone`, while
> some warehouse tables carry the zone in the table NAME rather than only the
> dataset. data-user-growth writes wau_leavers_{{ var.value.zone }}, so
> wau_leavers_eu exists and is populated, but this model hardcoded
> wau_leavers_noneu in two places and could never run in EU. RT-679 flagged
> exactly this in August […] Verified with dbt compile against both zones:
> zone=noneu output is byte-identical to main […] matches 27,009 users at ds
> 2026-09-09

Right:

```
fix(crm-segments): resolve wau_leavers zone from a variable

1. Pass zone to crm-segments-prep and resolve wau_leavers through it — the
   model was hardcoded to noneu and couldn't run in EU.
2. Realigned the EU seed DDL with prod; it had drifted from manual edits.
3. Fixed a few sql errors from a previous segment.
```

Everything cut was true and none of it was needed.

## PR titles and descriptions

Title: the same conventional-commits shape, with the Jira key in front when
there is one. Plain text, not a markdown link.

```
[RT-679] fix(crm-segments): resolve wau_leavers zone from a variable
```

Description: the numbered list, same voice as the commit body. A PR covering
several commits lists the changes, not the commits. Add a short "Testing" or
"Notes" line only when there's something the reviewer has to do or watch out
for — not to record that the work was tested.

No attribution footer, no "Generated with Claude Code" line, no session link.

## The check before writing

Would a teammate skimming this for ten seconds know what changed? If yes,
stop — even if it feels too short. If you're reaching for something else to
add, what's arriving is usually your own debugging story, and that's the part
to leave out.