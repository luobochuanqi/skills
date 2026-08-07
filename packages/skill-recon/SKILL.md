---
name: skill-recon
description: Scan any installed skill — inventory its files, annotate each file's purpose, reconstruct its internal logic and design, and surface controllability weaknesses.
disable-model-invocation: true
---

# Skill Recon

Map a target skill's local files into a complete picture: what it is, how it works, why it is designed this way — and what **red flags** it carries.

## Locate the target

The target comes from the user's invocation: a skill name (resolve it in common skill directories such as `~/.agents/skills`) or a path. Use an explicit target from context; otherwise ask the user.

## Steps

### 1. Inventory the terrain
List every file path under the target directory, including subdirectories, excluding version-control and dependency noise (`.git`, `node_modules`, etc.).

Completion criterion: the list covers every content file, none omitted.

### 2. Annotate the legend
For each file in the list, find out: what it is, why it exists, what references it, what it references. Read as much as the reading strategy below dictates.

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
- **Script-heavy core**: core functionality implemented by running scripts — many scripts, heavy script references in SKILL.md, scripts carrying decision logic. A few mechanical util scripts are fine; main logic in scripts is a black box neither the agent nor the user can audit.
- **Undisclosed destructive operations**: deletions, overwrites, network requests, environment modifications without explicit disclosure or a confirmation gate in SKILL.md.
- **Sprawl**: SKILL.md oversized or poorly structured, key instructions drowned in text. Size alone is not a problem; low information density and hard-to-navigate instructions are.
- **No-op prose**: many "be careful / be thorough / ensure quality" instructions — they do not change default behavior, just pay tokens to say nothing.
- **Negation steering**: heavy "don't X / never Y" phrasing — it drags the forbidden behavior into context and makes it more likely, not less.
- **Duplication and contradiction**: the same instruction in multiple places; worse, instructions that contradict each other, leaving the agent no way to comply.
- **Sediment**: references to nonexistent files, mention of deprecated flows, leftovers from old versions.
- **Description mismatch**: the description does not match SKILL.md's actual content (claimed functionality differs from the body), or the description is overlong.

## Reading strategy

**≤ 20 files**: read everything.

**> 20 files**:
- The main document (`SKILL.md`, else `README.md`) **must be read in full** — it is the map's skeleton
- Companion files the main document references (e.g. `GLOSSARY.md`, templates, scripts, references) are read in full
- Other files: read only the beginning (~first 20–30 lines or first two sections) to capture their purpose
- A file whose purpose is not clear from the beginning gets read in full

## Output

Deliver a concise scan report to the user:

- **File map**: directory tree + one-line purpose per file
- **Understanding**: what the skill is, how it works, why it is designed this way (the three things established in step 3)
- **Red flags**: list of hits, each with evidence (the yield of step 4)
