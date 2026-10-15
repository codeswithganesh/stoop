# Four-turn script

This is the contract for the server, the agent, the page, and the video. If a later day needs a new behavior, it gets written here first.

The person at home is waiting on a fridge repair. A technician is coming to the house. There is a dog, a side gate, and another adult who is not in the room. The household is `fixtures/household.json`. No accounts. The partner's name is Sam, and the person never says "Sam" out loud.

The visit day on the card is the word Thursday. The server resolves that to a real calendar date when it asks Open-Meteo: the next Thursday in `America/Los_Angeles`, including today when today is Thursday and 4pm has not passed. The card keeps saying "Thursday, 2 to 4pm". The protocol drawer can show the resolved date.

## What the session holds

`initialize` loads the household onto the MCP session identified by `Mcp-Session-Id`.

| Field | After initialize | Written by |
| --- | --- | --- |
| `partner_name` | `Sam` | fixture, at initialize |
| `timezone`, latitude, longitude | from the fixture | fixture, at initialize |
| `current_visit` | empty | `find_visit` |
| `last_prep` | empty | `prep_checklist` |
| `last_weather` | empty | `weather_for_window` |

A new session does not see another session's visit. That is the product: the memory is the MCP session, and it starts when the conversation starts.

## Words that only work because of the session

| Spoken | Resolves to | Because |
| --- | --- | --- |
| "someone" / "they" | Technician Priya, the fridge visit | `find_visit` stored `current_visit` |
| "before they get here" | That same visit's prep list | `prep_checklist` reads `current_visit` and takes no visit id |
| "the side gate" | Priya's note: she uses the side gate, the dog is in that yard, the gate stays latched until she is there | the note came back with the visit |
| "it is supposed to rain" | A belief. The agent checks the visit window | `weather_for_window` uses the session window, not a date from the sentence |
| "that" | The gate decision just given | the model passes that decision as the draft body |
| "my partner" | Sam | `draft_update` addresses the draft from `partner_name` on the session |

## Tools

Four tools. No fifth.

### `find_visit`

Argument: `query` (string).

Match `query` against the fixture's `match` list, case-insensitive. "fridge" hits. "refrigerator" hits. "plumber" misses.

On a hit, store the visit as `current_visit` and return:

- `card_when`: `Thursday, 2 to 4pm`
- `card_who`: `Technician Priya`
- `card_title`: `Fridge repair`
- `notes`: the side-gate note from the fixture
- `ui`: resource `ui://stoop/visit-card.html`

On a miss, leave `current_visit` unchanged and return a not-found the agent can say: there is no visit for that query. The fridge repair is the one on the household list.

### `prep_checklist`

Arguments: none. A visit id in the arguments is the wrong shape. The server reads `current_visit`.

If `current_visit` is empty, return an error the agent can say: no visit yet in this session, ask when someone is coming. This is how we know turn two depends on turn one.

On success, store `last_prep` and return:

- items, in this order: `Clear a path`, `Secure the dog`, `An adult at home`
- `ui`: resource `ui://stoop/checklist-card.html`

### `weather_for_window`

Arguments: none. The window is `current_visit`. If there is no visit, same class of error as `prep_checklist`.

Call Open-Meteo, no key:

`https://api.open-meteo.com/v1/forecast`

Query: fixture latitude and longitude, hourly `precipitation_probability` and `precipitation`, timezone `America/Los_Angeles`. Read the hours of the resolved Thursday from 14:00 through 15:00. Those two hours are the 2 to 4pm window.

Return the resolved date, the probability for each of those hours, and a short summary. Store that as `last_weather`. The tool does not decide the gate. The note on the visit does.

If Open-Meteo fails, return the error. The agent still answers the gate from the visit note: keep it latched until Priya is there. Say that the forecast could not be checked.

### `draft_update`

Argument: `body` (string). The recipient is not an argument.

Address the draft to `partner_name` on the session. If a model puts a name in the body, the `to` field is still Sam. `sent` is false. This tool does not call a phone, an email API, or Alexa messaging.

The body is the gate decision from turn three. If `body` is empty, the tool returns an error: there is nothing to pass on yet.

## The four turns

Play them in order, on one session. The video records this order.

### Turn 1

**Say:** When is someone coming about the fridge?

**Tool:** `find_visit` with `query` `fridge`.

**Card:** Thursday, 2 to 4pm. Technician Priya.

**Say back:** Priya is coming Thursday, 2 to 4pm, for the fridge. Include the window and her name. The side-gate note can wait. This turn is the visit.

**Session after:** `current_visit` is the fridge visit.

### Turn 2

**Say:** What should I do before they get here?

**Tool:** `prep_checklist` with no visit id.

**Card:** Clear a path. Secure the dog. An adult at home.

**Say back:** Those three items, tied to Priya's visit. "They" is Priya. The agent does not ask which visit.

**Session after:** `last_prep` is those three items.

The proof this turn is stateful: on a fresh session, the same sentence calls `prep_checklist` and the server says there is no visit yet. The demo does not show that failure. Saturday's test does.

### Turn 3

**Say:** It is supposed to rain. Do I leave the side gate open?

**Tool:** `weather_for_window`. One call. The arguments do not contain a date, a city, or the word gate. The session visit supplies the window. Open-Meteo supplies the rain. The visit note supplies the gate.

**Answer, both facts in the same reply:**

- The forecast for Thursday 2 to 4pm, with the precipitation probability Open-Meteo returned.
- The note: Priya uses the side gate, and the dog is in that yard.
- The decision: keep the gate latched. Open it when Priya is there, not before.

Rain changes the reason, not the decision. At or above 40% on either hour in the window, say the path will be wet and an open gate lets the dog out. Below 40%, say the window looks dry and the gate still stays latched because of the dog.

The spoken belief ("it is supposed to rain") can be wrong. Quote the forecast anyway. Do not pretend it is raining when the probability is low.

**Session after:** `last_weather` holds the Open-Meteo summary.

### Turn 4

**Say:** Text my partner that.

**Tool:** `draft_update`. The body is the gate decision from turn 3. The recipient argument does not exist.

**Show:** a draft, not a sent message.

**To:** Sam. The source line a judge can see: household on this MCP session.

**Body shape:**

> Sam — Priya is coming Thursday, 2 to 4pm, for the fridge. {forecast in one clause.} Keep the side gate latched until she is there. An adult will be home.

`{forecast in one clause}` uses `last_weather`, for example "Rain is likely then, about 60% at 2pm." or "The forecast looks dry, about 10% at 2pm."

**Say back:** the draft, and that it has not been sent. The person can read it to Sam, or copy it. Stoop does not send it.

## Page states

Monday builds these. The copy is fixed now.

| State | What the page shows |
| --- | --- |
| Empty | Large type. No cards. One line: someone is coming to the house. Ask when. |
| After turn 1 | The visit card. |
| After turn 2 | The visit card and the checklist card, both still visible. |
| After turn 3 | The same cards. The reply is the forecast and the gate. |
| After turn 4 | The draft in the conversation, marked as not sent. |
| Unknown visit | The person asks about a plumber, or anything else not in the fixture. No card appears. The reply says the household list has the fridge repair, and no plumber. |

The protocol drawer is closed in every one of these states. Opening it shows the latest POST to `/mcp`: the JSON-RPC body, `MCP-Protocol-Version`, `Mcp-Session-Id`, and `Origin`. On turn 3 the body is `tools/call` for `weather_for_window` with empty arguments. That empty argument list is the point. The window came from the session.

## Video

Under 3 minutes. English. Public YouTube. No copyrighted music. Two takes on Tuesday. Use the take where turn 2 clearly follows turn 1, the weather reply quotes a real probability, and the draft says Sam.

| Seconds | Picture |
| --- | --- |
| 0–20 | The person at home, the door, the hour before a stranger arrives. Fridge. Side gate. |
| 20–100 | The four turns, in order, cards on screen. Drawer stays closed. |
| 100–140 | Open the drawer on the weather call. Then the terminal: Strands, Bedrock, the four tool names on one session. |
| 140–160 | This is the add-on. The visit, the prep, and the message to the other adult. Alexa+ already calls MCP servers. This one remembers the visit. |

## Checks the later days have to pass

Friday

- `initialize` response includes `Mcp-Session-Id`.
- The following `tools/list` sends `MCP-Protocol-Version: 2025-11-25` and that session id.
- `tools/call` of `find_visit` with `fridge` returns Priya, Thursday, 2 to 4pm.
- A request with `Origin: https://evil.example` gets HTTP 403.

Saturday

- `prep_checklist` on a new session fails with no visit.
- `find_visit` then `prep_checklist` on that same session returns the three items.
- `weather_for_window` hits Open-Meteo and returns probabilities for the resolved Thursday window.
- `draft_update` sets `to` to Sam and `sent` to false.
- Both cards are `ui://` resources referenced by `_meta.ui.resourceUri`.

Sunday

- One Strands agent, one Bedrock model, one MCP session.
- The four utterances in order.
- Printed trace includes `find_visit`, `prep_checklist`, `weather_for_window`, `draft_update`.
- The draft text contains Sam. The utterances do not.

Monday

- Empty state before any turn.
- Unknown visit leaves the cards unchanged and says there is no plumber.
- The drawer is closed when the page loads.
