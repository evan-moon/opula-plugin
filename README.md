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

One connector, the `opula` remote MCP server at `https://opula.io/api/mcp`, plus six skills.
Each skill fires on its own from what you ask, or you can invoke it directly as
`/opula:<name>`.

| Skill | Fires on | Leads with |
| --- | --- | --- |
| `check-in` | "오늘 포트폴리오 어때?", "how's my portfolio" | what changed since your last visit, then the verdicts |
| `analyze-holding` | "MU 어때?", "is NVDA overvalued" | the analysis frames that actually triggered, landed on your own exposure |
| `what-if` | "다음 주 FOMC 어떻게 봐?", "what if the Fed hikes" | the dollar impact, sized against a normal day |
| `fire` | "언제 은퇴할 수 있어?", "will I hit my number" | the probability band, with every assumption quoted |
| `month-end` | "이번 달 결산", "close out the month" | a complete period written in one call, then what moved |
| `import-ledger` | "거래내역 넣어줘", "here's my CSV" | your mapping confirmed before anything is written |

## License

MIT
