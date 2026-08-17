# skill-recon

## 0.3.0

### Minor Changes

- Work whole-first instead of file-by-file: survey the skill from its main document and layout, then read closely only the files that earn special attention. Deliver findings as a single Markdown report file — every claim grounded in the skill's own files and cited by path and quote — saved where the repo keeps such notes, or somewhere sensible when there is no convention.

## 0.2.0

### Minor Changes

- Recognize modern skill terrain: the `agents/` subdirectory (sub-agent definitions and cross-runtime `.yaml` config), `scripts/`, `references/`, `dist/`, and packaging files (`LICENSE`, `VERSION`, `package.json`, `CHANGELOG.md`). Add a cross-runtime divergence red flag, sharpen the script-heavy core criterion, exclude packaging noise from the file count, and lead the report with red flags when present.

## 0.1.0

### Minor Changes

- 95e41c1: Initial release of skill-recon: scan any installed skill — inventory its files, annotate each file's purpose, reconstruct its internal logic and design, and surface controllability weaknesses from the red-flag list.
