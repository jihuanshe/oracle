# Release Checklist (GitHub Release tarball)

This fork publishes a prebuilt CLI npm tarball through GitHub Releases. The
release path intentionally does not use npm publish, Homebrew, private runners,
or Apple signing/notarization secrets.

The `Release package` GitHub Action builds from a `vX.Y.Z` tag or manual
dispatch tag, uploads `oracle-X.Y.Z.tgz` plus SHA1/SHA256 checksums to the
GitHub Release, and keeps `dist/` out of git. Development and CI tool versions
are pinned in `.mise/config.toml`; run `mise install` before local release
checks.

1. **Version & metadata**
   - [ ] Update `package.json` version (e.g., `1.0.0`).
   - [ ] Update any mirrored version strings (CLI banner/help, docs metadata) to match.
   - [ ] Confirm package metadata (name, description, repository, keywords, license, `files`/`.npmignore`).
   - [ ] If dependencies changed, run `mise run install` so `pnpm-lock.yaml` is current.
2. **Artifacts**
   - [ ] Run `mise run build` (ensure `dist/` is current).
   - [ ] Verify `bin` mapping in `package.json` points to `dist/bin/oracle-cli.js`.

- [ ] Produce npm tarball and checksums locally only when you need to inspect them before tagging:
  - `mise exec -- pnpm pack --pack-destination /tmp` (after build)
  - Move the tarball into repo root (e.g., `oracle-<version>.tgz`) and generate `*.sha1` / `*.sha256`.
  - Keep these files for local verification only; do **not** commit them. The GitHub Action uploads the release copies after the tag is pushed.

3. **Changelog & docs**

- [ ] Update `CHANGELOG.md` (or release notes) with highlights.
- [ ] Keep changelog entries product-facing only; avoid adding release-status/meta lines (e.g., “Published to npm …”)—that belongs in the GitHub release body.
- [ ] Verify changelog structure: versions strictly descending, no duplicates or skipped numbers, single heading per version.
- [ ] Ensure README reflects current CLI options (globs, `--status`, heartbeat behavior).
- [ ] **Release notes must exactly match the version’s changelog section** (full Added/Changed/Fixed/Tests bullets, no omissions). After creating the GitHub release, compare the body to `CHANGELOG.md` and fix any mismatch.

4. **Validation**
   - [ ] `mise run check` (zero warnings allowed; fail on any lint/type warnings).
   - [ ] `mise run test`
   - [ ] `mise run build`
   - [ ] `mise run test-packed-cli`
   - [ ] Optional live smoke (with real `OPENAI_API_KEY`): `mise exec -- env ORACLE_LIVE_TEST=1 pnpm vitest run tests/live/openai-live.test.ts`
   - [ ] MCP sanity check: with `config/mcporter.json` pointed at the local stdio server (`oracle-local`), run `mcporter list oracle-local --schema --config config/mcporter.json` after building (`mise run build`) to ensure tools/resources are discoverable.
5. **Publish GitHub Release**
   - [ ] Ensure git status is clean; commit and push any pending changes.
   - [ ] Create and push the tag: `git tag vX.Y.Z && git push origin vX.Y.Z`.
   - [ ] Or rerun the workflow manually for an existing tag:
         `gh workflow run release-package.yml --ref main -f tag=vX.Y.Z`.

6. **Post-publish**

- [ ] Verify GitHub release exists for `vX.Y.Z` and has `oracle-X.Y.Z.tgz`,
      `oracle-X.Y.Z.tgz.sha1`, and `oracle-X.Y.Z.tgz.sha256`.
- [ ] Confirm the GitHub release body matches the intended release notes.
- [ ] Verify install from the release tarball on macOS:
      `mise exec -- npm install -g https://github.com/jihuanshe/oracle/releases/download/vX.Y.Z/oracle-X.Y.Z.tgz`.
- [ ] Verify the installed CLI: `oracle --version`.
- [ ] After local artifact inspection, remove any untracked tarball/checksum
      files from the repo root.
- [ ] Announce / share release notes.
