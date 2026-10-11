# Changelog

## 0.18.0 - unreleased (work in progress, not stable)

- Port ARIA to Zed. The VS Code extension code is replaced by a Zed extension (`extension.toml`, `src/lib.rs`) that starts a Node language server (`server/`, npm package `aria-review-lsp`).
- Green/red highlight in the normal editor through semantic tokens (`aria-added`, `aria-removed`) colored by `global_lsp_settings.semantic_token_rules`.
- Removed code is shown as inlay hints (`− …` / `was: …`) in the normal editor; deletions are marked with a one-character red notch.
- The hook skips edits from agents Zed runs in its Agent Panel (ACP), which Zed already reviews.
- Agent edits open Zed's diff view (`zed --diff`) with the baseline on the left, so changes show in red/green; setting `openDiff` (off by default; opens in the existing window).
- Pending change blocks are diagnostics with the old/new lines in the message; Accept/Reject change, file and all files, and Undo are code actions. Project Diagnostics serves as the changes list.
- Reused unchanged: diff algorithm, hook runtimes and installers for all six agents, event store, saved review, exclude globs.
- Settings move to `lsp.aria.settings` in Zed's settings.json. VS Code-only features (status bar and toolbar buttons, Changes view, inline diff tabs, settings export/import, Init Directory) are dropped.
- Cleanup is a CLI: `npx aria-review-lsp --remove-adapters` or `--uninstall`.

## Unreleased (VS Code version)

- README: add live badges for VS Marketplace version and installs, Open VSX version and downloads, and the latest GitHub release.

## 0.17.3 - 2026-10-11

- README: replace the Mermaid flowchart with a PNG diagram so "How it works" also appears on the Marketplace, Open VSX and in VS Code's extension page (they do not render Mermaid).

## 0.17.2 - 2026-10-11

- Change the extension ID to krispachi.aria-review (the earlier aria-inline-review listing was removed and its name cannot be reused). Settings (aria.*), commands and hooks are unchanged; uninstall krispachi.aria-inline-review before installing.
- Rename the display name to "ARIA: A Review Inline Agent" (the previous display name stays reserved by the removed listing).
- Reinstall missing agent hooks automatically on the next start when an earlier automatic setup had succeeded (for example after an older ARIA build's uninstall removed them). Only failed automatic setups are not retried.
- README: hooks are installed automatically; add a short troubleshooting step for missing hooks.

## 0.17.1 - 2026-10-11

- Change the extension ID to krispachi.aria-inline-review because the name "aria" is already taken on the Marketplace. Settings (aria.*), commands and hooks are unchanged; uninstall a previously sideloaded krispachi.aria before installing.
- Group settings into titled categories in the Settings editor: General, Agents and Hooks, Buttons and Display, Review Flow, Saved Review and Undo, Performance and Privacy, Deprecated.
- Rewrite the README: logo, what the name ARIA (A Review Inline Agent) means, how it works, supported agents, controls, commands and settings by category.
- Link the GitHub repository (krispachi/aria): repository, homepage and issue links in package.json; README images and links are rewritten to GitHub URLs when packaging.
- Move README screenshots to images/readme/ and add CONTRIBUTING, SECURITY, issue and pull request templates for the GitHub repository.

## 0.17.0 - 2026-10-11

- Edits from a disabled agent (or untagged legacy hooks while any agent is disabled) now move that file's baseline when it is not under review, so a later edit by an enabled agent no longer shows them as its own changes.
- Document supported agents and their verification status: OpenCode, Claude Code, Codex CLI and Cursor CLI verified in real sessions; GitHub Copilot CLI and Gemini CLI adapters included but not yet verified. Update marketplace description and keywords.
- Document that changes made while ARIA is disabled are never recorded.
- Never wait on notifications inside event ingestion or Undo: an unread "unsaved manual edits" or "cannot undo" toast no longer stops all later agent edits from being reviewed or blocks every Accept/Reject.
- Record event offsets before the baseline snapshot so an agent edit made while a large workspace is being scanned is still reviewed.
- Wait for a pending local save before restoring a saved review, so a quick stop/start cannot bring back an already accepted change.
- Upgrade an existing untagged Codex hook block instead of reporting "Already installed"; untagged events were ignored once any agent was disabled. Codex [hooks] conflict message is now in English with the exact command to add.
- Do not retry a failed automatic adapter setup on every settings or folder change for the same version and agent selection (it rewrote CLI configs and repeated the warning). Install Agent Adapters still retries manually.
- Stop applying agent buffer edits when the session is stopped mid-review, and clear stale review state for oversized files regardless of drive-letter case.

## 0.16.2 - 2026-10-11

- Raise default diff colors slightly: line backgrounds from about 7% to 10% opacity, text highlights from 14% to 18%.

## 0.16.1 - 2026-10-11

- Fix Accept/Reject File and Accept/Reject All buttons never appearing: VS Code reports an implicit false for the deprecated showFileActions/showAllActions toggles, which was read as "hidden". Only explicit user values now count.
- Add `aria.blockActionsAnchor` (beforeGroup, firstChangedRow) and a Quick Settings toggle. The default puts per-group buttons above the whole group again; firstChangedRow keeps them between red and green rows.

## 0.16.0 - 2026-10-11

- Add `aria.fileActionsLocation` and `aria.allActionsLocation` (statusBar, editorToolbar, both, hidden). Decision buttons now default to labelled status bar buttons; choose quickly with ARIA Review: Choose Button Location or Quick Settings.
- Remove ARIA's optional editor-toolbar Previous/Next; status bar arrows, commands and VS Code's native diff arrows remain.
- Deprecate `showFileActions`, `showAllActions` and `showNavigationToolbar`. Older settings exports still import; an explicit false on the old toggles keeps hiding the buttons.

## 0.15.1 - 2026-10-11

- Compute review groups with VS Code's own diff algorithm (vscode-diff, same options as the diff editor) instead of jsdiff, so Accept/Reject rows match the red/green rows shown.
- Place Accept/Reject directly above each group's first changed row instead of one context row earlier; first-row replacements now get inline actions too.
- Replace a resolved file's snapshot diff tab with a normal editor (keeping selection and scroll), so line numbers and the diff toolbar return to normal after Accept All or session stop.
- Keep the cursor on the focused change after an agent edit opens the review (a whole-buffer refresh used to move it to the end of the file).

## 0.15.0 - 2026-10-11

- Add a profile-wide Enable/Disable switch with command-palette and Quick Settings controls while preserving directory Init and Enable/Disable preferences.
- Add per-agent switches and Choose Agents for OpenCode, Claude Code, Codex, Cursor, Copilot and Gemini. Review only enabled tagged sources and install only selected adapters; keep existing hooks and pending reviews when disabling a source.
- Tag hook events with their producer ID without changing the project hash. Ignore ambiguous legacy events when any agent is disabled, and suspend Track All to prevent bypassing source filters.
- Include master/agent/setup choices in validated settings export/import. Defer setup until settings selection/import completes and avoid extra snapshots for agent-only changes.
- Preserve unrelated commands in mixed hook groups during install/removal, including user hooks in Copilot files.
- Stop tracking resolved files and refresh their native comparison display without advancing the review baseline, so ignored edits cannot revive completed controls.
- Add regressions for source filtering, global disable/restore, empty agent selection, targeted setup, portable application-scope settings and hook ownership.


## 0.14.5 - 2026-10-10

- Refresh the native disk version before saving Reject on an agent-owned dirty buffer whose content matches disk. Reuse the existing diff tab, preserve its selection, and protect unsaved manual work and divergent disk changes.
- Clear final accepted agent buffers by native reload when their exact content already exists on disk, avoiding unnecessary writes and stale-version Save conflicts.
- If a subsequent native Save fails, retain the rejected result in the unsaved editor and explain that rejection was applied but not saved. Never force Overwrite.
- Add disk-version conflict, mixed Accept/Reject, Undo, divergence and save-race regressions, plus a real extension-host scenario for repeated external CLI writes.


## 0.14.4 - 2026-10-10

- Add a real terminal-agent and inline-review screenshot to the extension README, bundled as images/cli-agent-review.png.
- Verify existing Gemini AfterTool and Cursor afterFileEdit/postToolUse user hooks and shared runtime before CLI testing. No user configuration changes were required.


## 0.14.3 - 2026-10-10

- Invalidate Explorer cached pending-review decorations after decisions even when tabs are closed and editor models released. Accept All clears resolved ARIA M markers without reopening or saving files.
- Refresh closed-file decorations on review clear/reset too; preserve remaining pending markers and leave file contents untouched.
- Add regression coverage for saved/closed files, cached markers and content preservation.

## 0.14.2 - 2026-10-10

- Soften native diff line colors to about 7% opacity and text highlights to 14%, including the text color native VS Code applies to whole additions/deletions.
- Hide ARIA's extra editor-toolbar navigation pair by default; keep native diff arrows, cross-file ARIA commands/status bar and an explicit opt-in in Quick Settings.
- Retain insertion action alignment and document native replacement/deletion CodeLens limits.
- Refresh feature metadata and canonical per-agent integration instructions; remove the duplicate prompt and provide a guarded cleanup script for obsolete development artifacts.

## 0.14.1 - 2026-10-10

- Fix startup waiting for snapshot warning dismissal, which kept baseline progress running.
- Treat binary and oversized files as normal silent skips. Log categorized counts, elapsed time and sampled file paths to Output > ARIA Review.
- Keep actionable budget/read/scan-limit warnings nonblocking so sessions become ready immediately after scanning.

## 0.14.0 - 2026-10-10

- Anchor insertion actions directly above added lines; retain native diff constraints for replacements/deletions.
- Remove current-change toolbar icons, prioritize Previous/Next, and move editor Undo to overflow.
- Add native shortcut customization, validated settings import/export and optional directory preferences.
- Add export-first Clean Extension Data; preserve settings/backups on normal uninstall and retain unrelated mixed hooks/files.
- Bound snapshot concurrency and total source bytes, cache unchanged diffs, release closed models and bound expensive comparisons.
- Use Krispachi as author display name while preserving the krispachi.aria identity; refresh release documentation.

## 0.13.1 - 2026-10-10

- Automatically save agent-owned buffers after the final acceptance, clearing the native unsaved marker.
- Preserve unsaved manual edits during decisions; add `aria.autoSaveResolved` to disable automatic acceptance saves.

## 0.13.0 - 2026-10-10

- Add Changes view, current-group actions and position counter with navigation.
- Move inline decisions before groups through preceding context; first-row replacements use toolbar/panel actions.
- Add bounded guarded Undo, restart persistence, conflict recovery, retention/clear controls and secret-file exclusions.
- Add auto-unfold, terminal focus, visibility preferences and Quick Settings.
- Require Workspace Trust for automatic setup and coalesce concurrent startup/setup requests.

## 0.12.0 - 2026-10-10

- Use softer native line backgrounds with brighter character highlights and separate old/new row defaults.
- Place block decisions after complete groups, with an EOF context fallback.
- Restore pending snapshot diff when saved source files reopen in the active session.
- Add Explorer pending-review M marker using modified-resource theme color.
- Refresh the 256px PNG logo with rounded gradient cards.

## 0.11.1 - 2026-10-10

- Remove overlapping source decorations; let the native diff own all red/green backgrounds.
- Add Previous/Next Change toolbar buttons with wraparound navigation, without changing code or decisions.

## 0.11.0 - 2026-10-10

- Add once (default), minimal and full automatic notification modes.
- Preserve welcome state across restarts/upgrades and keep automatic progress quiet outside full mode.
- Focus the next pending change after Accept/Reject, with an option to disable navigation.
- Refresh Marketplace metadata, PNG icon, documentation and package exclusions.

## 0.10.0

- Editable native diff, source tab reuse, native unsaved indicator.
- Separate manual edits from agent changes and protect unsaved manual buffers.

## 0.9.0

- Persistent directory initialization and Enable/Disable policy.
- Gemini adapter and shared CLI hook improvements.
