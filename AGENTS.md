# AGENTS.md

Oracle is a Node/pnpm CLI used by coding agents and humans to bundle prompts,
files, and browser/API context for stronger model review. Keep changes small,
verified, and compatible with packaged installs.

## Development environment

- Use the repo-local mise config in `.mise/config.toml` for Node, pnpm, prek,
  and task entrypoints. Do not rely on globally installed Node or pnpm.
- After checkout, run `mise install`, then `mise run install`.
- Prefer mise tasks over raw package-manager commands:
  - `mise run check` — formatting, typecheck, and lint.
  - `mise run test` — Vitest suite.
  - `mise run build` — compile `dist/` and copy vendored assets.
  - `mise run docs-check` — verify docs against CLI help.
  - `mise run test-packed-cli` — pack/install smoke for the published shape.
  - `mise run ci` — local default CI bundle.
- Project dependencies still live in pnpm. Mise selects tool versions and wraps
  common commands; pnpm owns `node_modules` and `pnpm-lock.yaml`.
- Git hooks are managed by `prek.toml`. Install them with
  `mise exec -- prek install`. Hook commands must call tools through
  `mise exec -- ...` when versions matter.
- `prek.toml` is intentionally excluded from oxfmt because oxfmt currently
  rewrites it into invalid TOML. Validate it with
  `mise exec -- prek validate-config prek.toml`.

## Build and package invariants

- Do not commit `dist/`, package tarballs, or notifier build products.
- The package install surface is the npm tarball shape, not a git checkout.
  Keep `package.json` `files` and `bin` entries aligned with `pnpm pack`.
- Use `mise run test-packed-cli` after changes that affect packaging, CLI
  startup, build scripts, package metadata, vendored files, or release workflow.
- `build:vendor` should stay a real script (`scripts/build-vendor.js`), not an
  inline one-liner in `package.json`.
- pnpm v10 settings belong in `pnpm-workspace.yaml`, not `package.json#pnpm`.

## Release workflow

- GitHub release artifacts are built by `.github/workflows/release-package.yml`.
  The workflow builds from a `vX.Y.Z` tag or manual dispatch tag, creates
  `oracle-X.Y.Z.tgz`, and uploads SHA1/SHA256 checksum files.
- The package job runs on the private `hetzner-hel1-ex63` runner. Keep release
  commands routed through mise tasks so local and CI toolchains match.
- The macOS notifier app is built in a macOS job, signed/notarized, archived as
  an artifact, restored on the private package runner, and included before
  `pnpm pack`.
- Release notifier builds must fail if signing or notarization secrets are
  missing. Required GitHub Actions secrets:
  - `MACOS_CODESIGN_CERTIFICATE_BASE64`
  - `MACOS_CODESIGN_CERTIFICATE_PASSWORD`
  - `CODESIGN_ID`
  - `APP_STORE_CONNECT_API_KEY_P8`
  - `APP_STORE_CONNECT_KEY_ID`
  - `APP_STORE_CONNECT_ISSUER_ID`
- Stable releases dispatch the Homebrew tap update workflow after GitHub Release
  assets are uploaded. Prerelease tags such as `v1.2.3-beta.1` must be marked as
  GitHub prereleases and must not update Homebrew stable.
- npm publish still requires OTP. Prepare/tag/release first, then stop at
  `Enter OTP:` and ask the user for the code.
- Beta npm publishing requires a new beta version (for example,
  `0.4.4-beta.1`); npm will not let you overwrite an existing beta tag.
- Sparkle signing key lives at
  `/Users/steipete/Library/CloudStorage/Dropbox/Backup/Sparkle`; set
  `SPARKLE_PRIVATE_KEY_FILE` to that path when notarizing the notifier outside
  GitHub Actions.

## Verification expectations

- For normal code changes, run the narrowest relevant mise task.
- For workflow/tooling/package changes, run at least:
  - `mise run format-check`
  - `mise exec -- prek validate-config prek.toml` when hook config changed.
  - `mise run docs-check` when CLI help or docs changed.
  - `mise run build`
  - `mise run test-packed-cli`
- For hook changes, also run:
  - `mise exec -- prek run --all-files --hook-stage pre-commit`
  - `mise exec -- prek run --all-files --hook-stage pre-push`
- Before a release, skim `docs/manual-tests.md` and rerun any manual smoke that
  covers the touched surface, especially browser and `oracle serve` paths.

## Browser and live-test rules

- Pro browser runs can take 10 minutes or longer. Never click or auto-click
  ChatGPT's `Answer now` button; it skips long thinking and changes behavior.
- Keep at least 1–2 Pro live tests truly Pro when the change touches Pro browser
  selection/reattach behavior. Move unrelated smokes to faster models when safe.
- OpenAI live tests are opt-in. Use
  `mise exec -- env ORACLE_LIVE_TEST=1 pnpm vitest run tests/live/openai-live.test.ts`
  only when a real `OPENAI_API_KEY` is available and the background path matters.
- GPT-5 Pro API runs detach by default; pass `--wait` to stay attached. GPT-5.1
  and browser runs block by default. Every run prints `oracle session <id>` for
  reattach.
- Browser smokes should preserve Markdown lists and fences. If output looks
  flattened or echoed, inspect the captured assistant turn before shipping.
- If browser smokes echo the prompt, rerun with keep-browser plus verbose mode,
  then inspect DOM state with `scripts/browser-tools.ts eval`.

## Browser-mode debug notes

- Session data lives under `~/.oracle`; delete that directory for a clean slate.
- When a ChatGPT folder/workspace URL is set, Cloudflare can block automation
  even after cookie sync. Use `--browser-keep-browser`, solve the interstitial
  manually, then rerun.
- If a run looks finished but the CLI did not stream output, check
  `oracle status`, then inspect with `oracle session <id> --render`.
- Active Chrome port and pid live in
  `~/.oracle/sessions/<id>/meta.json`. Use the saved port with
  `scripts/browser-tools.ts eval --port <port> ...`.
- To debug with agent-tools, launch Chrome via an Oracle browser run so cookies
  are copied, keep it open, then connect to the port in session metadata. Avoid
  starting a fresh browser-tools Chrome when synced cookies are required.
- Double-hop navigation is implemented (root, then target URL), but Cloudflare
  may still require manual clearance or inline cookies.

## Product and compatibility notes

- The first line of any top-level CLI start banner should use the oracle emoji,
  for example `🧿 oracle (<version>) ...`. Keep it only for the initial command
  headline. The TUI exit message is the exception and also keeps the emoji.
- Working on Windows requires reading and updating `docs/windows-work.md` before
  starting Windows-specific work.
- Model access note (2025-11-23): `gpt-5.1-pro` and `grok-4.1` are not yet
  available on Peter's keys; live tests requiring them will fail until access is
  granted.
- On Node 25, if `pnpm dlx @steipete/oracle --help` fails with a missing
  `node_sqlite3.node`, rebuild sqlite3 in the pnpm dlx cache using system
  Python from the sqlite3 package directory printed in the error:
  `PYTHON=/usr/bin/python3 /Users/steipete/Projects/oracle/runner npx node-gyp rebuild`.
- If browser cookie sync on Node 25 fails with a missing `keytar.node`, rebuild
  keytar in the pnpm dlx cache using the same `node-gyp` pattern.
- ChatGPT project URLs:
  - `steipete@gmail.com`:
    `https://chatgpt.com/g/g-p-691edc9fec088191b553a35093da1ea8-oracle/project`
  - `studpete@gmail.com`:
    `https://chatgpt.com/g/g-p-69505ed97e3081918a275477a647a682/project`
  - Prefer the `studpete` URL if the `steipete` project is not found.

## Changelog discipline

- After finishing a feature, ask whether it matters to end users. If yes,
  update `CHANGELOG.md`.
- Read the top ~100 lines first and group related edits into one entry instead
  of scattering multiple bullets.
