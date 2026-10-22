# Replication prompt — set up the Support Analysis Agent for another product

Paste the block below into Claude Code, opened at the root of your team's workspace (the folder that holds all of your product's repositories). Replace the three `<…>` values first. Claude does the verification and setup and stops for approval wherever a human decision or a database write is involved.

Background (Bangla): [sharing-guide.md](sharing-guide.md) · design: [README.md](README.md)

---

```text
Set up the Client Support Analysis Agent for our product, reusing the existing
implementation. Do not build a new one.

Product name: <e.g. Managerium ERP>
Profile id:   <e.g. managerium>   (lowercase; becomes the /support:<id> command)
Kit location: <path to peopledesk-business-ai/scripts/support-analyst, or the repo URL to clone>

Read first, in full:
  1. <kit>/README.md
  2. <kit>/golden-reference.md
  3. engineering-blueprint/support-analysis-agent/sharing-guide.md (if available)

Then, step by step — report each result before moving on:

1. Discovery (read-only).
   - List every repository folder in this workspace and what each contains
     (backend, frontend, reporting, scheduler, real-time, knowledge base, SQA, other).
   - Run `peopledesk-support-analyze doctor` (install the kit first with
     `PDSA_WORKSPACE_ROOT=<this workspace> bash <kit>/install.sh` if the command is missing)
     and find our product's row: intProductId, strCode, IsActive. Do not guess them.

2. Profile.
   - Copy <kit>/profiles/_template.json to <kit>/profiles/<id>.json and fill it from
     the discovery results. `repositories` must list EVERY repository of this
     workspace that defines, executes, reports, schedules, pushes, documents or
     tests product behaviour, each with a precise `role`. Every listed repository
     becomes mandatory for every ticket, so do not leave one out.
   - Validate: `peopledesk-support-analyze profiles` lists it and
     `peopledesk-support-analyze --profile <id> command-md --workspace <this workspace>`
     renders without errors.
   - If this product has no knowledge base, set knowledgeBase to null and tell me;
     recommend building one.

3. View. Generate the script with
   `peopledesk-support-analyze --profile <id> view-sql`, show it to me, and STOP.
   A human/DBA reviews and runs it. Never run DDL yourself.

4. Backfill (optional). If I want old tickets excluded, prepare the backfill from
   <kit>/sql/03-backfill-mark-read-before-2026-10-18.sql for our product code and my
   cutoff date: preview counts first, backup table, verified counts, rollback script.
   Show me the preview and STOP for my approval.

5. Local configuration. In ~/.config/peopledesk-support-analyst/config.json add a
   "profiles.<id>" section with our workspaceRoot (if different) and our Google Chat
   webhook if I give you one. Never write secrets into any repository, never print them.

6. Install. Re-run install.sh so /support:<id> is generated. Tell me to restart
   Claude Code.

7. First run. Run `/support:<id> --limit 1 --dry-run`, open the preview, and check it
   against the golden reference quality bar. Report what is good and what is weak.

Rules throughout:
- The agent analyses and recommends only; it never changes product source code,
  commits, pushes or deploys.
- Only the CLI writes to the database, only through `save`, only the three columns
  strAIFeadbackWithHTML, strAIFeadback, IsReadFromPlatform.
- Every repository in the profile is investigated for every ticket; `save` rejects
  analyses that skip one or cite files/lines that do not exist. Do not weaken this.
- If something in the kit does not fit our product, propose a change to the shared
  kit (with tests) instead of forking it.
```
