---
name: git-commit-workflow
description: Commit coding work as verified, rollback-sized Git checkpoints with conventional messages. Use whenever the user asks to commit, stage, write or fix a commit message, split a large feature into incremental commits, decide when to checkpoint during long work, recover from a wrong or failed commit, coordinate commits across parallel agents, or clean generated files out of Git — even if they do not mention "conventional commits."
---

# Git Commit Workflow

Treat a commit as a **verified, coherent checkpoint**, not a save button. A useful
history lets a reviewer answer: what changed, why, what was verified, and which
phase/gate it belongs to.

Default to **rollback-sized checkpoints**: the smallest independently revertible
unit that still leaves the repository buildable and the relevant checks green.
Commit at each verified slice as the work progresses; do not wait until the whole
feature, phase, or milestone is finished. Avoid both one giant end-of-feature
commit and a noisy commit per edit.

## 1. Before committing

Inspect before you stage:

```bash
git status --short
git diff --stat
git diff
git log --oneline -10
```

Check three things:

1. **Scope** — every changed path is intentional. Revert debug edits, scratch
   files, editor settings, and unrelated formatting.
2. **Verification** — run the focused check that proves the change: build,
   targeted tests, conformance suite, doc link check, or artifact hash check.
   Do not commit code with failing checks on a shared branch.
3. **Generated artifacts** — builds, caches, logs, binaries, coverage output,
   `__pycache__/`, and release artifacts belong in `.gitignore`, not in Git.

If a generated file appears, fix the ignore rule first, then stage again:

```bash
git rm --cached path/to/generated
# add the pattern to .gitignore
```

## 2. Stage deliberately

Prefer explicit paths:

```bash
git add src/core/server.cpp tests/unit/server_test.cpp docs/roadmap.md
git diff --cached
```

Use `git add -A` / `git add .` only after reading **every** path in
`git status --short`. In multi-agent work, other agents may have uncommitted
changes in the same worktree; blind staging can capture their work.

Never stage:

- `build/`, `build-*`, `out/`, `dist/`, coverage/benchmark output
- `__pycache__/`, `*.pyc`, logs, temporary files
- credentials, local config, editor state
- another agent's unreviewed partial work

## 3. Commit granularity and phase history

One commit should be one coherent change that can be reviewed and reverted
independently. Do not mix an unrelated refactor with a feature or a bug fix.

For multi-phase or gated work, keep the phase story visible:

1. **Implementation** — feature/fix plus its tests.
2. **Review remediation** — separate `fix(...)` commit after a review/gate,
   containing only the accepted findings and focused evidence.
3. **Documentation/roadmap** — separate `docs(...)` commit that records status,
   decisions, test counts, and next steps.

That pattern makes it possible to see what was implemented, what review changed,
and what was merely recorded. Do not squash remediation into the feature commit
unless the user explicitly asks for a single commit.

### Rollback-first commit sizing

Before committing, ask: **if this one commit is reverted, does the repository
stay buildable and does the revert undo exactly one behavior?** If not, split it.

- Commit after each green vertical slice (schema/parser, one endpoint, one client
  path, its tests, its docs) even when the overall requirement is still in
  progress.
- A phase/gate is a **review boundary**, not necessarily a commit boundary. A
  large phase should usually contain several checkpoints.
- For long or multi-day work, save verified checkpoints early. A private branch
  may hold intentionally incomplete WIP, but prefer commits that build.
- Do not postpone all commits until "the feature is done": that creates a large,
  hard-to-revert blob and loses the ability to bisect or revert one slice.
- Do not over-split into mechanical micro-commits that cannot stand alone. Each
  commit must be coherent, reviewable, and green.

Example of slicing a large feature instead of one final blob:

```text
feat(protocol): parse resultType on tool results
feat(server): emit ttlMs/cache fields for 2026 reads
feat(client): accept the 2026 discover response
test(protocol): cover MRTR request/retry pairing
docs(protocol): record the wire decision
```

If a slice cannot be made green independently, keep it on a private branch and
commit the smallest green prerequisite first; do not merge a half-working slice
into the main history merely to make the log look granular.

Use a `wip(...)` commit only on a private scratch branch. Do not publish WIP
history as the final story of a change.

## 4. Commit message format

```text
<type>(<scope>): <imperative summary>

Optional body: why this change exists, a non-obvious constraint,
or the verification evidence a reviewer should know.
```

- **Type**: `feat`, `fix`, `test`, `docs`, `chore`, `refactor`, `perf`, `build`.
- **Scope**: the component, not the file: `core`, `protocol`, `http`, `server`,
  `client`, `executor`, `security`, `legacy`, `conformance`, `amalgamation`,
  `examples`, `roadmap`, `docs`.
- **Summary**: imperative, lowercase after the colon, no trailing period,
  roughly ≤ 72 characters. State the behavior, not "update files".
- Omit the scope when it adds no information (`chore: ignore python bytecode`).
- Use the body for rationale/tradeoffs; do not paste full logs.

Examples from a real phase-gated project:

```text
feat(protocol): align 2026 wire with official conformance (D18)
fix(executor): freeze IExecutor contract and self-call shutdown
test(conformance): add fixture server and runner; clear 2025/2026 required suites
docs(roadmap): record M2 completion and Gate 2b
chore: ignore python bytecode
```

Avoid:

```text
fix
update stuff
wip
feat: lots of changes
```

## 5. Multi-agent coordination

Before committing in a shared worktree:

- inspect `git status` and `git diff` for files you do not own;
- stage only your lane's paths;
- reconcile or wait for other lanes instead of committing their partial work;
- do not run `git reset --hard` or discard changes you did not create;
- if a commit accidentally includes another agent's work, coordinate before
  rewriting history.

## 6. Amend, force, and hooks

- Do not amend or force-push **published** commits.
- Do not bypass hooks with `--no-verify`.
- If a hook or commit fails, fix the cause and create a **new** commit.
- Amend only when the user explicitly asks and the commit is local/unpublished.
- Force-push only on explicit user request, never as a convenience.

These rules protect collaborators and keep review evidence intact.

## 7. Recovery

**Wrong files staged**

```bash
git restore --staged path/to/file
git add path/to/intended-file
```

**Generated file already committed**

```bash
git rm --cached path/to/generated
# update .gitignore
git commit -m "chore: ignore generated artifacts"
```

**Commit message is wrong but not pushed**

Ask before rewriting. If approved, amend the local commit; otherwise add a
follow-up commit describing the correction.

**A commit was made on the wrong branch or with mixed work**

Inspect first (`git log`, `git show`, `git status`). Prefer a corrective commit
over history rewriting. Never `reset --hard` without confirming the user has no
uncommitted work to lose.

## 8. Pre-commit checklist

```bash
git status --short                 # only intended paths
git diff --stat                    # expected surface
git diff                            # real content
git log --oneline -5                # message/style context
# run focused verification here
git add <explicit paths>
git diff --cached                   # final staged review
# can this commit be reverted alone and keep the build green?
git commit -m "<type>(<scope>): <summary>"
git status --short                  # clean or intentionally remaining
git log -1 --stat                   # confirm the checkpoint
```

If the change spans implementation + docs, ask whether the user wants one
release commit or separate phase commits; default to the repository's existing
history style.

## 9. Anti-patterns

- `git add -A` without reading `git status`.
- Committing build output to "make it work."
- Mixing a refactor, a feature, and a roadmap update in one commit.
- Saving all work until the whole feature/milestone is done, then committing one
  large blob.
- Committing only at phase gates when smaller green slices were available.
- Hiding failed verification in a WIP commit.
- Amending published history to make a review finding disappear.
- Committing another agent's uncommitted files.
- Writing a message that says what files changed but not what behavior changed.
