# Replication prompt — set up the Client Support Analysis Agent for your product

For product teams (Managerium, APON, PrimeVAT, …) that want the same agent PeopleDesk uses: it reads your pending Project Tracker support tickets oldest-first, investigates each one across **all** of your repositories, stores a Bangla-first bilingual report on the ticket, posts a Google Chat card with the Project Tracker link, and — for a **confirmed** bug — opens a fix pull request for your developers to review.

**How to use:** open Claude Code at the root of **your** workspace (the folder that holds all of your product's repositories), paste the block below, and fill in the `<…>` values. Claude asks for anything missing, verifies every fact, and stops for your approval before anything that writes to a database or a repository.

Background (Bangla): [sharing-guide.md](sharing-guide.md) · design: [README.md](README.md)

> **Credentials are yours, and they never go into the chat.** Each team member enters their **own** database login and their **team's own** Google Chat webhook with `peopledesk-support-analyze --profile <id> setup`, typed into their own terminal (the password is hidden). The tool stores them only under `profiles.<id>` in a private file on that machine, refuses a login or webhook that already belongs to another product (e.g. PeopleDesk), and refuses any login left at the shared top level. Never reuse PeopleDesk's database login, webhook or tokens.

---

```text
Set up the Client Support Analysis Agent for our product by REUSING the shared
implementation. Do not build a new one and do not fork it.

== Our values — NOT secrets (ask me for anything left as <…>; never guess) ==
Product name ............ <e.g. Managerium ERP>
Profile id .............. <lowercase, e.g. managerium>  -> becomes /support:<id>
Kit ..................... <path or git URL of peopledesk-business-ai>; the kit is scripts/support-analyst/
Workspace root .......... <absolute path of THIS workspace>
Project Tracker ticket URL with {ticketId} ... <e.g. https://pt.ibos.io/support/tickets/view/{ticketId}>
Code repositories where confirmed-bug fix PRs may be opened:
  <folder name> = <github org/repo>, base branch <e.g. main>   (one line each)
Secrets (database login, Google Chat webhook): I enter them MYSELF in step 3 with
the `setup` command in my own terminal. Do NOT ask me for them in this chat.

== Credential rules (non-negotiable) ==
- NEVER ask me to type, paste or confirm a password, login or webhook URL in this chat,
  and never accept one if I paste it — tell me to rotate it and use `setup` instead.
- Credentials are entered only with `peopledesk-support-analyze --profile <id> setup`,
  which I run in my own terminal (outside this chat). It stores them only under
  "profiles.<id>" in ~/.config/peopledesk-support-analyst/config.json (chmod 600) and
  refuses a login or webhook that already belongs to another profile.
- Never read, copy, print, reuse or look up another team's credentials (e.g. the
  "peopledesk" profile), even if they exist on this machine. Never edit the config
  file by hand to add or change credentials.
- Secrets never go into a repository, a commit, a PR, a log or a message.
- GitHub access is my own `gh auth` login; check it with `gh auth status`.

== Read first, in full ==
1. <kit>/scripts/support-analyst/README.md
2. <kit>/scripts/support-analyst/golden-reference.md
3. engineering-blueprint/support-analysis-agent/sharing-guide.md (if available)

== Steps — report each result and wait where it says STOP ==

1. Install the kit.
   PDSA_WORKSPACE_ROOT=<workspace root> bash <kit>/scripts/support-analyst/install.sh
   Make sure ~/.local/bin is on PATH. Run `npm test` in the kit and report the result.

2. Discovery (read-only).
   - List every repository folder in this workspace and what it contains: backend,
     frontend, reporting, scheduler, real-time, knowledge base, SQA, other.
   - For each code repository: `git remote get-url origin`, the default branch, and
     whether it has Git hooks (.husky/) and a CLAUDE.md.

3. Credentials — STOP and hand over to me. Tell me exactly this, then wait:
     "Open a separate terminal (not this chat) and run:
        peopledesk-support-analyze --profile <id> setup
      Enter: SQL Server host, port, database (VPT), YOUR SQL login and password
      (hidden), YOUR team's Google Chat webhook (optional), and the workspace root.
      Reply 'done' here when finished — do not paste any of the values."
   After I reply, run `peopledesk-support-analyze --profile <id> doctor` (read-only).
   It must show credentialSource "profiles.<id>" — if it shows anything else, or
   CONFIG_MISSING, stop and tell me. Then find our product row in its output:
   intProductId, strCode, IsActive. Report them; do not guess.

4. Profile. Copy <kit>/scripts/support-analyst/profiles/_template.json to
   profiles/<id>.json and fill it from discovery:
   - productName, productCode, productId exactly as `doctor` shows them;
   - viewName vw<Product>PendingSupportTicketAIAnalysis;
   - commandName support:<id>; knowledgeBase (path or null — tell me if null and
     recommend building one);
   - trackerTicketUrl with {ticketId};
   - repositories: EVERY repository that defines, executes, reports, schedules,
     pushes, documents or tests product behaviour, each with a precise "role".
     Every listed repository is investigated for every ticket, so leave none out.
     Add "github" and "baseBranch" ONLY to the code repositories I listed for fix PRs.
   Validate: `peopledesk-support-analyze profiles` lists it, and
   `peopledesk-support-analyze --profile <id> command-md --workspace <workspace root>`
   renders without errors. Run `npm test` again.

5. View. Print the script with `peopledesk-support-analyze --profile <id> view-sql`,
   show it to me, and STOP. A human/DBA reviews and runs it. Never run DDL yourself.
   After I confirm it ran, verify read-only: pending count, exactly one product code,
   the oldest ticket.

6. Backfill (optional). If I want old tickets excluded, prepare it from
   <kit>/scripts/support-analyst/sql/03-backfill-mark-read-before-2026-10-18.sql for
   OUR product code and MY cutoff date: preview counts first, a dated backup table,
   counts verified inside one transaction, and the rollback script. Show me the
   preview and STOP for my approval. Only IsReadFromPlatform may change.

7. Fix-PR policy for our code repositories. Confirmed bugs get a fix PR that skips
   Git hooks (--no-verify) and is never merged by the agent. If any of our
   repositories' CLAUDE.md or testing policy forbids --no-verify, show me the exact
   rule and propose a one-line written exception limited to support-agent fix PRs
   (regression test still required, never auto-merged). Apply it only after I approve.

8. Install the command. Re-run install.sh so /support:<id> is generated; tell me to
   restart Claude Code.

9. First run — dry run. Run `/support:<id> --limit 1 --dry-run`. Open the preview
   HTML and the chat JSON and check them against the golden reference quality bar:
   every repository covered, evidence real, Project Tracker link correct.
   Report what is good and what is weak. Then STOP.

10. First live run — only when I say so: `/support:<id> --limit 1`. Verify read-only
    that exactly one ticket row changed and only strAIFeadbackWithHTML,
    strAIFeadback and IsReadFromPlatform were written, and that the Google Chat card
    arrived in OUR space.

== Rules for every run of the agent ==
- Analyse every repository in the profile for every ticket; the CLI rejects
  analyses that skip one or cite files/lines that do not exist. Never weaken this.
- Only the CLI writes the database, only through `save`, only the three columns.
- Confirmed bug (rootCauseStatus CONFIRMED + technicalChangeType BUG_FIX) ->
  fix-start / fix-open: separate worktree, ai-fix/<ticket> branch, minimal fix plus a
  regression test, commit and push with --no-verify, PR opened, NEVER merged. The PR
  link goes into the report and the chat card. Anything less than CONFIRMED gets no PR.
- Never push to a base branch, never merge, never deploy.
- If something in the kit does not fit our product, propose a change to the shared
  kit (with tests) instead of forking it.
```
