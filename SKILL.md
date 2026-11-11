---
name: de-jira-card-creator
description: >
  Creates structured, well-researched Jira cards for data engineering projects on the Lightspeed (LS) or DE Feature Squad (DENMFS) Jira boards at Veterans United. Use this skill whenever a user wants to create a Jira ticket, story, task, or card for a data engineering project — even if they just say "make a Jira card", "create a ticket", "I need to track this work", "create a story for this", or describe a feature, pipeline, or data task without explicitly mentioning Jira. Also handles migration card spin-ups: when the user mentions a discovery card, a Miro board, SSIS migration, DBT/Dagster migration, or says "spin up migration cards", invoke this skill. Accepts any input: screenshots, Slack conversation snippets, rough descriptions, or detailed specs.
---

## What this skill does

Takes any description of work — however rough — and turns it into a complete, ready-to-create Jira card by:
1. Asking upfront whether this is a regular card or a migration card spin-up
2. Finding the right Confluence project automatically
3. Reading the project's GitHub code for context
4. Pulling in relevant team standards (Snowflake, DBT, Python, Lightspeed)
5. Drafting a card with stakeholder info, description, and tight acceptance criteria
6. Listing the process subtasks as a checklist on the card itself
7. Getting your approval before touching Jira
8. Creating the card — one Story, with no separate subtask issues

---

## Step 0 — Determine card type

Before doing anything else, ask:

> "Is this a **regular card** or a **migration card spin-up** (based on a discovery card with a Miro board)?"

- If **regular card** → proceed to Step 1 (regular path).
- If **migration card** → jump to the **Migration Path** section below.

---

## Step 1 — Understand what the user wants (regular path)

Read everything provided: text descriptions, screenshots (examine image content carefully), Slack threads, rough notes. Extract:
- What needs to be built, changed, or fixed
- Any project, system, or pipeline names mentioned
- Any stakeholder, team, or department hints
- Any SLA, deadline, or urgency signals

Don't ask the user for clarification yet — try to find the project yourself first.

---

## Step 2 — Find the Confluence project (before asking)

Search Confluence for data engineering projects matching the user's description. Use the Atlassian MCP's Confluence search.

Try searches like:
- The project/system name the user mentioned
- Key technical terms (e.g., "Snowflake ingestion", "DBT model", "data pipeline", "ETL")
- Stakeholder or team names if mentioned

Present **2–4 matching Confluence pages** to the user, each with a one-line description:

> "I found a few projects that might be related. Which one is this work for?"
> 1. **[Page title]** — [one sentence summary]
> 2. **[Page title]** — [one sentence summary]
> ...

If nothing matches after a genuine search, ask the user directly for the project name or Confluence URL.

---

## Step 3 — Confirm and read the project page

Once the user identifies the project, fetch the full Confluence page. From it, extract:
- **GitHub repository URL** — there should be a link to the repo
- **Documentation URL** — the Confluence page URL itself (save this for the card)
- Any stakeholder, department, or SLA information mentioned on the page

---

## Step 4 — Read the GitHub codebase

Fetch the project's GitHub repository (main branch, unless the user explicitly names a different branch — never deviate from main on your own).

Review:
- README and any architecture/design docs
- Key source files to understand the current data flow and patterns
- Existing naming conventions, file structure, and code style
- What's already implemented and what's clearly missing or incomplete

Use `WebFetch` to read GitHub content (e.g., `https://raw.githubusercontent.com/org/repo/main/README.md`, directory listings via the GitHub API). If the repo is private and inaccessible, note it and ask the user to describe the architecture or paste relevant snippets.

---

## Step 5 — Look up team standards

Search Confluence for the standards documentation relevant to this work. Look for pages covering:
- Lightspeed data engineering standards
- Python coding standards
- Snowflake conventions and patterns
- DBT model standards and naming rules
- Any other team-specific guidelines that apply

Use these standards when writing acceptance criteria — ACs should explicitly reference what's expected by the team, not just what needs to happen functionally.

---

## Step 6 — Draft the card

Generate the card using this exact structure:

```
**Primary Stakeholder:** [Name or "TBD"]
**Stakeholder Department:** [Department or "TBD"]
**SLA:** [Copy verbatim from the Confluence project page — e.g. "Tier 0", "Tier 1". Do not paraphrase or add urgency language of your own.]

**Description:**
[3–4 sentences, plain text. What does this project do, and what specifically needs to be implemented in this card? Be direct — no filler words. Ground this in what the user described plus what you found in the code and Confluence docs.]

**Acceptance Criteria:**
- [ ] AC1: [One verifiable outcome. One sentence. Use "is", "returns", "validates" — not "should" or "may". Compact and direct.]
- [ ] AC2: [Same format]
- [ ] AC3: [Same format]
[Add more ACs as scope warrants]

**Subtasks:**
- [ ] Point this card
- [ ] Dev review
- [ ] PO review
- [ ] Post prod validation

**Notes:**
[Leave blank — user fills in]

**Documentation URL:** [Confluence page URL]
```

**Checklists:** Both the Acceptance Criteria and the Subtasks sections are checklists (one unchecked box per item). In the draft shown to the user, write them as `- [ ]` items. In Jira they must be real checkboxes — see Step 9 for the required format.

**Subtasks section:** A checklist in the card description — the only place subtasks exist. **Never create subtask issues (or any separate cards) for these items.** Default items: `Point this card`, `Dev review`, `PO review`, `Post prod validation` (migration cards add the validation item from Migration Step 6). If the user edits the list, update the section; if they want no subtasks, remove the section entirely.

**Writing good ACs:**
The goal is that any developer reading AC1 knows exactly what done looks like. Each AC should name a concrete, testable outcome. Reference team standards where relevant (e.g., "follows DBT staging model naming conventions per the DE standards doc"). Derive ACs only from what the user explicitly asked for — do not add ACs for things that are implied, adjacent, or "nice to check while you're at it."

**Common AC mistakes to avoid:**
- **Redundant ACs**: Don't add an AC that is just a restatement of another one or an obvious consequence of it.
- **Stretch ACs**: Don't invent scope that wasn't requested.
- **Process ACs**: Steps like "dev review" and "PO review" belong in the Subtasks checklist, not the ACs. Don't duplicate them as acceptance criteria — even if the user removes them from the checklist.

When in doubt, ask yourself: "Did the user ask for this, or am I adding it on my own?" If you added it on your own, cut it.

**ACs stay on the card.** ACs live only in the card's Acceptance Criteria section. Never turn ACs into subtasks, checklist items, or separate cards, and never propose an `Implement [AC]` item per AC.

If stakeholder info wasn't mentioned anywhere, put "TBD" — don't invent it.

---

## Step 7 — Show the user the draft

Send the full card text (formatted exactly as in Step 6), including the **Subtasks** checklist. Then say:

> "Here's the draft — does this look right? Let me know if you'd like to adjust anything: the description, ACs, stakeholder info, the Subtasks checklist, notes, or anything else. Once you're happy with it, I'll create it in Jira."

Make any changes the user requests and keep iterating until the user explicitly approves the card. The Subtasks checklist is part of the card, so approving the card approves it — no separate subtask approval is needed, because no subtask issues are created.

---

## Step 8 — Ask about placement

Once the user approves the card, ask two questions (can be in one message):

1. **Which board?** "Should this go on the **Lightspeed** board or the **DE Feature Squad** board?"
2. **Sprint or backlog?** "And should I add it to the **current sprint** or the **backlog**?"

---

## Step 9 — Create the Jira card

Use the Atlassian MCP to create the issue:

| Field | Value |
|-------|-------|
| Project | `LS` (Lightspeed) or `DENMFS` (DE Feature Squad) |
| Issue type | Story |
| Summary | A concise title derived from the description (not the full description text) |
| Description | Full card content as approved, in HTML (see below) |

If the user chose the current sprint, move the newly created issue into the active sprint after creation using the sprint transition.

**Description format — required for the checklists:** Pass the description with `contentFormat: "html"`. Markdown `- [ ]` items do not become checkboxes in Jira; they're stored as literal "[ ]" text. Write each checklist (Acceptance Criteria and Subtasks) as a Jira task list:

```html
<p><strong>Acceptance Criteria:</strong></p>
<ul data-type="task-list">
  <li data-type="task-item"><input type="checkbox"> AC1: ...</li>
  <li data-type="task-item"><input type="checkbox"> AC2: ...</li>
</ul>
<p><strong>Subtasks:</strong></p>
<ul data-type="task-list">
  <li data-type="task-item"><input type="checkbox"> Point this card</li>
  <li data-type="task-item"><input type="checkbox"> Dev review</li>
</ul>
```

Use `<p>`, `<strong>`, `<br>`, `<code>` and `<a href>` for the rest of the card. After creating the card, check that the description reads back with `- [ ]` items (unescaped). If it shows `\[ \]`, the checklists didn't convert — rewrite the description in the HTML format above with `editJiraIssue` and `contentFormat: "html"`.

---

## Step 10 — No subtask issues

Do **not** create subtask issues (`Task`/`Sub-task` with a parent) or any other separate cards for the subtasks or ACs. The card's Subtasks checklist is the complete record of them. Only create a separate issue if the user explicitly asks for that specific issue in the current request.

---

## Finish

Once the card is created, confirm to the user with a link to the Jira card. Something like:

> "Done! The card is live: [Jira issue link]. The ACs and Subtasks are checklists on the card."

---

---

# Migration Path

Use this path when the user says this is a migration card spin-up.

## Migration Step 1 — Get the discovery card

Ask the user for the discovery Jira card key (e.g., `DENMFS-1234`). Fetch it using the Atlassian MCP (`getJiraIssue`).

From the fetched issue, extract and save:
- **Epic** — the epic linked to the discovery card (check `fields.epic`, `fields.customfield_10014`, or `fields.parent` depending on your Jira setup). Save the epic key (e.g., `LS-1234`). Every migration card you create will be linked to this same epic.
- **Miro board URL** — scan both the **description** and all **comments** for a URL matching `https://miro.com/app/board/...`.

If no Miro URL is found, ask the user to paste it.
If no epic is found on the discovery card, note it and proceed — you'll skip epic assignment.

---

## Migration Step 2 — Read the Miro board

Use the Miro MCP tools (`mcp__Miro__context_explore` and `mcp__Miro__context_get`) to read the board — this is more reliable than the Chrome extension.

Look for a section labeled something like **"Card Split"**, **"Card Division"**, **"Card Breakdown"**, or similar. For each card entry in that section, extract:
- Card title / name
- What it covers (the migration step, the data it handles, the transformation logic, etc.)
- Any notes or constraints listed

If the Miro MCP is not available, fall back to the Claude in Chrome browser extension (`mcp__claude-in-chrome__*` tools). If neither is connected, ask the user to paste or describe the contents of the card split section directly.

---

## Migration Step 3 — Pull project context

From the discovery card, identify the project name. Run the same Confluence search as the regular path (Step 2 above) to find the project page, GitHub repo, stakeholders, SLA, and team standards. This context applies to all migration cards you are about to draft.

---

## Migration Step 4 — Draft all migration cards

For each card identified in the Miro card split section, draft a full card using the standard template from Step 6 above.

**Migration-specific guidance for descriptions:**
Each description should make clear:
- What legacy SSIS component or logic this card is migrating
- What the new implementation will be (DBT model, Dagster job, etc.)
- What data or pipeline step this card is responsible for

**Migration-specific guidance for ACs:**
Keep ACs scoped to what this card's migration step actually delivers. Do not add ACs that belong to adjacent cards.

Show all drafted cards to the user at once, numbered. Each card includes its **Subtasks** checklist (the default set from Migration Step 6). Then ask:

> "Here are the [N] migration cards I drafted from the Miro board. Do any of them — including their Subtasks checklists — need adjustments before I create them?"

Iterate until the user approves all drafts. The Subtasks checklist is part of each card; no subtask issues are created.

---

## Migration Step 5 — Ask about a swap-over card

After the user approves the migration card drafts, ask:

> "Do you also want a **swap-over card** as the final card in this migration? This would cover cutting over to the new system — things like deactivating the legacy SSIS job, rescheduling or activating the new Dagster/DBT pipeline, setting up alerts and logging, and verifying end-to-end flow. Want one, and if so, is there anything specific you'd like in it beyond the standard swap-over checklist?"

If they say **yes**:
- Draft the swap-over card with ACs covering: deactivation of legacy SSIS, activation of new pipeline, alerting configured, logging verified, end-to-end data flow confirmed, rollback plan noted.
- Show it with its Subtasks checklist (Migration Step 6) and get explicit approval, as in Migration Step 4.
- Add this card as the last card in the set.

If they say **no**, skip it.

---

## Migration Step 6 — Subtasks checklist for migration cards

Every migration card — including the swap-over card if created — gets one additional item in its Subtasks checklist beyond the standard set:

> `Validate data output and run comparisons against legacy system for this step`

Drop this item on cards where it doesn't fit (e.g., a decommission card). The user's approved checklist is final.

**Default Subtasks checklist for every migration card:**
1. `Validate data output and run comparisons against legacy system for this step`
2. `Point this card`
3. `Dev review`
4. `PO review`
5. `Post prod validation`

As on the regular path, these exist only as a checklist in the card description — never as subtask issues — and the description is created in HTML per Step 9 so both checklists render as real checkboxes.

---

## Migration Step 7 — Ask about placement

Ask once for the whole set:

> "Should all of these go on the **Lightspeed** board or the **DE Feature Squad** board? And should I add them to the **current sprint** or the **backlog**?"

---

## Migration Step 8 — Create all cards

Create each migration card in Jira one by one (Stories), in order from first to last (swap-over card last if present). For each card:
- Create the Story with the epic as its parent (`parent: "<epic key>"`). Every migration card must be linked to the same epic as the discovery card.
- Move to sprint if the user chose current sprint.
- Do **not** create subtask issues — the Subtasks checklist in the description covers them.
- If the user stops or interrupts creation, stop immediately, check what was actually created, and report it.

After all cards are created, confirm with a link to each.
