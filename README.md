# Case Manager Assignment — Prototype

A working prototype of the case manager assignment tool. Single HTML file, no server, no build step, no dependencies.

This is **Phase 0**: a proof of the assignment logic, not the production tool. See "What this is not" below before showing it to anyone.

---

## Run it

Open `index.html` in a browser. That's the whole setup.

To share it with others, either send them the file, or push this repo and turn on GitHub Pages (Settings → Pages → deploy from branch → `main` / root).

## Try it

1. Type your name in the sidebar.
2. **Assign a case** — enter any case reference, pick a language and state, and find a case manager.
3. Message nobody (this is a prototype), then start the timer.
4. Either accept, decline with a reason, or let the timer run out and watch the 48-hour suspension land.
5. Check **Do not assign**, then flip on **Supervisor mode** in the sidebar to clear someone early.
6. **Weekly import** — load `sample-data/sample-caseload-export.csv` to see the case-count refresh.
7. **Audit log** — every offer is recorded, and exports to CSV.

Tip for demos: the timer is a real three minutes. To shorten it, change `TIMER_SECONDS` near the top of the script block in `index.html`.

---

## Where the data lives

**Nowhere but your browser.** The CSV you load is read locally by the browser and is never uploaded, transmitted, or stored. Nothing in this repo contains real case data, and nothing should ever be committed that does.

The sample export in `sample-data/` is entirely fabricated — invented names, invented case numbers.

> **Do not commit a real SmartAdvocate export to this repository, public or private.** Load it through the Weekly import screen instead. That is what the screen is for.

State resets when you reload the page. That's deliberate for a prototype — see "Known gaps."

---

## What it implements

Rules are numbered to match the technical specification document.

| Rule | Behaviour |
|---|---|
| R1 | Eligibility: language and state must match; excludes anyone on leave or suspended |
| R2 | Ranked by active case count ascending, ties broken alphabetically |
| R3 | 180-second timer, started manually by the rep after they message the case manager |
| R4 | A timeout is treated exactly like a decline, penalty included |
| R5 | Decline or timeout writes a 48-hour suspension |
| R6 | Only supervisor mode can clear a suspension early |
| R7 | Leave is a separate status from suspension and carries no penalty |
| R8 | No eligible match returns an explicit error; filters are never widened |
| R10 | Every offer and outcome is recorded, with reason and handling rep |
| R11 | Case count increments on acceptance; the weekly import resets it |
| R12 | Someone who passed on a case isn't offered that same case again |

Escalation (pool exhausted) is detected and flagged in the audit log.

---

## What this is not

These are the honest gaps between the prototype and the specified tool. Worth stating out loud when demoing, so nobody assumes the hard parts are done.

- **Single user.** A static page has no shared server, so two reps running this have two separate, unaware copies. Real multi-rep use needs a backend. This is the single biggest gap and the largest piece of the real build.
- **No persistence.** Refreshing clears everything. Fine for a demo, wrong for daily use.
- **No real authentication.** "Supervisor mode" is an unprotected checkbox.
- **Client-side timers.** Closing the tab stops the countdown. The spec requires server-authoritative timers.
- **No SmartAdvocate API.** Case counts come from a manual CSV import, by design at this stage.
- **No Teams integration.** Messaging case managers stays a manual step, as specified. Supervisor escalation alerts are shown in-app only.
- **Roster is defined in code.** Adding and editing case managers through the interface isn't built yet; leave status can be toggled.

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
