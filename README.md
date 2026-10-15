# Stoop

Stoop is an Alexa+ add-on for the person at home in the hour before a stranger arrives: a repair, a nurse, an installer.

The demo is one fridge repair. Technician Priya comes Thursday, 2 to 4pm. The household lives in [`fixtures/household.json`](fixtures/household.json), so a judge can run the visit with no accounts. Weather comes from Open-Meteo, which needs no key. The agent has to put the visit and the forecast in the same answer, and the second turn only makes sense because the first turn is still on the MCP session.

This repo is a new project for the [Build, Ship, Shape Amazon Developer Hackathon](https://amazonappdev2026.devpost.com/). Primary track is Alexa+. Mini challenges are AWS Builder and Open Source. The public deadline is 23 October 2026, 12:00pm PDT. We finish on 22 October.

The server is not in the repo yet. Friday 16 October is the `/mcp` endpoint. A judge-ready runbook replaces this page on Wednesday 21 October.

| | |
| --- | --- |
| [PLAN.md](PLAN.md) | The week, and what a 5 looks like on each score |
| [docs/demo-script.md](docs/demo-script.md) | The four turns |
| [fixtures/household.json](fixtures/household.json) | The household |
| [docs/friction-log.md](docs/friction-log.md) | Real breakage only |

License: MIT.
