# test-devdigest-marketplace

A test repository for trying out changes to:

- [devdigest-ai-marketplace](https://github.com/PalkoNazar/devdigest-ai-marketplace) — a Claude Code plugin marketplace
- [dev-digest](https://github.com/PalkoNazar/dev-digest)

## Purpose

This repository is a sandbox where we install and run plugins from
[devdigest-ai-marketplace](https://github.com/PalkoNazar/devdigest-ai-marketplace)
to verify new changes before using them in real projects.

## Adding the marketplace

In Claude Code, opened in this repository:

```
/plugin marketplace add PalkoNazar/devdigest-ai-marketplace
```

Then install the plugin you want to test:

```
/plugin install <plugin-name>@devdigest-ai
```

The marketplace is named `devdigest-ai` (see its `.claude-plugin/marketplace.json`). Available plugins:

| Plugin | Description |
| --- | --- |
| `architecture-review` | Read-only architecture reviewer agent: layer and import rules with evidence for every finding |
| `engineering-paved-path` | React, Next.js, Fastify, architecture, testing and security skills for coding agents |
| `hello-world` | Example plugin with a single greeting skill |
| `research-tools` | Read-only researcher agent that answers one question with cited evidence |
| `sdd-engineering` | Spec-driven development: spec, plan, implement and verify with agents, then a workflow retro |

You can also browse them with `/plugin`.

## Testing changes

1. Make your changes in `devdigest-ai-marketplace` or `dev-digest` and push them to a branch.
2. Refresh the marketplace here: `/plugin marketplace update devdigest-ai`.
3. Reinstall or update the plugin and check how it works in this repository.
