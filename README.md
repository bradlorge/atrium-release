# Atrium release artifacts

Auto-published build artifacts for the Atrium Harness runner (and its skills pack).
Installed runners auto-update from the releases here via electron-updater.

Source lives in the private `bradlorge/atrium` repo; this repo is public and
artifacts-only so the shipped app can read releases without a token.

## Install the Atrium skills plugin

This repo is also a **Claude Code plugin marketplace**. Any developer can install
the Atrium skills pack — the operating rituals for working a product through
Atrium's compact MCP surface — into their own `claude` session:

```
/plugin marketplace add bradlorge/atrium-release
/plugin install atrium@atrium
```

Pin a version with `/plugin install atrium@atrium@0.2.0`.

The plugin itself lives under [`plugins/atrium`](plugins/atrium) and is registered
by the root [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).
See [`plugins/atrium/README.md`](plugins/atrium/README.md) for what the pack needs
to run (an Atrium MCP server via `--mcp-config` + a product-scoped token) and the
full tool vocabulary.

The same pack is attached to each Harness release as `atrium-skills-<version>.zip`
for offline/pinned use.
