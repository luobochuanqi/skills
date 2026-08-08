# skill-recon

## 0.2.0

### Minor Changes

- Recognize modern skill terrain: the `agents/` subdirectory (sub-agent definitions and cross-runtime `.yaml` config), `scripts/`, `references/`, `dist/`, and packaging files (`LICENSE`, `VERSION`, `package.json`, `CHANGELOG.md`). Add a cross-runtime divergence red flag, sharpen the script-heavy core criterion, exclude packaging noise from the file count, and lead the report with red flags when present.

## 0.1.0

### Minor Changes

- 95e41c1: Initial release of skill-recon: scan any installed skill — inventory its files, annotate each file's purpose, reconstruct its internal logic and design, and surface controllability weaknesses from the red-flag list.
