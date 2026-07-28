# Setup

## Connect the connector first

Installing the plugin does not connect it. In the Claude apps the bundled `opula` connector starts
disconnected, and it is connected from **Settings > Plugins > Opula > Connectors**. Until that
happens the skills load but no opula tool exists to call.

So when an opula tool is missing or every call fails to resolve, the cause is almost always this
step rather than the account or the skill. Point the user at that screen in one line instead of
retrying the call or improvising an answer from the web. In Claude Code there is no such screen:
the sign-in prompt appears on the first tool call.

## Then: the empty account

Once connected, the one thing that blocks a useful first answer is an empty account.

## First run

Call `setup_status`. It returns the account's real state and, when the account is empty, the FIRST
RUN playbook. Follow that playbook rather than reciting a setup flow from memory, because it
reflects what this account is actually missing.

If the account has no transactions, the fastest path to a real answer is the `import-ledger` skill:
the user pastes whatever they already have, a brokerage export, a CSV, a screenshot of a holdings
screen, or just a list, and it lands in one write.

Do not walk the user through an interview when they have a file. Ask for what they already have
first.

## What "connected" does not mean

A connected account with no data still answers every question with nothing. If a first question
arrives before any data exists, say so plainly in one line and offer the import, rather than
returning an empty brief as though it were a finding.

## Currency

opula stores everything in USD internally and asks on both sides. On input, ask which currency a
figure is in before writing it, since conversion happens at the entry date via historical FX. On
output, ask which currency to display in. Never let a USD figure appear in a non-USD answer without
saying so.
