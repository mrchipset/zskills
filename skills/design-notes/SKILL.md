---
name: design-notes
description: Write staged, decision-first design notes while implementing non-trivial features, migrations, refactors, or architecture changes. Use when a change needs durable reasoning (problem, rejected options, chosen approach, trade-offs, accepted risks), when coordinating multi-phase or multi-agent work, when a review finding changes a decision, or when promoting a note to an ADR. Not for task tracking, changelogs, or mechanical edits.
---

# Design Notes

A design note is **engineering memory**: it explains why a change exists, what was
rejected, what risk is accepted, and where the code lives. It is not a diary, a
backlog, or a changelog.

Default to **one note per staged feature or significant decision**, written while the
work is happening and committed with the feature. If the repository has its own note
rules (`AGENTS.md`, `CONTRIBUTING.md`), those win; this skill fills the gaps.

## When to write

- A non-trivial feature, cross-module change, migration, public contract change, or
  architecture choice.
- Several viable options with real trade-offs, or a decision that is expensive to
  reverse.
- Multi-phase or multi-agent work where the reasoning must survive context loss.
- A review finding that changes a decision, confirms a risk, or explains a deviation.
- External facts (library behavior, provider semantics) that later reviewers would
  otherwise have to rediscover.

Skip it for typos, mechanical renames, and one-line fixes. An empty ceremony note is
worse than no note.

## One note per decision

- Location: `notes/<feature-slug>.md` (kebab-case, descriptive).
- Language: English by default; follow the repository if it says otherwise. Keep
  user-facing docs in their required language; notes are engineering reasoning.
- Size: usually 60–150 lines. Bullets, short paragraphs, small diagrams. No giant
  catch-all file.
- Include a status line: `proposed`, `implemented in <path>`, `superseded by <link>`.
- Commit it with the feature (or a `docs:` commit immediately after), never as a
  late retrospective that rewrites history to look prescient.

## Required structure

1. **Title and status** — one sentence on what shipped or is proposed.
2. **Problem and constraints** — the actual problem, boundaries, and repo rules.
3. **Assumptions and open questions** — what is believed, what is undecided, and how
   a wrong assumption would show up.
4. **Options considered** — every serious option, each with why it was rejected.
   Include the chosen option so it does not look like the only idea.
5. **Chosen approach** — the decision, the mechanism, and the invariant it protects.
6. **Trade-offs and risks** — blast radius, accepted risks, performance/operational
   costs, and what remains unverified.
7. **References** — file paths, tests, issues, ADRs, external docs. References must
   resolve.

Optional sections when useful: confirmed external research with source URLs,
migration/rollback notes, rollout/operation steps, follow-ups that are real (not a
wish list).

```markdown
# <Feature>: <decision>

Status: implemented in `path/`.

## Problem and constraints
## Assumptions
## Options considered
**Option A vs option B.** A was rejected because ...
## Chosen approach
## Trade-offs and risks
## References
```

## Workflow

1. **Before coding** — write the problem, constraints, assumptions, and candidate
   options. Research first; record confirmed facts with sources and label guesses.
2. **During implementation** — update the chosen approach when reality diverges.
   Record the deviation and why; do not leave the note describing a design that no
   longer exists.
3. **Before commit** — make the note match the shipped code and tests, remove
   speculation, and check that every reference resolves.
4. **After review** — append accepted findings and remediation decisions. If a fix
   changes the reviewed decision, say so; do not silently rewrite the earlier text.
5. **When behavior changes later** — update the note or mark it superseded and link
   the successor. If docs and code disagree, confirm the real behavior, then fix one
   of them immediately.
6. **Promotion** — when a decision becomes durable, promote it to an ADR,
   `architecture.md`, `design.md`, or the project's equivalent. Keep the note as
   context or delete it; do not maintain two sources of truth.

## Multi-agent and review coordination

- **One owner per note.** Announce ownership before editing; avoid concurrent writes.
- Reconcile research and specialist findings into **accepted facts** before a review,
  so reviewers do not repeat discovery.
- Reviewers add batched, actionable findings; the note records what was accepted and
  what was deliberately deferred.
- Never paste full logs, prompts, customer data, or credentials into a note.

## Quality bar

A future maintainer must be able to answer:

- Why does this exist, and what breaks if it is removed or changed?
- Which alternatives were rejected, and why?
- What is the blast radius, and which risk is explicitly accepted?
- Where is the code, and which test locks the behavior?
- What is still unverified or uncertain?

If the note cannot answer these, it is too vague.

## Anti-patterns

- Notes used as a task tracker, backlog, roadmap, or changelog.
- A single giant `notes.md`; a diary entry per commit.
- Pasted code, diffs, or full logs instead of decisions and references.
- Duplicating the README or copying the architecture document.
- "We chose X" without rejected options, consequences, or risks.
- Recording guesses as facts, or omitting the fact that something is unverified.
- Stale notes that contradict shipped code.
- Secrets, customer names, personal data, or internal URLs.

## Commit checklist

- [ ] One feature/decision per note, with a status line.
- [ ] Problem, constraints, assumptions, and rejected options are present.
- [ ] The chosen approach matches the code and tests in this change.
- [ ] Accepted risks and blast radius are explicit.
- [ ] References resolve; no secrets or customer data.
- [ ] The note is committed with the feature (or a `docs:` commit).
- [ ] Durable decisions are promoted or linked to an ADR/design document.
