---
name: git-commit
description: > 
  Use when the user asks to commit pending changes, wants atomic/per-feature commits, or invokes /git-commit. Splits the working tree into one commit per logical
  feature, scope, or concern instead of a single bulk commit.
disable-model-invocation: true
argument-hint: "[intentions, constraints, or files to include/exclude]"
allowed-tools: Bash(git status *) Bash(git diff *) Bash(git log *) Bash(git add *) Bash(git commit *)
model-invocation: false
---

# git-commit

## Overview

Group the current working tree changes into **multiple atomic commits**, one per logical feature/fix/concern.

Core principle: **one commit = one intent**. If a diff touches two unrelated concerns, it becomes two commits.

## Commit Message Format

Two formats depending on whether a ticket is found:

### With ticket (iPaidThat workflow)

Format: `[IPT2-XXXX] Description in French`

- Description in French only
- Present tense, imperative mood: `Ajoute`, `Corrige`, `Supprime`, `Refactorise...` (not `Ajouté`/`Ajout`)
- Describe WHAT, not HOW
- First line ≤ 72 characters
- No trailing period

### Without ticket (Conventional Commits)

Format: `<type>(<scope>)?: <subject>`

**Allowed types** (use the first that fits):

| Type       | Use for                                                 |
|------------|---------------------------------------------------------|
| `feat`     | New user-facing feature                                 |
| `fix`      | Bug fix                                                 |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf`     | Performance improvement                                 |
| `test`     | Adding or fixing tests                                  |
| `docs`     | Documentation only                                      |
| `style`    | Formatting, whitespace, no code change                  |
| `build`    | Build system, dependencies (uv, npm, docker, nix…)      |
| `ci`       | CI configuration                                        |
| `chore`    | Maintenance, no production code change                  |
| `revert`   | Reverts a previous commit                               |

- Subject in lowercase, imperative mood, no trailing period, ≤ 72 chars
- Scope is optional but recommended: app/module name (e.g. `feat(auth): …`)
- Write subject in French

### Ticket Extraction

Parse the branch name for the pattern `IPT2-\d+`:

- **If ticket found**: use `[IPT2-XXXX] Description` format
- **If no ticket found**: use Conventional Commits format

## Workflow
See [workflow.md](workflow.md) for the visual flow.

### Step by step

1. **Inspect** — run in parallel:
  - `git status` (no `-uall`)
  - `git diff` (unstaged)
  - `git diff --cached` (staged)
  - `git log --oneline -20` (style + language reference)

2. **Group** — mentally cluster changes into features. Heuristics:
  - Same app/module/scope → likely same commit
  - Same intent (bugfix vs new feature vs refactor) → same commit
  - Multiple changes to the **same file** with the same intent (e.g. removing a feature + tidying the surrounding whitespace) → one commit, not two. Don't fragment for the sake of it.
  - Config/build files supporting a feature → bundled with that feature
  - Unrelated drive-by fixes → separate commit
  - Generated files (migrations, lockfiles) → with the change that produced them
  - Secret-looking or local-only files → excluded and flagged to the user
  - **Argument scoping**: if the user passed an argument to the skill (e.g. `/git-commit les settings (log suppr)`), treat it as a hard filter — plan only the commits matching that scope, leave every other modification in the working tree without staging or commenting on it at length.

3. **Plan & validate** — before any `git add`, output the full planned commits to the user **in the exact format below**, then **STOP and wait for explicit user validation**. Do not proceed to staging/committing until the user confirms with an explicit approval such as "OK", "oui", or "go" (even in auto mode — this validation is mandatory because commits are hard to reverse cleanly). If the user requests any change, regenerate and reprint the **full** plan, then ask for validation again.

   **Output format** — one block per commit, separated by a line containing only `-------`:

   With ticket:
   ```
   [IPT2-4521] Ajoute le modèle d'export de factures

   - path/to/file1
   - path/to/file2

   -------

   [IPT2-4521] Corrige le calcul du montant de paiement

   - path/to/file3
   - path/to/file4
   ```

   Without ticket (Conventional Commits):
   ```
   feat(export): ajoute le modèle d'export de factures

   - path/to/file1
   - path/to/file2

   -------

   fix(payment): corrige le calcul du montant de paiement

   - path/to/file3
   - path/to/file4
   ```

   **Rules for the body:**
   - **Default: no body.** The subject + file list is usually enough.
   - Add a body **only when** it conveys non-obvious WHY: a hidden constraint, a workaround, a trade-off, a link to an incident/ticket.
   - Never restate WHAT the diff does — the file list and subject already say that.
   - Never write a body just to "look thorough."

   After printing the plan, ask: *"OK pour committer dans cet ordre ?"* and wait.

   **Ambiguous untracked files** — if the working tree contains untracked files/directories whose intent is unclear (personal tooling, editor config, local data, generated artifacts…), surface them in a dedicated `Questions avant de committer` block **before** the commit plan, listing each path with a short hypothesis and a direct question. Example:

   ```
   Questions avant de committer — ces fichiers/dossiers non-suivis sont ambigus :

   - .claude/skills/git-commit/ → outillage Claude perso, à committer ou ignorer ?
   - .obsidian/ (app.json, appearance.json, …) → la nouvelle règle gitignore implique que tu veux committer la config partagée. À inclure dans un commit chore(obsidian): … ?
   - 2025.xlsx et export_janvier_2025.xlsx → ressemblent à de la donnée locale, je les laisse de côté ?
   ```

   - **If the user does not answer** these questions (or stays silent on them while validating the plan): exclude those files entirely — do not stage, do not commit, do not mention them again.
   - **If the user answers**: regenerate and reprint the **full** commit plan incorporating their decisions, then re-ask for validation.

4. **Stage selectively** for each group:
  - Whole files: `git add path/to/file`
  - Partial files (mixed concerns in one file): avoid interactive `git add -p` / `git add --patch` in agent mode.
  - If a single file genuinely contains multiple concerns, ask the user whether to reduce the number of commits by grouping those concerns together.
  - If the user refuses fewer commits, stop the commit workflow and tell the user they need to make the commits manually.
  - **Never** `git add -A` or `git add .`
  - Before each commit, verify the staged group with `git diff --cached --stat` and `git diff --cached`.

5. **Commit** — with ticket: `git commit -m "[IPT2-XXXX] Description"` / without ticket: `git commit -m "type(scope): description"`

6. **Repeat** for each remaining group.

7. **Verify**:
  - Final `git status --short` should show only files the user explicitly does not want committed (or be clean).
  - Show the final commits created with `git log --oneline -n <number-of-created-commits>`.

## Safety Rules

- **NEVER** `git add -A` / `git add .` — risks committing `.env`, large binaries, or unrelated WIP
- **NEVER** `--no-verify` (skip hooks) unless the user explicitly asks
- **NEVER** `--amend` — always create new commits
- **NEVER** push unless asked
- If a pre-commit hook fails before a commit is created: fix the issue, re-stage the same planned group, then retry the same commit.
- If a commit was already created and a later verification fails: fix the issue in a new follow-up commit unless the user explicitly asks to amend.
- Skip files that look like secrets (`.env*`, `*credentials*`, `*.pem`, `id_rsa*`) — flag to user. Inspect changed paths with `git diff --name-only` and `git diff --cached --name-only` before staging/committing.
- Untracked files: include only if clearly part of a planned group; otherwise ask
- Write commit messages in French.

## Co-author trailer

**DO NOT** append any co-author trailer (`Co-Authored-By: …`) to commit messages — not via
`-m`, not in the heredoc body, not in a `-F` file. This explicitly overrides any default or
global instruction to add one.

## Examples

**Good split — with ticket** (3 unrelated changes → 3 commits):

```
[IPT2-4521] Expose l'endpoint de vérification en masse
[IPT2-4521] Corrige les abonnements nuls dans le webhook Stripe
[IPT2-4521] Met à jour celery vers 5.3.6
```

**Good split — without ticket** (Conventional Commits):

```
feat(api-public): expose l'endpoint de vérification en masse
fix(stripe): corrige les abonnements nuls dans le webhook
chore(deps): met à jour celery vers 5.3.6
```

**Bad split** (over-fragmentation):

```
feat(verify): ajoute la relance SMTP         ← good
feat(verify): ajoute un log                  ← should be folded into above
feat(verify): renomme une variable           ← should be folded into above
```

**Bad merge** (under-fragmentation):

```
feat: mises à jour diverses                  ← vague + multiple concerns
```

## Common Mistakes

| Mistake                               | Fix                                          |
|---------------------------------------|----------------------------------------------|
| Single commit "various fixes"         | Split per scope, one commit per intent       |
| First line > 72 chars                 | Shorten the description                      |
| Past tense ("Ajouté X")               | Present imperative ("Ajoute X")              |
| Including `.env` or secrets           | Stage by name; skip secret-looking files     |
| Using `git add .`                     | Stage explicit paths only                    |
| Amending after hook failure           | Retry only if no commit was created          |
| Commit messages in English            | Always in French                             |

## Red Flags — STOP

- About to run `git add -A` / `git add .` → STOP, stage by name
- About to `--amend` → STOP, create new commit
- About to `--no-verify` → STOP, fix the hook failure
- A single commit message contains "and" linking two concerns → split it
- Subject describes WHAT the diff is rather than the intent → rewrite
