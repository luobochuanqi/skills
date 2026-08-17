---
name: skill-recon
description: Scan any installed skill — build the whole picture of its logic and design, mark the files that deserve a close read, and surface controllability red flags an unfamiliar skill may carry.
disable-model-invocation: true
---

# Skill Recon

Map a target skill into a complete picture: what it is, how it works, why it is designed this way — and what **red flags** it carries.

Work **whole-first**: build the picture from the skill's main document and layout before going deep anywhere, then read closely only the files that earn it. Reading every content file cover-to-cover is not the job — most files' purposes are evident from their role alone.

## Locate the target

The target comes from the user's invocation: a skill name (resolve it in common skill directories such as `~/.agents/skills`) or a path. Use an explicit target from context; otherwise ask the user.

## Steps

### 1. Survey the whole

Read the main document (`SKILL.md`, else `README.md`) **in full** — it is the skeleton of the skill. List the target directory so the layout is in front of you; the subdirectory roles below tell you what kind of thing a file is before you open it. Packaging noise (below) is not skill logic — note its presence, do not read it.

From these two inputs, form the whole picture:

- **What problem it solves** — target scenarios, when it is invoked
- **How it works** — flow, steps, branches
- **Why it is designed this way** — division of labor, organizational intent

Completion criterion: you can state all three, and the "why" is not a restatement of the "what".

### 2. Mark the files that matter

Decide which files deserve a close read — the **special-attention set**. A file earns it when:

- The main document references it — read it in full
- Its role is not evident from its name and location — read until it is
- It may carry behavior the main document does not — `scripts/` entries, `agents/*.yaml`, any file flagged by its subdirectory role
- A red-flag indicator applies — anything acting outside the skill's own directory (writes, network requests, deletions, environment changes)

Everything else needs only a one-line purpose from its evident role. Read the special-attention set in full; annotate each file: what it is, why it exists, what it references, what references it. Let the annotations refine the step-1 picture.

Completion criterion: every content file is either annotated from a full read or one-lined from an evident role — no file's purpose is a guess.

### 3. Hunt for red flags

Check the target against the red-flag list below, actively looking for weak controllability. Every reported red flag needs evidence: quote the triggering text and its file. Unconfirmed suspicions are reported too, marked as such.

Completion criterion: every item on the list has been checked, and every hit is reported with evidence.

### 4. Write the report

Write the findings to a single Markdown file:

- **Ground every claim in primary evidence** — the skill's own files, never a second-hand impression. Follow each claim back to the file that owns it and cite it: path plus quoted text.
- **Structure** — red flags first when present (the user's first question is whether the skill is safe to trust), then the file map (directory tree + one-line purpose per file), then the understanding (what / how / why).
- **Save it where the repo already keeps such notes** — match the existing convention; if there is none, put it somewhere sensible and say where in your reply.

Completion criterion: the report exists as one Markdown file at a stated path, every claim carries its source, and every red-flag hit appears with evidence.

## Red-flag list

- **Implicit side effects**: file operations outside the working directory (writing to `~/.agents`, `~/.claude`, system directories, or other project paths). Operations inside the working directory are visible; external paths are implicit. Clearly disclosed, justified external writes do not count; hidden instructions the user never sees do.
- **Script-heavy core**: core functionality implemented by running scripts — many scripts, heavy script references in SKILL.md, scripts carrying decision logic. A few mechanical util scripts (formatting, counting, scaffolding) are fine; the flag is for logic the agent cannot follow without executing the script. Audit each `scripts/` entry: read its source, classify as mechanical util or decision logic, and flag when core behavior lives in code rather than SKILL.md.
- **Undisclosed destructive operations**: deletions, overwrites, network requests, environment modifications without explicit disclosure or a confirmation gate in SKILL.md.
- **Cross-runtime divergence**: an `agents/*.yaml` file reshapes behavior for a second runtime (e.g. OpenAI) — different default prompts, display names, or instruction subsets. Read the `.yaml` against SKILL.md and flag any case where one runtime gets instructions or capabilities the other does not; behavior that is not uniform across runtimes is a controllability gap, and silent divergence is worse than disclosed.
- **Sprawl**: SKILL.md oversized or poorly structured, key instructions drowned in text. Size alone is not a problem; low information density and hard-to-navigate instructions are.
- **No-op prose**: many "be careful / be thorough / ensure quality" instructions — they do not change default behavior, just pay tokens to say nothing.
- **Negation steering**: heavy "don't X / never Y" phrasing — it drags the forbidden behavior into context and makes it more likely, not less.
- **Duplication and contradiction**: the same instruction in multiple places; worse, instructions that contradict each other, leaving the agent no way to comply.
- **Sediment**: references to nonexistent files, mention of deprecated flows, leftovers from old versions.
- **Description mismatch**: the description does not match SKILL.md's actual content (claimed functionality differs from the body), or the description is overlong.

## What the common subdirectories hold

Modern skills carry subdirectories with fixed roles — recognize them rather than treat each file cold:

- **`agents/`** — secondary agent definitions. `.md` files are sub-agent prompt bodies (read enough to know each agent's role); `.yaml` files are cross-runtime interface config (display name, default prompt). A `.yaml` here means the skill is shaped for more than one runtime — its behavior may not be fully captured by SKILL.md alone.
- **`scripts/`** — executable helpers. Read each and classify it (mechanical util vs decision logic) under Script-heavy core.
- **`references/`** — reference material split out of SKILL.md (disclosed reference). Read enough of each to know what it covers.
- **`dist/`** — build output, not source. Note its presence, do not read it as a first-class file; the source it was built from is what matters.

## What is not skill logic

Packaging, version-control, and dependency noise — `LICENSE`/`LICENSE.txt`, `VERSION`, `package.json`, `CHANGELOG.md`, `dist/`, lockfiles — carries no skill logic. Note its presence; do not read it as first-class material.
