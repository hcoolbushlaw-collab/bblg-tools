# Case Manager Assignment — Prototype

A working prototype of the case manager assignment tool. Single HTML file, no server, no build step, no dependencies.

This is **Phase 0**: a proof of the assignment logic, not the production tool. See "What this is not" below before showing it to anyone.

---

## Run it

Open `index.html` in a browser. That's the whole setup.

To share it with others, either send them the file, or push this repo and turn on GitHub Pages (Settings → Pages → deploy from branch → `main` / root).

## Try it

1. Type your name in the sidebar.
2. **Assign a Case Manager** — enter a case reference, pick a case type, language, and state, and find a case manager.
3. Message nobody (this is a prototype), then start the timer.
4. Either accept, decline with a reason, or let the timer run out and watch the 48-hour suspension land.
5. Check **Do not assign**, then flip on **Supervisor mode** in the sidebar to clear someone early. Supervisor mode also reveals the roster, roster loading, weekly import, and audit log — those four screens are hidden from CS reps.
6. **Load the roster** — download the blank template, or load `sample-data/sample-caseload-export.csv` on the Weekly import screen to see the case-count refresh.
7. **Assign an Attorney** — pick Texas to see the ranked bench, or Florida to see a single-attorney state.
8. **Audit log** — every offer and assignment is recorded, filterable by role, and exports to CSV.

Tip for demos: the timer is a real two minutes. To shorten it, change `TIMER_SECONDS` near the top of the script block in `index.html`.

---

## Where the data lives

**Nowhere but your browser.** The CSV you load is read locally by the browser and is never uploaded, transmitted, or stored. Nothing in this repo contains real case data, and nothing should ever be committed that does.

The sample export in `sample-data/` is entirely fabricated — invented names, invented case numbers.

> **Do not commit a real SmartAdvocate export to this repository, public or private.** Load it through the Weekly import screen instead. That is what the screen is for.

State resets when you reload the page. That's deliberate for a prototype — see "Known gaps."

---

## Putting your real roster in

Real names never go in this repository. Load them from a file instead.

**Build a CSV in Excel** with these columns, one case manager per row. Separate multiple languages or states with semicolons:

| name | teams_handle | case_types | languages | states | active_cases |
|---|---|---|---|---|---|
| Alvarez, Marisol | @malvarez | MVA;PL | English;Spanish | CA;NV;AZ | 38 |
| Boone, Dana | @dboone | MVA;CVA | English | TX;NM | 44 |

Then go to **Load the roster** and select the file. There's a "Download a blank template" button on that screen to start from.

Case types are MVA, CVA, PL, and GVA. Column order doesn't matter and the header wording is flexible — "Full Name", "Teams Handle", "Case Type", "Language", "State" all work. `name`, `case_types`, `languages`, and `states` are all required.

**Write names exactly as your SmartAdvocate export writes them.** That's what lets the weekly import match rows to people automatically.

**Saving between sessions:** the roster clears on refresh. Use "Save current roster as .json" to keep it, then reload that file next time. Keep that file on your machine — `.gitignore` blocks `roster.json` and `roster*.csv` from being committed, but the safest habit is storing it outside the repo folder entirely.

## Three ways someone leaves the queue

| | Trigger | Duration | Penalty? |
|---|---|---|---|
| **Penalty** | Declined, or the timer ran out | 48 hours | Yes |
| **Cool-off** | Took a case, or marked out of office | Until end of day | No |
| **On leave** | Set by a supervisor | Until cleared | No |

The cool-off exists so nobody picks up two cases in one day.

**One thing to confirm with stakeholders:** the request said "the rest of that day (for 24 hours)," which are different rules. Taking a case at 4pm means free at midnight under one reading, and free at 4pm tomorrow under the other. This is built as **end of day**, since the stated goal is one case per day. To switch it, change `COOLOFF_MODE` near the top of the script in `index.html` from `'endOfDay'` to `'rolling24'`.

## Attorneys

A second roster with the same fields as case managers, and its own assignment screen.

**Attorney assignment is deliberately simpler than the case manager flow.** No timer, no reason codes, no cool-off, no penalty. State is the deciding filter, then lowest active caseload, ties broken alphabetically.

Texas isn't special-cased. The same rule covers both situations: in a state with one attorney it returns that attorney, and in Texas it ranks the deeper pool. One code path, nothing to keep in sync.

"Show next attorney" walks down the ranked list when the recommended one isn't right. Skipping records nothing against anyone — it isn't a decline.

Attorney case counts come from the weekly import. The importer defaults to **column N** for the attorney name and can be remapped if the export layout shifts.

## What it implements

Rules are numbered to match the technical specification document.

| Rule | Behaviour |
|---|---|
| R1 | Eligibility: case type, language, and state must all match; excludes anyone on leave or suspended |
| R2 | Ranked by active case count ascending, ties broken alphabetically |
| R3 | 120-second timer, started manually by the rep after they message the case manager |
| R4 | A timeout is treated exactly like a decline, penalty included |
| R5 | Decline or timeout writes a 48-hour penalty hold |
| R5a | Accepting a case starts a cool-off until end of day — one case per person per day |
| R5b | Out of office is its own button, not a decline reason. Cool-off until tomorrow, no penalty |
| R6 | Only supervisor mode can clear a hold early |
| R7 | Leave is a separate status from any hold and carries no penalty |
| R8 | No eligible match returns an explicit error; filters are never widened |
| R10 | Every offer and outcome is recorded, with reason and handling rep |
| R11 | Case count increments on acceptance; the weekly import resets it |
| R12 | Someone who passed on a case isn't offered that same case again |
| A1 | Attorneys: filter by state and case type, rank by lowest caseload, ties alphabetical |
| A2 | Attorney assignment carries no timer, no reason code, no cool-off, and no penalty |
| A3 | Skipping an attorney is not recorded against them |

Escalation (pool exhausted) is detected and flagged in the audit log.

---

## What this is not

These are the honest gaps between the prototype and the specified tool. Worth stating out loud when demoing, so nobody assumes the hard parts are done.

- **Single user.** A static page has no shared server, so two reps running this have two separate, unaware copies. Real multi-rep use needs a backend. This is the single biggest gap and the largest piece of the real build.
- **No persistence.** Refreshing clears everything. Fine for a demo, wrong for daily use.
- **No real authentication.** "Supervisor mode" is an unprotected checkbox. It hides screens rather than securing them — anyone can tick it.
- **Client-side timers.** Closing the tab stops the countdown. The spec requires server-authoritative timers.
- **No SmartAdvocate API.** Case counts come from a manual CSV import, by design at this stage.
- **No Teams integration.** Messaging case managers stays a manual step, as specified. Supervisor escalation alerts are shown in-app only.
- **Rosters are not persisted.** You can add, edit, and load people in the interface, but it clears on refresh. Save to a file and reload next session.

---

## Using your real export

The import screen doesn't assume a column layout — it reads the headers from whatever file you give it and lets you pick which column holds the case manager and which holds the status. It then counts every row whose status doesn't contain one of your "not active" words.

Two things to check on first use:

- **The not-active word list.** It defaults to `drop, closed, settled, resolved, referred out`. Adjust it to match the status values your firm actually uses.
- **Unmatched names.** If the export writes names differently than the roster does, they'll show up in an unmatched warning rather than failing silently. The matcher handles `Last, First` and `First Last` automatically.

---

## Files

```
index.html                              the whole application
sample-data/sample-caseload-export.csv  fabricated sample data — safe to commit
README.md                               this file
```

---

## Next steps

1. Demo it, and let the rules get argued with. Cheaper to change now than after a developer builds them.
2. Confirm the real export's column headers and status values.
3. Decide on the backend approach for multi-user, which is the point where this stops being a prototype.
