# Stoop — plan through 22 October

Finish Thursday 22 October 2026, in the morning. The Devpost listing closes Friday 23 October 2026 at 12:00pm PDT. The extra day is cushion, not scope.

Stoop is an Alexa+ add-on for the person at home in the hour before a stranger arrives: a repair, a nurse, an installer. The demo household is one fridge repair. Technician Priya, Thursday 2 to 4pm. The same server is the thing a real add-on would sit on later.

Primary track: Alexa+. Mini challenges: AWS Builder and Open Source. A project can win one track prize and one mini-challenge prize, so both mini challenges go on the form and only one of them can pay. Alexa+ first place is $25,000 plus $15,000 in AWS credits.

## What a 5 is

Scores are equal, 1 to 5. The friction log is a separate bonus, up to 10%, applied by the stage-two panel if stage one recommends it.

**Technical implementation.** One `/mcp` endpoint on MCP spec 2025-11-25, Streamable HTTP. The official SDK accepts POST and GET. A test proves four things: `initialize` returns `Mcp-Session-Id`, the next call sends `MCP-Protocol-Version: 2025-11-25`, `tools/list` and `tools/call` succeed, and a bad `Origin` gets 403. The agent is the Strands SDK on Amazon Bedrock, and the tool trace is visible. That trace is the AWS Builder write-up. The repo calls this stack from code, not from a paragraph in the README.

**Design.** The page is a household conversation in large type. Two MCP App cards: the visit, and the checklist. A "show the protocol" drawer stays closed while the story plays and opens when a judge asks. Record the story first, then the drawer.

**Impact.** The user is specific. Someone is home, about an hour before a stranger arrives, and needs three things: the window, what to do before the door opens, and a message the other adult can read. The README says, in one paragraph, that this server is the add-on a household would actually register.

**Idea.** Context across the session is the product. "They," "the gate," and "my partner" resolve because the MCP session still holds the visit and the household. Amazon's own scale calls a single-turn question bot the obvious Alexa+ entry, and calls a stateful add-on that orchestrates across services the creative one. Turn two is nonsense unless turn one happened. Turn three calls the weather for the visit window and answers with the visit notes in the same breath.

The friction log is written when something actually breaks. Each entry has the task, the steps, the expected result, the actual result, a severity, the workaround, and one change we want from Amazon or the SDK. Five real entries by Wednesday. No invented ones.

## What we are building

A Python server and a small web host.

- The MCP server exposes four tools and two `ui://` card resources. Household data is `fixtures/household.json`. No accounts.
- Open-Meteo is the one external service. No key. The visit window and the forecast have to appear in the same answer.
- Strands talks to that server over Streamable HTTP, on one session, with Bedrock as the model. Sunday's terminal run prints every tool call.
- The household page is an MCP client. It can play the four turns by calling the tools directly, so a judge needs no AWS account. The Bedrock path is the same tools, and the video shows that path.
- `draft_update` writes a draft. It does not send a text. The partner's name comes from the session.

Pin the protocol at **2025-11-25**. That is the minimum the rules name, and it is the revision that still has `Mcp-Session-Id`. The July 2026 revision drops protocol sessions. This product is the session, so we do not follow the SDK upward if that means losing the session id. If the current SDK refuses to mint one, that break goes in the friction log on Friday and we pin a release that still speaks 2025-11-25.

## Tracks, and where they show up

| Track | Where a judge sees it |
| --- | --- |
| Alexa+ | `/mcp` on 2025-11-25, four tools, two MCP App cards, the page acting as a real MCP client |
| AWS Builder | Strands `Agent` on Amazon Bedrock, tool trace in the terminal and in the video |
| Open Source | This repo, MIT license, created inside the hackathon window |

Product feedback on Wednesday covers the MCP Python SDK, MCP Apps, Strands, Amazon Bedrock, and Open-Meteo. Feature requests use the form's urgency scale: Critical, Important, Nice-to-have. They come from the friction log, not from a wishlist written in advance.

## The four turns

The contract is `docs/demo-script.md`. The household is `fixtures/household.json`. Short form:

1. "When is someone coming about the fridge?" Tool: `find_visit`. Card: Thursday, 2 to 4pm, technician Priya. The session stores the visit.
2. "What should I do before they get here?" Tool: `prep_checklist`, with no visit id in the arguments. The server reads the session. Card: clear a path, secure the dog, an adult at home.
3. "It is supposed to rain. Do I leave the side gate open?" Tool: `weather_for_window`, window taken from the session visit, forecast from Open-Meteo. The answer cites that forecast and Priya's note about the side gate.
4. "Text my partner that." Tool: `draft_update`. The utterance never says the partner's name. The draft is addressed to Sam because the household on the session says so.

## Calendar

| Day | Done when | Score it feeds |
| --- | --- | --- |
| Thu 15 | Repo, MIT license, four-turn script, household fixture, friction log file started | Idea, Open Source |
| Fri 16 | Server on `/mcp`. Test passes for session id, `tools/list`, `tools/call`, and Origin 403. `find_visit` returns the fixture | Technical |
| Sat 17 | All four tools, Open-Meteo, and two MCP App cards. Second turn reads the session. A call to `prep_checklist` before any visit fails in a way the agent can say out loud | Technical, Idea |
| Sun 18 | Strands on Bedrock runs the four utterances in a terminal. Tool trace printed. Same session for all four turns | Technical, AWS Builder |
| Mon 19 | Household page in large type. Empty state and unknown-visit state. Protocol drawer closed by default, and it shows the POST body and the session headers when opened | Design |
| Tue 20 | Two takes. Public YouTube, under 3 minutes, English, no copyrighted music | Design, Impact |
| Wed 21 | README a judge can run in 10 minutes with no accounts. Product feedback. Five real friction-log entries. Feature requests with urgency | Impact, bonus |
| Thu 22 | Submit in the morning. Add the GitHub reviewers the same day | — |

Video shape, 2 minutes 40 seconds:

- 20 seconds. The person, the door, the hour before someone arrives.
- 80 seconds. The four turns, cards on screen.
- 40 seconds. Open the protocol drawer on the weather turn, then the Bedrock tool trace.
- 20 seconds. Why this server would ship as an add-on: the visit, the prep, and the message to the other adult, on a session Alexa+ already knows how to call.

## Repo, by the day it appears

Already here: `LICENSE`, `README.md`, `PLAN.md`, `docs/demo-script.md`, `docs/friction-log.md`, `fixtures/household.json`.

| Day | Adds |
| --- | --- |
| Fri 16 | Python project, `/mcp` server, session store, `find_visit`, protocol test |
| Sat 17 | `prep_checklist`, `weather_for_window`, `draft_update`, Open-Meteo client, `ui://stoop/visit-card.html`, `ui://stoop/checklist-card.html` |
| Sun 18 | `python -m stoop.demo` — Strands, Bedrock, printed trace, one session across four turns |
| Mon 19 | Household page. Conversation, two cards, empty state, unknown visit, protocol drawer |
| Wed 21 | Judge runbook in the README, `docs/product-feedback.md`, filled friction log, feature requests |

AWS credentials stay in the environment. Nothing secret goes in the repo.

## Judge path

Ten minutes, no accounts:

1. Install dependencies.
2. Run the protocol test.
3. Start the server and open the household page.
4. Play the four lines. Turn two uses the visit from turn one. The weather line calls Open-Meteo. The draft is addressed to Sam.
5. Open the protocol drawer and read the POST.

The Bedrock trace is the Sunday command and the video. A judge without AWS credentials still sees the session, the tools, and the cards.

## Thursday morning

Submit on Devpost before noon local time. Fields ready on Wednesday: description, repo URL, YouTube URL, primary track Alexa+, mini challenges AWS Builder and Open Source, product feedback, feature requests, friction log.

The repo is public and MIT, which is the open-source path in the rules. The same morning, add these GitHub users so the private-repo reviewer list is covered too: `chris-trag`, `knmeiss`, `giolaq`, `anishamalde`, `mosesroth`, `emersonsklar`.

Open-source form: contribution URL is this repo, created during the window, MIT. Say what the session does, how the four tools work, and why a stateful visit is the add-on.

## Risks we already know about

- **SDK moves past sessions.** If `initialize` on the current SDK does not return `Mcp-Session-Id`, pin the SDK and log it. Do not rewrite Stoop onto the sessionless revision.
- **Bedrock access.** Model access or credentials can block Sunday. The page and the protocol test do not wait on that.
- **The forecast may be dry.** The gate stays latched either way. The answer quotes the real probability from Open-Meteo. We do not hard-code rain.
- **MCP App iframe handshake.** The cards are `ui://` resources linked with `_meta.ui.resourceUri`. If the host handshake breaks, the page still renders the tool's structured content in those same two cards, and the break goes in the friction log.
- **Scope.** No login, no real SMS, no second household, no fifth tool. A nurse or an installer is the same visit shape later. The demo stays the fridge.

## Day 1

Done. The script, the fixture, the license, and the friction log are in the repo. The log has no entries yet. The first one lands the day something breaks.
