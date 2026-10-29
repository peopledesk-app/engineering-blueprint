# Replication prompt — set up the Client Support Analysis Agent

Two ways to use this page:

- **A PeopleDesk team member on a new machine** — use [Part A](#part-a--a-peopledesk-team-member-joins). The commands and the team's credentials are already in the repositories; you only install and verify.
- **Another product team** (Managerium, APON, PrimeVAT, …) that wants the same agent — use [Part B](#part-b--another-product-adopts-the-agent). They get a **self-contained package** and need **no access to any PeopleDesk repository**. The agent reads their pending Project Tracker support tickets oldest-first, investigates each one across **all** of their repositories, stores a Bangla-first bilingual report on the ticket, posts a Google Chat card with the Project Tracker link (and later updates into the same thread), names the developer who last changed the code involved, and — for a **confirmed** bug — opens a fix pull request for their developers to review.

Background (Bangla): [sharing-guide.md](sharing-guide.md) · design: [README.md](README.md)

> **Credentials never go into the chat.** The PeopleDesk team's database login, debug databases and webhook are committed in `peopledesk-kb/scripts/support-analyst/shared-credentials.json` (owner decision 2026-10-28; private repository). Every other team enters its **own** login and webhook with `peopledesk-support-analyze --profile <id> setup`, typed in its own terminal; the tool stores them only under `profiles.<id>` in a private file on that machine and refuses a login or webhook that already belongs to another product. The package for other teams never contains `shared-credentials.json` — the export searches the package for every known credential value and fails if one is found.

## How the commands are shared

The `/support:<id>` and `/support:<id>-prod-debug` commands are **committed** in the kit's `commands/support/`, rendered from one template per profile with workspace-relative paths only. The installers copy them into the workspace root's `.claude/commands/support/` (PeopleDesk: `./update-repos` refreshes that copy after every pull). Nobody edits the copy: change the template or a profile, run `sync-commands`, and commit both (a test fails while the committed commands are stale). Another team keeps its profile and commands in its own kit repository.

---

## Part A — a PeopleDesk team member joins

Open Claude Code at the PeopleDesk **workspace root** and paste:

```text
Set up the PeopleDesk Client Support Analysis Agent on this machine. The kit, the
/support:peopledesk commands and the team credentials are already committed; only install
and verify.

== Credential rules (non-negotiable) ==
- NEVER ask me to type, paste or confirm a password, login or webhook URL in this chat,
  and never accept one if I paste it — tell me to rotate it and use `setup` instead.
- Never print the contents of shared-credentials.json or of my private config.

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
3. Run `npm test` in peopledesk-kb/scripts/support-analyst and report the counts (all must pass).
4. Run `peopledesk-support-analyze --profile peopledesk doctor` (read-only). It must show
   credentialSource "shared-credentials.json" (the team login) or "profiles.peopledesk" (if I
   chose a personal login with `setup` in my own terminal). Check `gh auth status`.
5. Tell me to reload VS Code (Developer: Reload Window) at the workspace root and confirm
   that /support:peopledesk and /support:peopledesk-prod-debug appear when I type "/".
6. First run — only a preview: `/support:peopledesk 1 dry`. Show me what it found.
```

---

## Part B — another product adopts the agent

Nothing to clone and no repository access to grant. A PeopleDesk team member builds the package from a committed state of `peopledesk-kb`:

```bash
cd peopledesk-kb/scripts/support-analyst
node bin/peopledesk-support-analyze.js export-kit --out ~/Desktop/support-analysis-agent-kit
# zip that folder and send it (or keep one current zip on a shared drive)
```

The package holds:

| Path | What |
|---|---|
| `START-HERE.md` | One page, Bangla + English: what it is, what they need, three steps |
| `SETUP-PROMPT.md` | **The one prompt** the team pastes into Claude Code at its workspace root — installs, and later updates, everything |
| `KIT-VERSION.json` | Source commit and export time |
| `kit/scripts/support-analyst/` | CLI, command template, golden reference, SQL templates, profile template and the full test suite |
| `kit/scripts/mac-linux/`, `kit/scripts/windows/` | The shared-command installers |

It never contains a product profile, a generated command, `shared-credentials.json`, a local config or a preview. The receiving team:

1. Unzips the package anywhere.
2. Opens Claude Code at its workspace root (all product repositories side by side).
3. Fills in the `<…>` values of `SETUP-PROMPT.md` and pastes it.

Claude then checks prerequisites; places the kit at `<workspace>/<id>-support-kit/scripts/support-analyst` as the team's own git repository; installs it and runs the tests; discovers the repositories, their verify commands and the team roster; hands credentials over to the user's own terminal (`setup`); reads the product row with `doctor`; writes `profiles/<id>.json` and the commands (`sync-commands`); prints the view script for the DBA; optionally prepares the backfill; checks the fix-PR (`--no-verify`) policy; installs the commands; does a dry run; and writes `TEAM-SETUP.md` so colleagues who clone the team's kit repository can join with one prompt. It stops for a human at every credential, DDL, database write and push.

**Updates:** export a new package and send it; the team pastes the same `SETUP-PROMPT.md`. Claude shows the two `KIT-VERSION.json` files, replaces the kit but keeps the team's profile, commands and golden-reference additions, re-runs `sync-commands` and the tests, and does a dry run.

**Changes the kit needs** (a new section, a new enum, a bug) come back to the PeopleDesk team as a proposal with a test, are merged in `peopledesk-kb`, and reach every team with the next package — teams change only their profile and golden-reference notes, so updates stay clean.

The full prompt is in [`peopledesk-kb/scripts/support-analyst/export/SETUP-PROMPT.md`](../../peopledesk-kb/scripts/support-analyst/export/SETUP-PROMPT.md); it is the single source, shipped unchanged in every package.
