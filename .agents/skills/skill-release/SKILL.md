---
name: skill-release
description: Standard end-to-end version-control flow for this skills repo. Use when creating a new skill package, updating an existing skill, writing a changeset, bumping versions, or releasing/tagging a skill here.
metadata:
  internal: true
---

# Skill Release

The end-to-end flow for shipping skills in this repo: create a new skill package or update an existing one, version it with changesets, and let CI release it.

## Repo facts

- Every skill is a private package under `packages/<name>/`, managed by the pnpm workspace at the repo root.
- Versions are managed by changesets. Pushing to `main` runs the Release workflow: open changesets accumulate into a version PR ("chore: version skills"); merging it triggers tagging.
- Tags use the form `name@version` (e.g. `skill-recon@0.1.0`), and private packages ARE tagged — `.changeset/config.json` sets `privatePackages.tag: true`.
- Skill text must be written in English (repo rule in `AGENTS.md`).
- The `protect` ruleset on `main` requires linear history and signatures: merge version PRs with a **squash** merge (web UI handles this). As repo owner you can bypass the ruleset entirely and push to `main` directly.
- CI tags `name@version` for every package whose version has no tag yet — it never inspects commit messages or PRs, which is what makes the direct-push path work.

## Create a new skill

1. **Scaffold the package** at `packages/<name>/`:
   - `package.json` — `name: <name>`, `version: 0.0.0`, `private: true`, one-line `description`
   - `SKILL.md` — frontmatter (`name`, `description`, optional `disable-model-invocation`) plus the body, in English

   Completion: both files exist and match the conventions of existing skills in `packages/`.

2. **Register in the workspace**: run `pnpm install` so the lockfile records the new package.

   Completion: `pnpm list --depth -1` shows the new package.

3. **Add a changeset** — run `pnpm changeset` (interactive) or hand-write `.changeset/<slug>.md`. A first release is `minor` (0.0.0 → 0.1.0); the frontmatter must use the package's exact `name`.

   Completion: a changeset file exists referencing the package.

4. **Ship it** — two ways, same result (remote tag `name@version`):
   - **Direct push (owner)**: you can bypass the `protect` ruleset on `main`, so skip the PR entirely. Run `pnpm changeset version` locally (bumps versions, updates CHANGELOG, consumes changesets), commit (e.g. `chore: version skills`), and push. CI sees no changesets and tags automatically — tagging only checks whether the `name@version` tag exists, never commit messages or PRs.
   - **Version PR**: push the changeset itself; CI opens the "chore: version skills" PR; squash-merge it (the ruleset requires linear history); CI tags on merge.

   Completion: `git ls-remote --tags origin` shows `name@version`.

## Update an existing skill

1. **Edit the skill** — SKILL.md or its companion files.
2. **Add a changeset**: `patch` for fixes, `minor` for new behavior, `major` for breaking changes.
3. **Ship it** — same two paths as step 4 of Create: direct push (owner) or version PR. Version PRs are cumulative (changesets from several skills pile into the same PR until you merge it), and a direct `pnpm changeset version` bumps every pending changeset at once — ship when you want, not per skill.

## Fallbacks

- **CI did not open the version PR** (Actions not permitted to create PRs): the workflow still pushes the `changeset-release/main` branch. Create it manually:
  `gh pr create --base main --head changeset-release/main --title "chore: version skills"`.
- **CI did not tag after merge**: confirm `.changeset/config.json` has `privatePackages.tag: true`, then verify locally with `pnpm changeset tag` (creates a local tag only, never pushes).

## Changeset format

```markdown
---
"<package-name>": patch | minor | major
---

Description of the change — this lands in the package's CHANGELOG.md.
```
