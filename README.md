# opula plugin

Turn Claude into your own wealth analyst. Track net worth across stocks, cash, deposits and
physical assets, run FIRE projections, and settle your month-end ledger with deterministic numbers.

Powered by the hosted [opula](https://opula.io) MCP server.

## Install

In Claude (web chat, Desktop Chat tab, or Cowork): **Customize > Plugins > + > Add marketplace**,
then paste this repository URL and install the `opula` plugin.

In Claude Code:

```bash
/plugin marketplace add evan-moon/opula-plugin
/plugin install opula@opula
```

You will be asked to sign in to opula the first time a tool runs.

## What's inside

| Component | Contents |
| --- | --- |
| Connector | `opula` remote MCP server (`https://opula.io/api/mcp`) |

Skills and commands land in the next release.

## License

MIT
