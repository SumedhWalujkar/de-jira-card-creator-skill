# de-jira-card-creator-skill

A Claude Code skill for creating structured Jira cards for data engineering projects at Veterans United.

## What it does
- Creates well-researched Jira cards for the Lightspeed (LS) or DE Feature Squad (DENMFS) boards
- Automatically searches Confluence and GitHub for project context before drafting
- Handles migration card spin-ups: reads a discovery card, opens the Miro board, extracts the card split section, and generates one Jira card per migration step
- Every migration card gets a mandatory data validation subtask
- Optionally adds a final swap-over card

## Install
```
claude skill install <repo-url>
```
