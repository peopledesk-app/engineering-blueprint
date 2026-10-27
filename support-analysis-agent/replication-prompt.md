# Replication prompt — set up the Client Support Analysis Agent

Two ways to use this page:

- **A PeopleDesk team member on a new machine** — use [Part A](#part-a--a-peopledesk-team-member-joins). The commands are already in the repositories; you only install the CLI and enter your own credentials.
- **Another product team** (Managerium, APON, PrimeVAT, …) that wants the same agent — use [Part B](#part-b--another-product-adopts-the-agent). The agent reads your pending Project Tracker support tickets oldest-first, investigates each one across **all** of your repositories, stores a Bangla-first bilingual report on the ticket, posts a Google Chat card with the Project Tracker link (and later updates into the same thread), and — for a **confirmed** bug — opens a fix pull request for your developers to review.

**How to use either part:** open Claude Code at the **workspace root** (the folder that holds all of the product's repositories *and* `peopledesk-kb/`), paste the block, and fill in the `<…>` values. Claude asks for anything missing, verifies every fact, and stops for your approval before anything that writes to a database or a repository.

Background (Bangla): [sharing-guide.md](sharing-guide.md) · design: [README.md](README.md)

> **Credentials are yours, and they never go into the chat.** Each team member enters their **own** database login and their **team's own** Google Chat webhook with `peopledesk-support-analyze --profile <id> setup`, typed into their own terminal (the password is hidden). The tool stores them only under `profiles.<id>` in a private file on that machine, refuses a login or webhook that already belongs to another product (e.g. PeopleDesk), and refuses any login left at the shared top level. Never reuse PeopleDesk's database login, webhook or tokens.

## How the commands are shared

The `/support:<id>` and `/support:<id>-prod-debug` commands are **committed** in `peopledesk-kb/scripts/support-analyst/commands/support/`, rendered from one template for every profile with workspace-relative paths only. The installers copy them into the workspace root's `.claude/commands/support/`, and `./update-repos` refreshes that copy after every pull — so everyone runs the same, reviewed version. Nobody edits the copy: change the template or a profile, run `sync-commands`, and commit both (a test fails while the committed commands are stale).

---

## Part A — a PeopleDesk team member joins

```text
Set up the PeopleDesk Client Support Analysis Agent on this machine. The kit and the
/support:peopledesk commands are already committed; only install and verify.

== Credential rules (non-negotiable) ==
- NEVER ask me to type, paste or confirm a password, login or webhook URL in this chat,
  and never accept one if I paste it — tell me to rotate it and use `setup` instead.
- Credentials are entered only with `peopledesk-support-analyze --profile peopledesk setup`,
  which I run in my own terminal (outside this chat).

== Steps — report each result ==
1. Check the workspace: this folder holds peopledesk-kb/ and the product repositories
   listed in peopledesk-kb/scripts/support-analyst/profiles/peopledesk.json. List any
   that are missing (./update-repos clones them).
2. Install: `bash peopledesk-kb/scripts/mac-linux/install-workspace-commands.sh`, then
   `bash peopledesk-kb/scripts/support-analyst/install.sh`. On Windows use
   `powershell -ExecutionPolicy Bypass -File peopledesk-kb\scripts\windows\install-workspace-commands.ps1`
   and `... -File peopledesk-kb\scripts\support-analyst\install.ps1`. Report the commands it
   installed. If ~/.local/bin is not on PATH, tell me the exact line (or Windows setting) to
   add (do not change it yourself).
3. Run `npm test` in peopledesk-kb/scripts/support-analyst and report the result.
4. Credentials — STOP and tell me: "Open a separate terminal (not this chat) and run:
     peopledesk-support-analyze --profile peopledesk setup
   Reply 'done' here when finished — do not paste any of the values."
   Then run `peopledesk-support-analyze --profile peopledesk doctor` (read-only): it must
   show credentialSource "profiles.peopledesk". Check `gh auth status`.
5. Tell me to reload VS Code (Developer: Reload Window) at the workspace root and confirm
   that /support:peopledesk and /support:peopledesk-prod-debug appear when I type "/".
6. First run — only a preview: `/support:peopledesk 1 dry`. Show me what it found.
```

---

## Part B — another product adopts the agent

```text
Set up the Client Support Analysis Agent for our product by REUSING the shared
implementation in peopledesk-kb/scripts/support-analyst. Do not build a new one and
do not fork it.

== Our values — NOT secrets (ask me for anything left as <…>; never guess) ==
Product name ............ <e.g. Managerium ERP>
Profile id .............. <lowercase, e.g. managerium>  -> becomes /support:<id>
Workspace root .......... <absolute path of THIS workspace>; peopledesk-kb must be cloned
                          directly inside it (the commands use paths relative to it)
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
1. peopledesk-kb/scripts/support-analyst/README.md
2. peopledesk-kb/scripts/support-analyst/golden-reference.md
3. peopledesk-blueprint/support-analysis-agent/sharing-guide.md (if available)

== Steps — report each result and wait where it says STOP ==

1. Install the kit.
   bash peopledesk-kb/scripts/support-analyst/install.sh
   (Windows: powershell -ExecutionPolicy Bypass -File peopledesk-kb\scripts\support-analyst\install.ps1)
   (it installs the CLI dependencies, links peopledesk-support-analyze into ~/.local/bin
   and installs the shared commands into this workspace's .claude/commands/support/).
   If ~/.local/bin is not on PATH, tell me the line to add; until then use
   `node peopledesk-kb/scripts/support-analyst/bin/peopledesk-support-analyze.js`.
   Run `npm test` in the kit and report the result.

2. Discovery (read-only).
   - List every repository folder in this workspace and what it contains: backend,
     frontend, reporting, scheduler, real-time, knowledge base, SQA, other.
   - For each code repository: `git remote get-url origin`, the default branch, whether
     it has Git hooks (.husky/) and a CLAUDE.md, and its own build / lint / test
     commands (from its README, package.json, solution files or run-tests scripts).
   - Run each candidate build / lint / test command ONCE on a clean checkout of the base
     branch and report pass/fail. A step that fails before any change (for example a lint
     tool with no configuration) must not go into the profile as-is: show me the failure
     and propose either the working command or leaving that step out, and wait for my
     decision. A verify step that can never pass blocks every fix PR.

3. Credentials — STOP and hand over to me. Tell me exactly this, then wait:
     "Open a separate terminal (not this chat) and run:
        peopledesk-support-analyze --profile <id> setup
      Enter: SQL Server host, port, database (VPT), YOUR SQL login and password
      (hidden), YOUR team's Google Chat webhook (optional), the workspace root,
      optionally the dev database as <appsettings file>#<ConnectionStrings key>
      (read at run time, never copied) and optionally YOUR team's PRODUCTION
      read-only connection string (hidden; used only by /support:<id>-prod-debug).
      Reply 'done' here when finished — do not paste any of the values."
   After I reply, run `peopledesk-support-analyze --profile <id> doctor` (read-only).
   It must show credentialSource "profiles.<id>" — if it shows anything else, or
   CONFIG_MISSING, stop and tell me. Then find our product row in its output:
   intProductId, strCode, IsActive. Report them; do not guess.

4. Profile. Copy peopledesk-kb/scripts/support-analyst/profiles/_template.json to
   profiles/<id>.json and fill it from discovery:
   - productName, productCode, productId exactly as `doctor` shows them;
   - viewName vw<Product>PendingSupportTicketAIAnalysis;
   - commandName support:<id>; knowledgeBase (workspace-relative path, or null — tell me
     if null and recommend building one);
   - trackerTicketUrl with {ticketId};
   - repositories: EVERY repository that defines, executes, reports, schedules,
     pushes, documents or tests product behaviour, each with a precise "role".
     Every listed repository is investigated for every ticket, so leave none out.
     Add "github", "baseBranch" and "verify" ONLY to the code repositories I listed for
     fix PRs, using only the verify commands I approved in step 2.
   Then generate the shared commands and check everything:
     peopledesk-support-analyze profiles                 # lists <id>
     node peopledesk-kb/scripts/support-analyst/bin/peopledesk-support-analyze.js sync-commands
     npm test                                            # in the kit, incl. the drift check
   `sync-commands` writes commands/support/<id>.md and <id>-prod-debug.md. Show me the
   rendered /support:<id> command and confirm it contains no absolute path.

5. View. Print the script with `peopledesk-support-analyze --profile <id> view-sql`,
   show it to me, and STOP. A human/DBA reviews and runs it. Never run DDL yourself.
   After I confirm it ran, verify read-only: pending count, exactly one product code,
   the oldest ticket.

6. Backfill (optional). If I want old tickets excluded, prepare it from
   peopledesk-kb/scripts/support-analyst/sql/03-backfill-mark-read-before-2026-10-18.sql for
   OUR product code and MY cutoff date: preview counts first, a dated backup table,
   counts verified inside one transaction, and the rollback script. Show me the
   preview and STOP for my approval. Only IsReadFromPlatform may change.

7. Fix-PR policy for our code repositories. Confirmed bugs get a fix PR that skips
   Git hooks (--no-verify) and is never merged by the agent. If any of our
   repositories' CLAUDE.md or testing policy forbids --no-verify, show me the exact
   rule and propose a one-line written exception limited to support-agent fix PRs
   (regression test still required, never auto-merged). Apply it only after I approve.

8. Share it. Re-run `bash peopledesk-kb/scripts/support-analyst/install.sh` (Windows: `install.ps1`) so this
   workspace gets /support:<id> (dev data only) and /support:<id>-prod-debug (may read
   production, read-only); tell me to reload Claude Code at the workspace root.
   The profile and its generated commands belong in the shared kit: prepare a branch in
   peopledesk-kb with profiles/<id>.json and commands/support/<id>*.md (plus any approved
   kit change, with tests), show me the diff, and open a pull request only when I say
   so — the PeopleDesk team reviews and merges it. Then every member of our team gets
   the commands with `./update-repos` (Windows: `update-repos.ps1`), or a pull and the installer.
   Verify with `peopledesk-support-analyze --profile <id> query --db dev --sql
   "SELECT DB_NAME() AS db"` (read-only).

9. First run — dry run. Run `/support:<id> 1 dry`. Open the preview HTML and the chat
   JSON and check them against the golden reference quality bar: every repository
   covered, evidence real, Project Tracker link correct. Report what is good and what
   is weak. Then STOP.

10. First live run — only when I say so: `/support:<id> 1`. Verify read-only that
    exactly one ticket row changed and only strAIFeadbackWithHTML, strAIFeadback and
    IsReadFromPlatform were written, and that the Google Chat card arrived in OUR space.

== Rules for every run of the agent ==
- Analyse every repository in the profile for every ticket; the CLI rejects
  analyses that skip one or cite files/lines that do not exist. Never weaken this.
- Only the CLI writes the database, only through `save`, only the three columns.
- Confirmed bug (rootCauseStatus CONFIRMED + technicalChangeType BUG_FIX) ->
  fix-start / fix-open: separate worktree, ai-fix/<ticket> branch, minimal fix plus a
  regression test, verification, commit and push with --no-verify, PR opened, NEVER
  merged. If verification cannot pass for a reason outside the change, record
  fixNotOpenedReason and tell me; never weaken a verify step to get a PR through.
  Anything less than CONFIRMED gets no PR.
- A ticket refined after it was saved (a PR opened later, a corrected classification)
  is re-saved with `save --update`: the stored report is overwritten and what changed is
  posted as a reply in the ticket's existing Google Chat thread. For a ticket whose
  thread is not on record, `--thread <message link>` attaches it to that thread.
- Never push to a base branch, never merge, never deploy.
- If something in the kit does not fit our product, propose a change to the shared
  kit (with tests) instead of forking it.
```
