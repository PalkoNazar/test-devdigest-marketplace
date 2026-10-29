# Spec: Plugin versions script | Spec ID: SPEC-01 | Status: draft
Supersedes: —
Packages: — · Designs: — (no UI: command-line output only)

## Problem and why
This repository is a sandbox for testing plugins from the DevDigest AI marketplace before they are used in real projects. Its project settings (`.claude/settings.json`) enable plugins from two marketplaces, `devdigest-ai` (main branch) and `devdigest-ai-stable` (the `stable` ref), and the project has already switched plugins between them. Today a developer can't see in one place which plugin runs from which channel and at which installed version. A plugin that is enabled in settings but missing on the machine stays unnoticed until a test behaves wrongly. CI can't record or check these versions at all. The plugin-versions script gives developers a readable listing and gives CI the same data as JSON.

## Goals / Non-goals
- Goal: a developer runs the plugin-versions script from the repository root and sees every enabled DevDigest AI plugin with its marketplace and installed version.
- Goal: a developer sees a clear message for each plugin that is enabled in the project settings but not installed.
- Goal: CI can get the same data as one JSON document.
- Non-goal: showing the latest available version, or whether an update is available (no marketplace or network lookup).
- Non-goal: installing, updating, enabling or disabling plugins, or changing any settings file.
- Non-goal: plugins from marketplaces other than the DevDigest AI ones [NEEDS CLARIFICATION: Q3].
- Non-goal: native Windows shells without bash [NEEDS CLARIFICATION: Q4].
- Non-goal: settings layers other than the project settings file (user settings, local overrides) [NEEDS CLARIFICATION: Q6].

## User stories
- US-1: As a developer testing plugins in this repository, I want to list each enabled DevDigest AI plugin with its marketplace and installed version, so that I know exactly which plugin build I am testing.
- US-2: As a developer, I want a clear message when an enabled plugin is not installed, so that I can install it before I test.
- US-3: As a developer maintaining CI, I want the same data as JSON, so that a CI job can record or check plugin versions without parsing text.

## Acceptance criteria (EARS)
- AC-1 (US-1): WHEN a developer runs the plugin-versions script from the repository root without options, the script SHALL print one line per enabled DevDigest AI plugin entry in the project settings [NEEDS CLARIFICATION: Q3].
- AC-2 (US-1): The script SHALL show on each plugin line the plugin name, the marketplace name and the installed version.
- AC-3 (US-1): The script SHALL treat a plugin entry as enabled only when its value in the project settings is the boolean `true`.
- AC-4 (US-1): The script SHALL take the installed state and installed version of each plugin only from Claude Code's local install records on the machine where it runs [NEEDS CLARIFICATION: Q1].
- AC-5 (US-1): IF a plugin has several install records (for example in different install scopes), THEN the script SHALL show the version that applies to this project [NEEDS CLARIFICATION: Q5].
- AC-6 (US-1): The script SHALL list plugins sorted by plugin name, then by marketplace name (assumed — P2).
- AC-7 (US-1): IF an install record has no version, THEN the script SHALL show `unknown` as the installed version (assumed — P5).
- AC-8 (US-2): IF an enabled plugin has no install record for its marketplace, THEN the script SHALL show `not installed` on that plugin's line in place of the version.
- AC-9 (US-2): IF a plugin is installed only from the other DevDigest AI marketplace (e.g. enabled as `hello-world@devdigest-ai-stable` but installed as `hello-world@devdigest-ai`), THEN the script SHALL report it as `not installed` (assumed — P6).
- AC-10 (US-2): IF Claude Code's install records don't exist on the machine, THEN the script SHALL report every enabled plugin as `not installed` (assumed — P7).
- AC-11 (US-2): WHEN at least one enabled plugin is not installed, the script SHALL end with an exit status of [NEEDS CLARIFICATION: Q2].
- AC-12 (US-1): WHEN every enabled plugin is installed, the script SHALL end with exit status 0.
- AC-13 (US-3): WHERE the JSON option is given (assumed — P1), the script SHALL print to standard output exactly one JSON document holding the same plugin entries, in the same order, as the text output.
- AC-14 (US-3): WHERE the JSON option is given, the script SHALL include in each JSON entry the plugin name, the marketplace name, whether the plugin is installed, and the installed version.
- AC-15 (US-3): WHERE the JSON option is given, IF a plugin is not installed, THEN the script SHALL give that entry a null installed version.
- AC-16 (US-3): WHERE the JSON option is given, the script SHALL write every error, warning and hint only to standard error (assumed — P1).
- AC-17 (US-1): IF no DevDigest AI plugin is enabled in the project settings, THEN the script SHALL print `No DevDigest AI plugins are enabled in .claude/settings.json` in text mode.
- AC-18 (US-3): WHERE the JSON option is given, IF no DevDigest AI plugin is enabled, THEN the script SHALL print a JSON document with an empty plugin list.
- AC-19 (US-1): IF the project settings file is not found in the current directory, THEN the script SHALL print to standard error a message that names the file and tells the developer to run the script from the repository root.
- AC-20 (US-1): IF the project settings file is not valid JSON, THEN the script SHALL print to standard error a message that names the file and says it couldn't be parsed.
- AC-21 (US-1): IF Claude Code's install records exist but can't be read or parsed, THEN the script SHALL print to standard error a message that names the install records as the cause.
- AC-22 (US-1): IF a tool the script needs is not available [NEEDS CLARIFICATION: Q4], THEN the script SHALL print to standard error a message naming the missing tool.
- AC-23 (US-1): IF the script is called with an unknown option, THEN the script SHALL print a usage message to standard error.
- AC-24 (US-1): IF the script stops for any error in AC-19 to AC-23, THEN the script SHALL end with a non-zero exit status different from the one in AC-11 [NEEDS CLARIFICATION: Q2].
- AC-25 (US-1): The script SHALL NOT write, create or delete any file.
- AC-26 (US-1): The script SHALL NOT make any network request.
- AC-27 (US-1): The script SHALL print plugin names, marketplace names and versions in text mode with control and escape characters replaced by `?` (assumed — P4).

## Edge cases
- EC-1: The same plugin is enabled from both `devdigest-ai` and `devdigest-ai-stable` → AC-1 (two separate lines, one per entry).
- EC-2: A plugin entry is present with value `false` or a non-boolean value → AC-3 (not listed).
- EC-3: A plugin is enabled from a non-DevDigest marketplace → Non-goal / Q3.
- EC-4: The project settings file has no `enabledPlugins` section, or it is empty → AC-17 / AC-18.
- EC-5: The project settings file is missing, e.g. the script is run from a subdirectory → AC-19 (P3 proposes support for subdirectories).
- EC-6: The project settings file is malformed JSON → AC-20.
- EC-7: A fresh CI runner where Claude Code was never run, so no install records exist → AC-10 / AC-11 / Q2.
- EC-8: The install records are corrupt or unreadable → AC-21.
- EC-9: The plugin is installed only from the other DevDigest AI channel → AC-9.
- EC-10: The plugin has several install records (user, project or local scope, or another project's path) → AC-5 / Q5.
- EC-11: The install record carries no version, or a commit hash instead of a semantic version → AC-7 (a commit hash is shown as recorded).
- EC-12: A plugin or marketplace name in settings contains terminal escape sequences or very long text → AC-27 (AC-13 relies on correct JSON encoding).
- EC-13: A required tool is missing on the machine → AC-22 / Q4.
- EC-14: An unknown or misspelled option → AC-23.
- EC-15: A local settings override disables or enables a plugin that the project settings say otherwise → Non-goal / Q6.

## Non-functional
- Perf: the script SHALL finish in ≤ 1 s for ≤ 50 enabled plugins on a developer laptop (assumed — P8).
- Security: the script reads only the project settings file and Claude Code's install records, and never reads `.env*`, credential or token files. It never evaluates or sources the content it reads (AC-25, AC-26, AC-27).
- Portability: it runs under bash on Linux and macOS with the dependencies decided in Q4.
- Cost: no LLM or paid external call.

## Inputs (provenance)
| Input | Source | Tag |
|---|---|---|
| Enabled plugin entries (`<plugin>@<marketplace>` → `true`/`false`) | project settings file `.claude/settings.json`, `enabledPlugins` section | [deterministic: project settings file] |
| Marketplace name of each plugin | the part after `@` in the enabled-plugin entry | [deterministic: computed from the project settings file] |
| Set of DevDigest AI marketplaces (`devdigest-ai`, `devdigest-ai-stable`) | the feature request. The `extraKnownMarketplaces` section of the project settings could replace the fixed list (Q3) | [deterministic: project settings file / fixed list — Q3] |
| Installed state and installed version per plugin and marketplace | Claude Code's local install records on the machine (location, format and scope rules: Q1) | [deterministic: Claude Code install records — Q1] |
| Output mode (text or JSON) | command-line option | [deterministic: command line] |

## Untrusted inputs
- Project settings file: a repo file that any branch or pull request can change → parsed as JSON data only, never sourced or evaluated. Names are printed with control characters replaced (AC-27) and JSON-encoded in JSON mode (AC-13).
- Claude Code install records: version strings come from third-party marketplace manifests → read as data, never executed. Values that are missing or malformed lead to AC-7 or AC-21.
- Command-line arguments → only known options are accepted, and anything else leads to AC-23.

## [NEEDS CLARIFICATION]
- Q1: needs research: where Claude Code stores the record of installed plugins (path, honouring any config-dir override), its format, how it records version, marketplace and install scope (user / project / local), and whether it is a stable interface to read — blocks AC-4, AC-5, AC-10, AC-21, EC-10, EC-11.
- Q2: Exit status when at least one enabled plugin is not installed, and the error exit status — blocks AC-11, AC-24, EC-7.
- Q3: Which marketplaces count as "DevDigest AI": exactly `devdigest-ai` and `devdigest-ai-stable`, every marketplace whose source is the `PalkoNazar/devdigest-ai-marketplace` repo in `extraKnownMarketplaces`, or all enabled plugins whatever the marketplace — blocks AC-1, EC-3.
- Q4: Which tools the script may depend on (bash + jq, bash + python3, or bash + POSIX tools only), and whether macOS's bash 3.2 must be supported — blocks AC-22, EC-13, the Portability line in Non-functional.
- Q5: When a plugin has several install records, which version to show: the one that applies to this project, or all of them — blocks AC-5, EC-10.
- Q6: Read only the project settings file (as requested), or also apply the local override file `.claude/settings.local.json` — blocks EC-15.
