# Wakeshift plugins for coding agents

[Wakeshift](https://wakeshift.ai) wakes your AI agent's conversation when something happens in an app it's connected to. This marketplace has Wakeshift's plugin for each agent that installs plugins with its own command, and for Cursor, which lists it in the Cursor Marketplace. Each plugin adds the agent's hooks, and the hooks run Wakeshift's connector.

| Agent | Install | Update |
| --- | --- | --- |
| Claude Code | `claude plugin install wakeshift --marketplace wakeshift/agent-plugins --config issuer=https://app.wakeshift.ai` | `claude plugin marketplace update wakeshift && claude plugin update wakeshift@wakeshift` |
| Codex | `codex plugin marketplace add wakeshift/agent-plugins && codex plugin add wakeshift@wakeshift` | `codex plugin marketplace upgrade wakeshift && codex plugin add wakeshift@wakeshift` |
| GitHub Copilot CLI | `copilot plugin marketplace add wakeshift/agent-plugins && copilot plugin install wakeshift@wakeshift` | `copilot plugin update wakeshift` |
| Cursor | From the Cursor Marketplace: open **Customize**, find **Wakeshift** and choose **Install** ([plugins/cursor](plugins/cursor)) | Cursor updates it once the Marketplace has reviewed the new version |

You don't normally type the Claude Code, Codex and Copilot CLI commands yourself. When your agent first needs a plugin, a Wakeshift app's tool result offers the command, and your agent asks you before it runs it. Then restart the agent:

- **Claude Code** has nothing to approve, but loads a newly installed or updated plugin only when a session starts: type `/reload-plugins` (or restart with `claude --continue`).
- **Codex** asks you to trust the plugin's hooks the first time it starts after an install or update. Choose "Trust all and continue".
- **Copilot CLI** has nothing to approve.
- **Cursor** has nothing to approve. Restart `cursor-agent`; the Cursor app loads the plugin by itself.

[Agents](https://wakeshift.ai/docs/agents) says what each install changes.

## What a plugin does

Each plugin directory, `plugins/<agent>`, holds:

- **The agent's hooks.**
- **`bin/wakeshift`:** a small POSIX shell launcher that the hooks run.
- **`wakeshift.env`:** the Wakeshift server the plugin trusts, and the version of the connector it pins.
- **`SHA256SUMS`:** the connector's checksum for each platform.

On a hook's first run, the launcher does three things:

1. **Downloads the connector.** It fetches it from that version's [signed release](https://github.com/wakeshift/connectors/releases), checks it against the plugin's `SHA256SUMS`, and installs it to `~/.wakeshift/bin/wakeshift`. A download that doesn't match is deleted, and nothing is installed.
2. **Sets up the connector.** It runs `wakeshift install --harness <agent> --issuer <server> --yes --plugin`. That makes the connector trust that Wakeshift server and add any allow rules. It also removes the hooks an earlier `curl … | sh` install wrote to the agent's settings, so they don't run twice.
3. **Runs the hook.**

Later runs only check that the connector is the pinned version or newer, so they use no network. A connector that's already newer stays.

## Other Wakeshift servers

The `main` branch trusts production, `https://app.wakeshift.ai`. The `staging` branch trusts Wakeshift's staging server, `https://app.staging.wakeshift.ai`:

```sh
codex plugin marketplace add wakeshift/agent-plugins --ref staging
```

Claude Code's plugin trusts the server in its `issuer` option instead (`--config issuer=<server>` when you install it, or `/plugin configure wakeshift@wakeshift`), so any branch works with any server; `wakeshift/agent-plugins#staging` is the staging branch.

## Uninstall

- **Claude Code:** `claude plugin uninstall wakeshift@wakeshift`, then `/reload-plugins`
- **Codex:** `codex plugin remove wakeshift@wakeshift`
- **Copilot CLI:** `copilot plugin uninstall wakeshift`
- **Cursor:** uninstall Wakeshift from **Customize**.

Then `~/.wakeshift/bin/wakeshift uninstall --harness <claude-code|codex|copilot|cursor>` removes what the connector set up: the server it trusted and the conversations it connected.

## Contributing and security

This repository is generated from Wakeshift's main repository: the plugins from `plugins/agent-plugins`, and each plugin's pin by Wakeshift's release workflow. The launcher's tests are published with the connector in [wakeshift/connectors](https://github.com/wakeshift/connectors), under `plugins/agent-plugins`. See [CONTRIBUTING](CONTRIBUTING.md).

Report a security problem privately to support@wakeshift.ai, not in an issue ([SECURITY](SECURITY.md)).

## License

Apache License 2.0; see [LICENSE](LICENSE) and [NOTICE](NOTICE).
