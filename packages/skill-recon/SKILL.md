---
name: skill-recon
description: Scan any installed skill — inventory its files, reconstruct its logic and design, and surface controllability red flags an unfamiliar skill may carry.
disable-model-invocation: true
---

# Skill Recon

Map a target skill's local files into a complete picture: what it is, how it works, why it is designed this way — and what **red flags** it carries.

## Locate the target

The target comes from the user's invocation: a skill name (resolve it in common skill directories such as `~/.agents/skills`) or a path. Use an explicit target from context; otherwise ask the user.

## Steps

### 1. Inventory the terrain
List every file path under the target directory, including subdirectories. Exclude version-control, dependency, and packaging noise (the full list is in Reading strategy) — note excluded files' presence but do not treat them as skill logic.

Completion criterion: the list covers every content file, none omitted.

### 2. Annotate the legend
For each file in the list, find out: what it is, why it exists, what references it, what it references. Recognize the common subdirectory roles (see Reading strategy) before reading — they tell you what kind of thing a file is before you open it. Read as much as the reading strategy below dictates.

Completion criterion: no blank spots on the map — every file has a clear annotation; a file you cannot annotate gets read further until you can.

### 3. Draw the full picture
Synthesize the files' purposes into three things:
- **What problem it solves**: target scenarios, when it is invoked
- **How it works**: flow, steps, branches
- **Why it is designed this way**: division of labor, organizational intent

Completion criterion: all three are clear, and "why" is not a restatement of "what".

### 4. Hunt for red flags
Check the target skill against the red-flag list below, actively looking for weak controllability. Every reported red flag needs evidence: quote the triggering text or file.

Completion criterion: every item on the list has been checked, every hit is reported with evidence; unconfirmed suspicions are reported too, marked as such.

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

## Reading strategy

**Threshold: 20 content files** — count only files the skill logic touches. Exclude packaging noise from the count: `LICENSE`/`LICENSE.txt`, `VERSION`, `package.json`, `CHANGELOG.md`, `dist/` (build output), and any lockfile.

**Small skill (at or below the threshold)**: read every content file in full.

**Large skill (above the threshold)**:
- The main document (`SKILL.md`, else `README.md`) **must be read in full** — it is the map's skeleton
- Companion files the main document references (e.g. `GLOSSARY.md`, templates, references) are read in full
- Other files: read only the beginning (~first 20–30 lines or first two sections) to capture their purpose
- A file whose purpose is not clear from the beginning gets read in full

### What the common subdirectories hold

Modern skills carry subdirectories with fixed roles — recognize them rather than treat each file cold:

- **`agents/`** — secondary agent definitions. `.md` files are sub-agent prompt bodies (read enough to know each agent's role); `.yaml` files are cross-runtime interface config (display name, default prompt). A `.yaml` here means the skill is shaped for more than one runtime — its behavior may not be fully captured by SKILL.md alone.
- **`scripts/`** — executable helpers. Read each and classify it (mechanical util vs decision logic) under Script-heavy core.
- **`references/`** — reference material split out of SKILL.md (disclosed reference). Read enough of each to know what it covers.
- **`dist/`** — build output, not source. Note its presence, do not read it as a first-class file; the source it was built from is what matters.

## Output

Deliver a concise scan report to the user. If red flags are present, open with them — the user's first question is whether the skill is safe to trust:

- **File map**: directory tree + one-line purpose per file
- **Understanding**: what the skill is, how it works, why it is designed this way (the three things established in step 3)
- **Red flags**: list of hits, each with evidence (the yield of step 4)
