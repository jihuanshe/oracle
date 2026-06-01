---
title: Install
description: "Install this Oracle fork from GitHub Releases with mise-managed Node/npm. Node 24+ required."
---

## GitHub Release tarball

This fork publishes prebuilt CLI npm tarballs on GitHub Releases. On macOS,
install Node with mise and install the release tarball with npm:

```bash
curl https://mise.run | sh
mise use -g node@24
mise exec -- npm install -g https://github.com/jihuanshe/oracle/releases/download/v0.13.0/oracle-0.13.0.tgz
```

Requires Node **24 or newer**. After install:

```bash
oracle --help
oracle --version
```

This path installs a prebuilt package artifact. Installing from a Git ref is not
the supported path because npm prepares Git dependencies in a temporary checkout
and can miss the pnpm-built `dist/` output.

## Upstream npm package

The upstream package remains available from npm:

```bash
npm install -g @steipete/oracle
npx -y @steipete/oracle --help
```

Use the GitHub Release tarball above when you want this fork's changes.

## API keys (optional)

API mode is opt-in and reads keys from the environment. Set whichever providers you'll use:

| Provider     | Env var                                                           | Models                                        |
| ------------ | ----------------------------------------------------------------- | --------------------------------------------- |
| OpenAI       | `OPENAI_API_KEY`                                                  | GPT-5.x, GPT-5.x Pro, GPT-5.1 Codex           |
| Azure OpenAI | `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_ENDPOINT`, `..._DEPLOYMENT` | Same models, hosted on Azure                  |
| Google       | `GEMINI_API_KEY`                                                  | Gemini 3.1 Pro (API-only), Gemini 3 Pro       |
| Anthropic    | `ANTHROPIC_API_KEY`                                               | Claude Sonnet 4.6, Claude Opus 4.1            |
| OpenRouter   | `OPENROUTER_API_KEY`                                              | Any OpenRouter id (e.g. `minimax/minimax-m2`) |

If no key is set, Oracle defaults to **browser mode** and drives ChatGPT directly — see [Browser Mode](browser-mode.md) for the manual-login flow.

## Where Oracle stores state

| Path                       | Contents                                                 |
| -------------------------- | -------------------------------------------------------- |
| `~/.oracle/config.json`    | Defaults (JSON5). See [Configuration](configuration.md). |
| `~/.oracle/sessions/<id>/` | Run logs, bundles, transcripts, generated artifacts      |
| `~/.oracle/cookies.json`   | (Optional) inline ChatGPT cookies for browser mode       |

Override the root with `ORACLE_HOME_DIR=/some/path` if you'd rather keep state under XDG config or per-project.

## Updating

```bash
mise exec -- npm install -g https://github.com/jihuanshe/oracle/releases/download/v0.13.0/oracle-0.13.0.tgz
```

`oracle --version` reports the current build. Fork releases land on [GitHub Releases](https://github.com/jihuanshe/oracle/releases).
