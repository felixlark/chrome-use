# Chrome-Use Runbook

## Repo Scope

- Owner/escalation: Longbiao for the local runtime and install surface.
- This repo owns the CDP runtime for explicit target-bound handoff; runtime checks and commands in this file are internal maintenance only.
- Outside this repo, route ordinary Codex Web app preview, interaction, and visual acceptance through the official Chrome plugin or in-app browser according to the interaction type.
- Current runtime contract: managed profile data under `~/.chrome-use/browser-data/<channel-or-version>` over CDP; do not attach to the user's daily Chrome profile or resurrect the old app shim.

## Canonical Commands

- Install/update runtime: `bash install/install.sh`
- Install client skills: `bash install/install-codex-skill.sh` or `bash install/install-agent-skill.sh`
- Verify manifest: `bash scripts/verify-manifest.sh`
- Runtime regression suite: `bash scripts/test-runtime.sh`
- Runtime doctor: `bash runtime/chrome-use/scripts/doctor.sh`
- Open managed browser: `~/.chrome-use/bin/chrome-use-open-google-chrome`

## Routine Operations

| Trigger | Command | Expected Result | Failure Recovery |
| --- | --- | --- | --- |
| Refresh local install after source changes | `bash install/install.sh` | `~/.chrome-use/install-manifest.json` points at the current repo build | Run `bash runtime/chrome-use/scripts/doctor.sh`, then reinstall |
| Install Codex skill wrapper | `bash install/install-codex-skill.sh` | Runtime wrapper skills from this checkout are registered from `~/.codex/skills` | Restart/reload Codex if slash-command metadata or trigger validation is stale |
| Validate inspect/auth runtime | `bash scripts/test-runtime.sh` | Target selection, toolbar filtering, and runtime scripts pass | Inspect failing script output, then run the narrower `runtime/chrome-use/scripts/*` test named in the failure |
| Prove runtime attach | `~/.chrome-use/bin/chrome-use-open-google-chrome` then `bash runtime/chrome-use/scripts/doctor.sh` | Doctor sees the managed endpoint and profile | Kill the stale owner process, remove only the managed lock file/profile state called out by doctor, and retry |

## Troubleshooting

| Trigger | Command | Expected Result | Failure Recovery |
| --- | --- | --- | --- |
| Wrong tab or window is controlled | Inspect target ids through the runtime doctor/debug output | Selected `targetId` / `bindingId` matches the intended page | Rebind explicitly; do not fall back to active-tab heuristics |
| `/inspect` waits forever | Read `~/.chrome-use/state/inspect/127.0.0.1-9223/session.json` | `activeWorkflowId` identifies the pending workflow | Resume with the matching await/capture command instead of starting a competing workflow |
| Managed browser fails to start | `bash runtime/chrome-use/scripts/doctor.sh` | Doctor reports the owner process, browser path, and profile root | Stop the stale owner, rerun `install.sh`, then launch the managed browser again |

## Verification

- Run `bash scripts/verify-manifest.sh` and `bash scripts/test-runtime.sh` after runtime or packaging changes.
- For install changes, reinstall and read back `~/.chrome-use/install-manifest.json`.
- For real runtime behavior, prove the intended target id or selection workflow from runtime diagnostics, not just mocked output.

## Release/Deploy

- Keep packaging cross-agent and cross-platform.
- Keep Codex-specific metadata optional and isolated from the public runtime.
- Public release docs must describe `~/.chrome-use` as the managed install root rather than exposing local checkout paths.

## Guardrails

- Do not re-expand this repo into generic Codex web preview or UI acceptance work now covered by official Chrome plugin + In-app Browser.
- Use project-specific CDP runtime only when the task is explicitly about this repo's internal runtime maintenance.
- Do not silently switch to ordinary `Google Chrome` because it has convenient cookies or extensions.

## Known State

- Runtime lives under `runtime/chrome-use/`; public runtime wrappers are maintained through this repo's install scripts.
- Managed install root is `~/.chrome-use`.
- Port `127.0.0.1:9223` and profile ownership are machine-specific; recheck live state before mutating runtime files.

## Browser Automation Constraint
- Follow the global `~/.codex/AGENTS.md` official browser/GUI automation policy: Chrome plugin for signed-in browser state, Browser plugin for unauthenticated rendering, and Computer Use for native desktop boundaries. Do not bypass it with AppleScript or `osascript` unless the global exception rules are met.
- Keep only repo-specific verification surfaces here; do not copy the full global policy block into this runbook.

## Worktree Policy

- Follow the global `~/.codex/AGENTS.md` main-first development rule: work in the current Local checkout by default and use a worktree only when the global exception list applies.
- Branch names should use `codex/<repo>-<short-task>`; manual long-lived worktree directories should use `~/Projects/<repo>-<short-task>`.
- Initialize dependencies inside each worktree and keep ports, databases, device/simulator state, build outputs, and ignored local config isolated per checkout.
- Preserve existing dirty checkouts. Inspect `git status --short` before editing, and do not stash, commit, remove, or migrate user changes unless explicitly asked.
- After merge or abandonment, clean up with `git worktree remove <path>` and use `git worktree prune` only for stale metadata.
