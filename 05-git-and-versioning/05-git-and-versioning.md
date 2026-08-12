# ⚙️ Git & Versioning

> **Traceability is non-negotiable.** Git history is not a log; it is a narrative of how the system evolved. If a future engineer cannot reconstruct the "why" from `git log`, documentation has already failed.

This document defines the commit conventions, branching strategies, and versioning discipline extracted from actual production repositories (`SentinalMesh`, `Milan`, `DevTrace`, `Omega`).

---

## 🧭 The Core Philosophy

Before applying the rules, understand the constraints that govern them:

> [!IMPORTANT]
> 1. **Commits Are Immutable History:** Every commit message must be semantic enough to stand alone in `git log --oneline` and `git blame`.
> 2. **Conventional Commits Enable Automation:** Enforced commit formats unlock auto-generated CHANGELOGs, semantic version bumps, and rollback choreography.
> 3. **Branch Names Reflect Intent:** The branch name should tell you what changed and why without opening a diff.

---

## 1️⃣ Conventional Commits

_Reference Repos: SentinalMesh, Design Dungeons_

This is the enforced commit format across all repositories. It is strictly structured so that `standard-version` or `semantic-release` can generate release notes without human intervention.

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Allowed Types

| Type | Purpose | Example |
|------|---------|---------|
| `feat` | A new feature | `feat: add webhook signature verification for QStash delivery` |
| `fix` | A bug fix | `fix: prevent infinite retry loop in Kafka consumer` |
| `chore` | Maintenance tasks (deps, tooling, CI) | `chore: bump Node.js to v20.11.0` |
| `refactor` | Code change that neither fixes a bug nor adds a feature | `refactor: extract CQRS bus registration into init sequence` |
| `test` | Adding or correcting tests | `test: add integration suite for ingestion pipeline` |
| `docs` | Documentation only changes | `docs: add ADR-003 for Milan dual-auth` |
| `perf` | A code change that improves performance | `perf: switch executor map to precomputed lookup for O(N×R)` |
| `revert` | Reverts a previous commit | `Revert "feat: add experimental caching layer"` |

> [!WARNING]
> **NO bare descriptions.** A commit like `updates` or `fix stuff` is a violation. If you cannot summarize the change in one imperative sentence, the commit is too broad.

### Breaking Changes

A `BREAKING CHANGE` footer signals a major version bump:

```text
feat: migrate event store schema to v2

BREAKING CHANGE: Event payloads now require ISO-8601 timestamps.
Consumers using epoch milliseconds must update their parsers.
```

---

## 2️⃣ Branch Naming

_Reference Repos: Milan, DevTrace, Omega, SentinalMesh_

Branch names encode **scope** and **intent**, separated by a slash.

### The Standard Pattern

```text
<type>/<short-description>
```

### Examples

| Branch | Purpose | Source |
|--------|---------|--------|
| `feature/auth-layer` | New auth middleware | Milan |
| `fix/memory-leak` | GC pressure fix | DevTrace |
| `chore/dependency-bump` | Patch dependencies | Design Dungeons |
| `refactor/bernoulli-recall-metrics` | Metrics calculation refactor | SentinalMesh |

### Namespaced Branches (Multi-Contributor Repos)

When multiple engineers work on the same repo, prefix the type with a namespace:

```text
<username>/<type>/<description>
```

| Branch | Context | Source |
|--------|---------|--------|
| `pd241008/data` | Data pipeline work | SentinalMesh |
| `pd241008/dev` | Development branch | SentinalMesh |
| `pd241008/orchestration` | Orchestrator implementation | SentinalMesh |

> [!NOTE]
> This prevents merge conflicts on long-lived branches and keeps `git branch` output readable by grouping work by owner.

---

## 3️⃣ Commit Body & Footers

_Reference Repos: SentinalMesh, Design Dungeons_

If the "why" cannot fit in the subject line, the body is mandatory. The body explains the motivation and contrasts with the previous behavior.

```text
fix: correct recall calculation for Bernoulli window

Previously, the recall denominator included flows with zero targets,
inflating recall to 1.0 for low-target windows. This change isolates
flows with at least one target event and scopes the matched-counterfactual
control to the same category before computing the fraction.

Closes #42
```

### Footer Conventions

| Footer | Usage | Example |
|--------|-------|---------|
| `Fixes #123` | Closes an issue on merge | `Fixes #42` |
| `Refs #123` | References an issue without closing | `Refs #88` |
| `Closes #123` | GitHub/GitLab auto-close | `Closes #15` |
| `BREAKING CHANGE:` | Signals major version | `BREAKING CHANGE: drop support for Node 18` |
| `Reviewed-by:` | Review attribution | `Reviewed-by: Kilo <kilo@ai>` |

---

## 4️⃣ Linking Commits to Documentation

_Reference Repos: SentinalMesh_

The history of a project should reference its documentation, not bury it. SentinalMesh demonstrates this pattern by using commit messages that link to the evolving artifacts.

```text
b83d195 Add diagrams to post mortem and link in README
2444ab4 Fix latency metric, add cold-start and clustered evaluation, update documentation
d9db65a Update README to explain purpose and match latest simulator features
0f24809 Fix mermaid syntax parsing errors in post_mortem.md
c05a247 Merge pull request #9 from pd241008/dev
```

> [!TIP]
> When a documentation file (`README.md`, `post_mortem.md`, `ADR-NNN.md`) is created or updated, include it in the commit body. This creates an auditable trail from code change → documentation update → decision record.

---

## 5️⃣ The Decision Framework

_Reference: Engineering Standards_

Before pushing a commit, run it through this checklist:

1. **Is the subject line imperative and under 50 characters?**
2. **Does the type match the change** (`feat` for behavior change, `fix` for bugs, `chore` for maintenance)?
3. **Does the body explain the "why," not just the "what"?**
4. **Are linked issues or ADRs referenced in the footer?**

> [!NOTE]
> Automated CI checks should fail if the commit message does not match the Conventional Commits regex: `^(feat|fix|chore|refactor|test|docs|perf|revert)(\(.+\))?: .{1,50}$`

---

## 🚫 Anti-Patterns

> [!WARNING]
> **DO NOT DO THE FOLLOWING:**
>
> - Committing secrets, API keys, or `.env` contents.
> - Using `git push --force` on shared branches (force-push is acceptable only on feature branches before review).
> - Writing commit messages in past tense (`fixed`, `added`, `updated`). Use imperative mood (`fix`, `add`, `update`).
> - Squashing commits in a way that destroys the documented history of a long-running feature.
> - Opening a PR with 20+ unstaged, uncommitted files and a message that says "WIP" or "initial commit."

---

## 6️⃣ Collaboration & Conflict Resolution

_Reference Repos: SentinalMesh, Milan, Omega_

Git is a collaboration tool, not just a backup system. These rules prevent the "merge hell" that kills velocity on multi-contributor projects.

### Pull Request Discipline

> [!IMPORTANT]
> A PR must be **small, focused, and reviewable**. If you cannot describe the change in one sentence, split it into multiple PRs.

| Rule | Rationale |
|------|-----------|
| **One feature per PR** | Mixing auth refactor with CSS tweaks makes review impossible. |
| **PR must pass CI before review** | Never ask a human to review code that fails lint, typecheck, or tests. |
| **Link the issue or ADR** | Use `Closes #42` or `Refs ADR-003` so the PR description traces back to the decision. |
| **Self-review before requesting review** | Re-read your own diff. Fix typos, remove debug logs, and confirm commit messages follow Conventional Commits. |

### Merge Strategy

> [!NOTE]
> **Squash and merge is the default for feature branches.** It keeps `main` linear and readable while preserving the commit messages in the PR description.

| Branch Type | Merge Strategy | Rationale |
|-------------|---------------|-----------|
| `feature/*` | **Squash and merge** | Keeps `main` linear; feature history lives in the PR. |
| `fix/*` | **Squash and merge** | Same as above. |
| `chore/*` | **Merge commit** (no fast-forward) | Dependency bumps often have many small commits; preserve them. |
| `main` / `develop` | **Merge commit** only | Never squash or rebase shared branches. |

> [!WARNING]
> **NEVER rebase a shared branch.** If `main` has moved forward since you branched, merge `main` into your feature branch, then squash-merge your feature branch back to `main`. Reversing this order destroys other engineers' commits.

### Resolving Merge Conflicts

When conflicts occur, follow this decision tree:

```text
Conflict in config/ or infra/?
  → Ask the owner of that file. Do not guess.

Conflict in generated code (migrations, protobuf)?
  → Regenerate the artifact on top of the target branch. Do not manually patch.

Conflict in business logic?
  → The engineer who opened the PR resolves it.
  → If the conflict spans two features, split the PR.
```

> [!TIP]
> **Rebase early, rebase often.** Merge `main` into your feature branch at least once per day. The longer a branch lives, the more painful the conflict resolution becomes.

### Working on Shared Branches

When multiple engineers work on the same long-lived branch (e.g., `develop`):

1. **Pull `develop` before you push.** Run `git pull --rebase origin develop` to keep your commits on top.
2. **Push atomic commits.** Small, logical commits are easier to cherry-pick or revert.
3. **Communicate in the PR.** If two PRs touch the same file, comment on both PRs to coordinate.
4. **Use `git rerere`.** Enable `git config --global rerere.enabled true` to automatically resolve repeated conflicts.

### The "main is always deployable" Rule

> [!IMPORTANT]
> `main` must pass CI and be deployable at all times. If a feature is half-done, keep it on a feature branch. Do not merge WIP code to `main` "to get it tested."

### Rollback Procedure

When a bad merge reaches `main`:

1. **Revert the merge commit, do not reset.** `git revert -m 1 <merge-commit-hash>` creates a new commit that undoes the merge without rewriting history.
2. **Tag the bad deployment.** Create a git tag (`bad-deploy-2026-08-12`) so you can reference the exact code that ran.
3. **Open a postmortem.** Link the postmortem in the revert commit message.

---

## 🔄 Revisit When

When adopting trunk-based development, feature flags, or a different merge tool (e.g., `git-absorb`, `squash-merge` with rebase), revisit this section to confirm the merge strategy and conflict resolution rules still align with the team's workflow.
