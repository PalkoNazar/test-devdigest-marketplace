# Spec: Channel switch script | Spec ID: SPEC-02 | Status: draft
Supersedes: —
Packages: — · Designs: — (no UI: command-line output only)

## Problem and why
This repository is a sandbox for testing plugins from the DevDigest AI marketplace. Its project settings (`.claude/settings.json`) enable the plugins from one of two release channels: `latest` (marketplace `devdigest-ai`, follows the main branch, auto-update on) and `stable` (marketplace `devdigest-ai-stable`, pinned to the `stable` ref). When a new release misbehaves, a developer has to roll back by hand-editing the plugin keys in the settings file, and later edit them back. The git history shows the cost of that: the rollback (commit c5c8a6c) and the return to latest (commit de6458a) were hand edits of several keys, and the commit titles name only `sdd-engineering` while the diffs also moved three other plugins. Nothing records to which version a switch moved the project, so a team lead can't tell afterwards who changed the channel, when, and what it was running on. The channel-switch script gives a developer a one-command switch in both directions and leaves a record that answers who, when and which version.

## Goals / Non-goals
- Goal: a developer runs one command from the repository root to move the `sdd-engineering` plugin to the stable channel (rollback) [NEEDS CLARIFICATION: Q1].
- Goal: a developer runs one command from the repository root to move the `sdd-engineering` plugin back to the latest channel [NEEDS CLARIFICATION: Q1].
- Goal: each successful switch leaves a record of who switched, when, from and to which channel, and which version [NEEDS CLARIFICATION: Q2, Q3, Q4].
- Goal: the script changes nothing except the plugin entries it is asked to move (and the record), and tells the developer what to do next for Claude Code to pick the change up [NEEDS CLARIFICATION: Q8].
- Non-goal: showing the installed or available versions of all plugins (that is SPEC-01, the plugin versions script).
- Non-goal: installing, updating, uninstalling or reloading plugins, or running Claude Code commands.
- Non-goal: any network request, including checking that the `stable` ref or the plugin exists upstream.
- Non-goal: creating, editing or removing marketplace registrations (`extraKnownMarketplaces`), including the `autoUpdate` setting [NEEDS CLARIFICATION: Q5].
- Non-goal: git commits, pushes or branch changes by the script [NEEDS CLARIFICATION: Q2].
- Non-goal: other settings layers (user settings, `.claude/settings.local.json`).
- Non-goal: channels other than `latest` and `stable`.
- Non-goal: native Windows shells without bash [NEEDS CLARIFICATION: Q6].
- Non-goal: protecting against two switches running at the same moment, or merging the records of two branches (one developer, one working copy at a time).

## User stories
- US-1: As a developer testing plugins in this repository, I want to switch the project to the stable channel with one command when a new release misbehaves, so that I can roll back without hand-editing the settings file.
- US-2: As a developer, I want to switch the project back to the latest channel with one command, so that I can resume testing new releases.
- US-3: As a team lead, I want to see later who switched the channel, when, and to which version, so that I can tell what the project ran on and who changed it.

## Acceptance criteria (EARS)
- AC-1 (US-1): WHEN a developer runs the channel-switch script with the argument `stable` from the repository root, the script SHALL change the enabled-plugin key `sdd-engineering@devdigest-ai` in the project settings to `sdd-engineering@devdigest-ai-stable`, keeping the value `true` [NEEDS CLARIFICATION: Q1].
- AC-2 (US-2): WHEN a developer runs the channel-switch script with the argument `latest` from the repository root, the script SHALL change the enabled-plugin key `sdd-engineering@devdigest-ai-stable` in the project settings to `sdd-engineering@devdigest-ai`, keeping the value `true` [NEEDS CLARIFICATION: Q1].
- AC-3 (US-1): WHEN the script changes the settings file, the script SHALL leave every other key, value, the key order and the indentation unchanged, so that the git diff of the file shows only the renamed key (assumed — P6).
- AC-4 (US-1): The script SHALL NOT modify the `extraKnownMarketplaces` section of the project settings [NEEDS CLARIFICATION: Q5].
- AC-5 (US-1): The script SHALL complete without asking any interactive question, in ≤ 1 s on a developer laptop (assumed — P8).
- AC-6 (US-1): WHEN the project is already on the requested channel, the script SHALL print a message naming that channel, SHALL end with exit status 0, SHALL NOT change the settings file and SHALL NOT add a record (assumed — P3).
- AC-7 (US-1): IF the `sdd-engineering` plugin is enabled in neither channel, or in both channels, or its value is not the boolean `true`, THEN the script SHALL print to standard error a message that names the state found, SHALL end with a non-zero exit status and SHALL NOT change any file.
- AC-8 (US-1): IF the marketplace of the requested channel is not registered in the project settings, THEN the script SHALL print to standard error a message that names the missing marketplace, SHALL end with a non-zero exit status and SHALL NOT change any file [NEEDS CLARIFICATION: Q5].
- AC-9 (US-1): IF `.claude/settings.json` is not found in the current directory, THEN the script SHALL print to standard error a message that names the file and tells the developer to run the script from the repository root, and SHALL end with a non-zero exit status.
- AC-10 (US-1): IF `.claude/settings.json` is not valid JSON, THEN the script SHALL print to standard error a message that names the file and says it could not be parsed, SHALL end with a non-zero exit status and SHALL NOT change any file.
- AC-11 (US-1): IF the script is called without an argument, with an argument other than `stable` or `latest`, or with an unknown option, THEN the script SHALL print a usage message to standard error, SHALL end with a non-zero exit status and SHALL NOT change any file.
- AC-12 (US-1): IF a tool the script needs is not available [NEEDS CLARIFICATION: Q6], THEN the script SHALL print to standard error a message naming the missing tool, SHALL end with a non-zero exit status and SHALL NOT change any file.
- AC-13 (US-1): IF `.claude/settings.json` has uncommitted changes in the working copy, THEN the script SHALL print to standard error a message saying so, SHALL end with a non-zero exit status and SHALL NOT change any file [NEEDS CLARIFICATION: Q7].
- AC-14 (US-1): IF writing the settings file fails, THEN the script SHALL leave the settings file with its original content, SHALL NOT add a record, SHALL print the cause to standard error and SHALL end with a non-zero exit status.
- AC-15 (US-3): WHEN the settings file was changed successfully, the script SHALL add exactly one record that holds the UTC time of the switch, the person who switched, the channel before, the channel after, and the version of `sdd-engineering` [NEEDS CLARIFICATION: Q2, Q3, Q4].
- AC-16 (US-3): The script SHALL write each record as one line of plain text that a person can read without the script [NEEDS CLARIFICATION: Q2].
- AC-17 (US-3): The script SHALL only append records and SHALL NOT modify or delete an existing record.
- AC-18 (US-3): The script SHALL replace control and escape characters, including line breaks, in every value it writes to a record with `?`, and SHALL cap each value at 200 characters (assumed — P5).
- AC-19 (US-3): IF the settings file was changed but the record could not be written, THEN the script SHALL print to standard error a message that says the channel was switched but not recorded, and SHALL end with a non-zero exit status.
- AC-20 (US-3): IF the version of `sdd-engineering` cannot be determined, THEN the script SHALL write `unknown` as the version in the record [NEEDS CLARIFICATION: Q3].
- AC-21 (US-1): WHEN the script has switched the channel, the script SHALL print the channel before, the channel after, and the next steps the developer must take for Claude Code to use the new channel [NEEDS CLARIFICATION: Q8].
- AC-22 (US-1): The script SHALL NOT make any network request.
- AC-23 (US-1): The script SHALL NOT read `.env*` files, credential files or tokens, and SHALL NOT write any such value to the settings file, the record or any output.
- AC-24 (US-1): WHERE the dry-run option is given (assumed — P2), the script SHALL print the change it would make and SHALL NOT change or create any file.
- AC-25 (US-3): WHERE the status option is given (assumed — P1), the script SHALL print the current channel of `sdd-engineering` and the newest record, without changing any file.

## Edge cases
- EC-1: The project is already on the requested channel → AC-6.
- EC-2: `sdd-engineering` is enabled in both channels, in neither, or set to `false` → AC-7.
- EC-3: Other plugins (e.g. `hello-world@devdigest-ai`) are on a different channel than `sdd-engineering`, as in the git history where `hello-world` stayed on latest → AC-1 / Q1 (decides whether they move too).
- EC-4: The `devdigest-ai-stable` marketplace is not registered, e.g. on an older branch → AC-8 / Q5.
- EC-5: The script is run from a subdirectory → AC-9.
- EC-6: The settings file is malformed or has no `enabledPlugins` section → AC-10 / AC-7.
- EC-7: The settings file has uncommitted edits → AC-13 / Q7.
- EC-8: Rolling back to `stable` means the project no longer auto-updates, because only the `latest` marketplace has `autoUpdate` on; the script does not change that → AC-4 / Non-goal (the next-steps message may say it, Q8).
- EC-9: The settings file is committed, so one switch changes the channel for every teammate once it is pushed → Non-goal (no commit or push) / AC-21 (next steps) / Q2.
- EC-10: The target plugin is not yet installed from the new channel, so no version of it is known at switch time → AC-20 / Q3.
- EC-11: The old channel's copy stays installed after the switch → Non-goal (no uninstall) / Q8.
- EC-12: Git user name or e-mail is unset, or contains a line break or escape sequences → AC-18 / Q4.
- EC-13: The settings file is changed but the record write fails (disk full, read-only folder) → AC-19.
- EC-14: The settings write fails halfway → AC-14.
- EC-15: The record is only on the branch where the switch was made, so the team lead sees it only after the branch is merged → Non-goal (no merge of records) / Q2.
- EC-16: A required tool is missing, e.g. no `jq` → AC-12 / Q6.
- EC-17: Double run (a developer runs `stable` twice) → AC-6.

## Non-functional
- Perf: the script finishes in ≤ 1 s (AC-5).
- Security: the script reads and writes only the project settings file and the record, never sources or evaluates content it reads, and never touches secrets (AC-23). It makes no network request (AC-22).
- Portability: it runs under bash on Linux and macOS with the dependencies decided in Q6.
- Cost: no LLM or paid external call.

## Inputs (provenance)
| Input | Source | Tag |
|---|---|---|
| Channel-to-marketplace mapping (`latest` = `devdigest-ai`, `stable` = `devdigest-ai-stable`) | the feature request and the marketplaces registered in `.claude/settings.json` | [deterministic: project settings file / fixed mapping] |
| Current channel of `sdd-engineering` | the `enabledPlugins` section of `.claude/settings.json` | [deterministic: project settings file] |
| Requested channel | command-line argument | [deterministic: command line] |
| Person who switched | git user configuration of the working copy, or the OS user as fallback (Q4) | [deterministic: git configuration] |
| Time of the switch | the machine clock, in UTC | [deterministic: computed] |
| Version of `sdd-engineering` | Claude Code's local install records on the machine (see Q3, and SPEC-01 Q1 which asks the same research question) | [deterministic: Claude Code install records — Q3] |
| Uncommitted-changes check on the settings file | git working copy state | [deterministic: git status] |
| Record of switches | the file decided in Q2 | [deterministic: record file — Q2] |

## Untrusted inputs
- Project settings file: a repo file that any branch or pull request can change → parsed as JSON data only, never sourced or evaluated. Only the one key is renamed; nothing in it is executed.
- Git user name and e-mail: free text set by each developer → treated as data, sanitised and length-capped before it goes into the record (AC-18).
- Claude Code install records: version strings come from third-party marketplace manifests → read as data, sanitised and length-capped before they go into the record (AC-18), never executed.
- Command-line arguments → only `stable`, `latest` and the options decided here are accepted, anything else leads to AC-11.

## [NEEDS CLARIFICATION]
- Q1: Which plugins the switch moves: only `sdd-engineering` (as requested), or every DevDigest AI plugin enabled in the project settings — blocks AC-1, AC-2, EC-3, Goals.
- Q2: How the "who, when, which version" record is kept: a log file in the repository that the script appends to, a git commit made by the script, or git history only — blocks AC-15, AC-16, EC-9, EC-15, Non-goal on commits.
- Q3: What "to which version" means, given that at switch time the new channel's version is not installed and not known without a network call: record the version read from the install records for the target marketplace (`unknown` if not installed), record the channel and ref only, or have the developer record the version after installing — needs research: where Claude Code stores installed plugin versions (same question as SPEC-01 Q1) — blocks AC-15, AC-20, EC-10.
- Q4: Where the "who" comes from: git user name and e-mail, only the OS user name, or both — blocks AC-15, EC-12.
- Q5: What happens when the target channel's marketplace is not registered in the project settings: stop with an error, or add the registration — blocks AC-4, AC-8, EC-4.
- Q6: Which tools the script may depend on (bash + jq, bash + python3, bash + POSIX tools only), and whether macOS bash 3.2 must be supported (SPEC-01 Q4 asks the same) — blocks AC-12, EC-16, Portability line.
- Q7: Whether the script refuses to run when the settings file has uncommitted changes — blocks AC-13, EC-7.
- Q8: What Claude Code needs after the settings change (restart, plugin reload, a manual install of the plugin from the other marketplace, removal of the old copy), and whether the script only prints this or the developer must do it — needs research: how Claude Code reacts to a changed `enabledPlugins` key that points at a not-yet-installed marketplace plugin — blocks AC-21, EC-11.
