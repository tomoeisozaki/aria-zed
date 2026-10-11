<p align="center">
  <img src="images/icon.png" alt="ARIA logo: a red removed line above a green added line" width="128" height="128">
</p>

<h1 align="center">ARIA for Zed</h1>

<p align="center">
  <strong>A Review Inline Agent</strong><br>
  Review every edit your terminal AI agent makes, right in Zed, and keep only what you accept.<br>
  Using VS Code? <a href="https://marketplace.visualstudio.com/items?itemName=krispachi.aria-review">ARIA for VS Code</a> (<a href="https://open-vsx.org/extension/krispachi/aria-review">Open VSX</a>).
</p>

---

> [!WARNING]
> **Work in progress, not stable.** This Zed port is under active development: the npm package `aria-review-lsp` is not published yet (use `npm link` in `server/` plus `zed: install dev extension`), and the editor highlight setup still needs manual settings. Expect breaking changes.

## What is ARIA?

Terminal agents (OpenCode, Claude Code, Codex CLI, Cursor CLI, GitHub Copilot CLI, Gemini CLI) write straight to your files and skip any review UI. ARIA notices each edit, marks every changed block in Zed, and lets you **Accept** or **Reject** each block, a whole file, or everything at once. It works without Git.

## How it works in Zed

Zed extensions cannot draw custom UI, so ARIA runs as a **language server** (`aria-review-lsp`, installed by the extension through npm) and uses what Zed already has:

| ARIA concept | In Zed |
|---|---|
| Green/red in the normal editor | Changed lines are **semantic tokens** (`aria-added`, `aria-removed`) that Zed paints with a background color once you add the rules below. Added/changed lines turn green; a one-character red notch marks where lines were removed. Removed rows cannot be drawn in a normal tab, so their text appears as an **inlay hint** (`− removed code`, or `was: old code` after a replaced line; full text in its tooltip). |
| Red/green diff view (optional) | With `"openDiff": true`, an agent edit opens **Zed's diff view** (`zed --diff`): baseline on the left with removed lines in red, the real, editable file on the right with added lines in green. The left side follows every Accept/Reject. |
| Pending change block | A **warning diagnostic** on the changed lines. The message shows the old (`-`) and new (`+`) lines. |
| Accept / Reject | **Code actions** (`cmd-.` / `ctrl-.`): Accept change, Reject change, Accept file, Reject file, Accept all files, Reject all files, Undo last decision. |
| Changes list | **Project Diagnostics** (`cmd-shift-m`) lists every pending block across files, including files that are not open. Code actions work there too. |
| Next / previous change | `editor: go to diagnostic` / `go to previous diagnostic` (`F8` / `shift-F8`). |

1. When the server starts, ARIA takes a **baseline snapshot** of your text files (in memory; nothing is written to the project).
2. A small **user-level hook** in each CLI reports which file the agent just edited (`~/.aria/projects/<hash>.jsonl`). Hooks live in your home folder, never in the project.
3. ARIA diffs the file against its baseline with VS Code's diff algorithm (the `vscode-diff` package) and publishes one diagnostic per change block.
4. **Accept** keeps the new code and makes it the baseline. **Reject** restores the old code: written to disk (Zed reloads the buffer), or applied as an undoable editor edit when the buffer has unsaved changes.

Edits you type yourself outside a pending block go into the baseline and are never shown as agent changes.

## Install

1. In Zed, open **Extensions** (`cmd-shift-x`) and install **ARIA: A Review Inline Agent**.
   For a local checkout, run `zed: install dev extension` and pick this folder (needs Rust installed through `rustup`).
2. Open a project and then any file. The language server starts with the first buffer of a supported language, takes the snapshot and installs the agent hooks.
3. **Restart your CLI agent.** If it asks you to trust the hook, approve it (Codex: run `/hooks`).
4. Ask the agent to edit a file. The changed lines get a warning; press `cmd-.` on them to decide.

## Supported agents

| Agent | Hook location | Status |
|---|---|---|
| OpenCode | global plugin in `~/.config/opencode/plugins/` | ✅ Tested |
| Claude Code | PostToolUse hook in `~/.claude/settings.json` | ✅ Tested |
| Codex CLI | PostToolUse hook in `~/.codex/config.toml` (trust it with `/hooks`) | ✅ Tested |
| Cursor CLI | afterFileEdit hook in `~/.cursor/hooks.json` | ✅ Tested |
| GitHub Copilot CLI | hook file in `~/.copilot/hooks/` | 🧪 Not yet verified |
| Gemini CLI | AfterTool hook in `~/.gemini/settings.json` | 🧪 Not yet verified |

Existing hooks are kept and a backup goes to `~/.aria/backup/` before any change. The hooks are the same ones the VS Code version of ARIA uses, so both can share them.

> Edits from agents that Zed runs in its Agent Panel (ACP, e.g. Claude Agent, which also loads `~/.claude/settings.json` hooks) are skipped by the hook, because Zed reviews them itself. Set `ARIA_RECORD_ALL=1` to record them anyway.

> Zed's own Agent Panel and ACP agents (Claude Code, Codex and Gemini run *inside* Zed) already have a native review. ARIA is for agents running in a terminal.

## Settings

Put them in Zed's `settings.json` (or `.zed/settings.json` for one project) under `lsp.aria.settings`. Every key is optional:

```jsonc
{
  "lsp": {
    "aria": {
      "settings": {
        "enabled": true,                 // false stops review for this project
        "agents": { "claude": true, "codex": true, "cursor": true, "opencode": true, "copilot": true, "gemini": true },
        "adapterSetup": "auto",          // "auto" | "ask" | "off": install missing hooks on start
        "eventsFile": "",                // empty = ~/.aria/projects/<hash>.jsonl
        "excludePatterns": ["**/.env", "**/.env.*", "**/*.pem", "**/*.key"],
        "maxFileSizeKb": 1024,
        "maxSnapshotSizeMb": 64,
        "persistReview": true,           // keep pending review and Undo across restarts
        "retentionDays": 7,
        "undoLimit": 20,
        "openDiff": false                // true: also open Zed's red/green diff view on agent edits
      }
    }
  }
}
```

- **Green/red highlight in the normal editor** needs two settings, because extensions cannot ship colors. In your user `settings.json` (project settings are ignored for `global_lsp_settings`):
  ```jsonc
  "semantic_tokens": "combined",
  "global_lsp_settings": {
    "semantic_token_rules": [
      { "token_type": "aria-added", "background_color": "#2ea04340" },
      { "token_type": "aria-removed", "background_color": "#f85149" }
    ]
  }
  ```
  For the removed-code hints also enable inlay hints, and make them red with a theme override (it recolors every inlay hint, so hide the type/parameter hints of other servers):
  ```jsonc
  "inlay_hints": { "enabled": true, "show_background": true, "show_type_hints": false, "show_parameter_hints": false, "show_other_hints": true },
  "theme_overrides": {
    "<your theme name>": {
      "hint": "<editor text color>", "hint.background": "#f8514955",
      "syntax": { "hint": { "color": "<editor text color>", "background_color": "#f8514955" } }
    }
  }
  ```
  `semantic_tokens` can also be set per project or per language (`"languages": { "TypeScript": { "semantic_tokens": "combined" } }`); `combined` also lets other language servers add their own semantic colors.
- **Turn one agent off** (for example, Claude Code already has its own review): set `"agents": { "claude": false }`. Its hook stays installed, but its edits move the baseline instead of being reviewed.
- **Turn ARIA off for a project**: `"enabled": false` in `.zed/settings.json`, or remove `aria` from `language_servers` for the languages you want.
- **Languages**: the server is attached to most common languages (see `extension.toml`). To add another, use `"languages": { "<Language>": { "language_servers": ["...", "aria"] } }`.
- **Custom server binary**: `"lsp": { "aria": { "binary": { "path": "/path/to/aria-lsp", "arguments": ["--stdio"] } } }`. An `aria-lsp` on your `PATH` (for example from `npm link` in `server/`) is used before the npm install.

## Privacy and local data

- ARIA does not call AI models, upload code or send telemetry.
- Hook events contain only file paths, timestamps and agent IDs (`~/.aria/projects/`).
- Pending snapshots, Undo history and the diff view's baseline copies are stored as **local plaintext** in `~/.aria/sessions/<hash>/` (outside the project). Set `persistReview: false` for memory-only review, and use `excludePatterns` for sensitive files.
- Zed has no uninstall hook. To remove the hooks: `npx aria-review-lsp --remove-adapters`. To remove hooks, event files and runtime: `npx aria-review-lsp --uninstall`.

## Limitations

- In the normal editor tab removed lines are not red rows: they appear as inlay hints (red background with the theme override above; inlay text cannot be syntax-highlighted, the hint tooltip shows the removed code highlighted), in the hover and in the optional diff view. There are no clickable buttons above blocks; use `cmd-.`.
- The diff opens in the most recently focused Zed window (the `zed` CLI cannot target a workspace). If the `zed` command is not on the server's PATH, set `ZED_CLI` via `lsp.aria.binary.env`.
- The server starts only after a buffer of an attached language opens in the project.
- Edits made by shell commands without a reported path are not attributed to an agent. Deleted files are not reviewed.
- Files over `maxFileSizeKb`, binary files and files over the snapshot budget are skipped. The scan stops at 20,000 files.

## Development

```bash
cd server && npm ci && npm test     # compile + diff, hook runtime and LSP end-to-end tests
cargo build --release --target wasm32-wasip2   # check the Zed extension builds
```

The repository layout: `extension.toml` + `src/lib.rs` (Zed extension, Rust → WASM) start `server/` (Node language server). `server/adapters/` contains the hook runtimes copied to `~/.aria/adapters/`. See [PUBLISHING.md](PUBLISHING.md) for releases.

## License

[MIT](LICENSE) © Krispachi
