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
6. Getting your approval before touching Jira
7. Creating the card + all subtasks in one go

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
AC1: [One verifiable outcome. One sentence. Use "is", "returns", "validates" — not "should" or "may". Compact and direct.]
AC2: [Same format]
AC3: [Same format]
[Add more ACs as scope warrants — each AC maps to one subtask later]

**Notes:**
[Leave blank — user fills in]

**Documentation URL:** [Confluence page URL]
```

**Writing good ACs:**
The goal is that any developer reading AC1 knows exactly what done looks like. Each AC should name a concrete, testable outcome. Reference team standards where relevant (e.g., "follows DBT staging model naming conventions per the DE standards doc"). Derive ACs only from what the user explicitly asked for — do not add ACs for things that are implied, adjacent, or "nice to check while you're at it."

**Common AC mistakes to avoid:**
- **Redundant ACs**: Don't add an AC that is just a restatement of another one or an obvious consequence of it.
- **Stretch ACs**: Don't invent scope that wasn't requested.
- **Process ACs**: Subtasks like "dev review" and "PO review" exist as subtasks — they are not ACs. Don't duplicate them as acceptance criteria.

When in doubt, ask yourself: "Did the user ask for this, or am I adding it on my own?" If you added it on your own, cut it.

If stakeholder info wasn't mentioned anywhere, put "TBD" — don't invent it.

---

## Step 7 — Show the user the draft

Send the full card text (formatted exactly as in Step 6) and say:

> "Here's the draft — does this look right? Let me know if you'd like to adjust anything: the description, ACs, stakeholder info, notes, or anything else. Once you're happy with it, I'll create it in Jira."

Make any changes the user requests. Keep iterating until they explicitly say it's good.

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
| Description | Full card content, formatted for Jira |

If the user chose the current sprint, move the newly created issue into the active sprint after creation using the sprint transition.

---

## Step 10 — Create subtasks

Create subtasks under the card in this order:

1. **One subtask per AC** — title: `Implement [brief AC summary]`
2. `Point this card`
3. `Dev review`
4. `PO review`
5. `Post prod validation`

So if there are 3 ACs, the card will have 7 subtasks total (3 AC subtasks + 4 process subtasks).

---

## Finish

Once everything is created, confirm to the user with a link to the Jira card. Something like:

> "Done! The card is live: [Jira issue link]. It has [N] subtasks including one for each AC plus the review and validation steps."

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

Show all drafted cards to the user at once, numbered, and ask:

> "Here are the [N] migration cards I drafted from the Miro board. Do any of these need adjustments before I create them?"

Iterate until the user approves all drafts.

---

## Migration Step 5 — Ask about a swap-over card

After the user approves the migration card drafts, ask:

> "Do you also want a **swap-over card** as the final card in this migration? This would cover cutting over to the new system — things like deactivating the legacy SSIS job, rescheduling or activating the new Dagster/DBT pipeline, setting up alerts and logging, and verifying end-to-end flow. Want one, and if so, is there anything specific you'd like in it beyond the standard swap-over checklist?"

If they say **yes**:
- Draft the swap-over card with ACs covering: deactivation of legacy SSIS, activation of new pipeline, alerting configured, logging verified, end-to-end data flow confirmed, rollback plan noted.
- Add this card as the last card in the set.

If they say **no**, skip it.

---

## Migration Step 6 — Data validation subtask (mandatory for every migration card)

Every migration card — including the swap-over card if created — gets one additional subtask beyond the standard set:

> `Validate data output and run comparisons against legacy system for this step`

**Full subtask order for every migration card:**
1. One subtask per AC — title: `Implement [brief AC summary]`
2. `Validate data output and run comparisons against legacy system for this step`
3. `Point this card`
4. `Dev review`
5. `PO review`
6. `Post prod validation`

---

## Migration Step 7 — Ask about placement

Ask once for the whole set:

> "Should all of these go on the **Lightspeed** board or the **DE Feature Squad** board? And should I add them to the **current sprint** or the **backlog**?"

---

## Migration Step 8 — Create all cards and subtasks

Create each migration card in Jira one by one (Stories), in order from first to last (swap-over card last if present). For each card:
- Create the Story, passing the epic key extracted in Migration Step 1 via `additional_fields: {"customfield_10014": "<epic key>"}`. Every migration card must be linked to the same epic as the discovery card.
- For the LS project, use issue type `Task` for subtasks (not `Sub-task`) — the LS project does not support the Sub-task issue type.
- Move to sprint if the user chose current sprint.
- Create all subtasks in the order defined in Migration Step 6.

After all cards are created, confirm with links to each.
