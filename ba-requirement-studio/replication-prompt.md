# Replication prompt — set up the BA Requirement Studio

The BA Requirement Studio turns the business team's notes into a reviewed, approved requirement — analysis, client document, BRS, developer task and a clickable HTML mockup — and hands it to development as a Project Tracker ticket, with every step announced in the BA Google Chat space. Code and templates: `peopledesk-ba-studio`. Design (Bangla): [implementation-plan.md](implementation-plan.md).

Two ways to use this page:

- **A PeopleDesk BA team member on a new machine** — [Part A](#part-a--a-peopledesk-ba-team-member-joins). The `/ba:peopledesk:*` commands are already in the repository; you only install and add your own Google Chat webhook.
- **Another product team** (Managerium, APON, PrimeVAT, …) — [Part B](#part-b--another-product-adopts-the-studio). You add one product profile to the shared studio; the commands `/ba:<product>:*` are generated from the same templates.

**How to use either part:** open Claude Code at the **workspace root** (the folder that holds `peopledesk-ba-studio/`, the product's Knowledge Base and its source repositories), paste the block, and fill in the `<…>` values. Claude asks for anything missing and stops for your approval before it commits or opens a pull request.

> **The webhook is yours, and it never goes into the chat.** Each person adds their team's Google Chat webhook to their own `~/.config/peopledesk-ba-studio/config.json` under `"profiles": { "<id>": { "googleChatWebhookUrl": "…" } }` (mode 600), in their own editor. The CLI refuses a webhook left at the top level of that file and only ever reports it as `configured (secret)`. Never reuse another team's webhook.

## How the commands are shared

The seven commands per product (`new`, `revise`, `mockup`, `review`, `approve`, `publish`, `status`) are **committed** in `peopledesk-ba-studio/commands/ba/<product>/`, rendered from `commands/*.md.tmpl` for every profile with workspace-relative paths only. `install.sh` (and peopledesk-kb's workspace installer) copies them into the workspace root's `.claude/commands/ba/`, and `./update-repos` refreshes that copy after every pull — so the whole team runs the same, reviewed version. To change a command, edit the template or the profile, run `ba-studio sync-commands`, and commit both; a test fails while the committed commands are stale.

---

## Part A — a PeopleDesk BA team member joins

```text
Set up the PeopleDesk BA Requirement Studio on this machine. The studio and the
/ba:peopledesk:* commands are already committed; only install and verify.

== Rules ==
- NEVER ask me to type, paste or confirm a Google Chat webhook URL (or any token) in this
  chat, and never accept one if I paste it — tell me to rotate it instead.
- Never edit ~/.config/peopledesk-ba-studio/config.json to add a secret yourself.

== Steps — report each result ==
1. Check the workspace: this folder holds peopledesk-ba-studio/, peopledesk-kb/ and the
   PeopleDesk repositories (app-front at least, for the mockups). List what is missing
   (./update-repos clones it).
2. Install: `bash peopledesk-kb/scripts/mac-linux/install-workspace-commands.sh` (if
   peopledesk-kb is present), then `bash peopledesk-ba-studio/install.sh`. Report the
   commands it installed. If ~/.local/bin is not on PATH, tell me the exact line to add
   to my shell profile (do not edit it yourself).
3. Run `npm test` in peopledesk-ba-studio/tools/ba-cli and report the result.
4. Google Chat — STOP and tell me: "Open ~/.config/peopledesk-ba-studio/config.json in
   your own editor and add, under "profiles": { "peopledesk": { "googleChatWebhookUrl":
   "<your BA space webhook>" } }. Reply 'done' here — do not paste the URL."
   Then run `node peopledesk-ba-studio/tools/ba-cli/bin/ba-studio.js --profile peopledesk config`:
   googleChatWebhook must read "configured (secret)".
5. Tell me to reload VS Code (Developer: Reload Window) at the workspace root and confirm
   that /ba:peopledesk:new … /ba:peopledesk:status appear when I type "/".
6. First use: `/ba:peopledesk:status` (read-only). Show me the requirement list.
```

---

## Part B — another product adopts the studio

```text
Set up the BA Requirement Studio for our product by REUSING the shared studio in
peopledesk-ba-studio. Do not build a new one and do not fork it.

== Our values — NOT secrets (ask me for anything left as <…>; never guess) ==
Product name ............ <e.g. Managerium ERP>
Profile id .............. <lowercase, e.g. managerium>  -> commands /ba:<id>:new, …
Requirement id prefix ... <2–6 capital letters, e.g. MGR>  -> MGR-2026-0001
Workspace root .......... <absolute path of THIS workspace>; peopledesk-ba-studio must be
                          cloned directly inside it (the commands use paths relative to it)
Knowledge Base .......... <workspace-relative path of its _index.md, e.g. managerium-kb/knowledge/_index.md>
Analysis modes .......... <workspace-relative paths of the new-feature / existing-feature
                          guidance, or "none">
Frontend repository ..... <workspace folder whose screens the mockups must match>
Project Tracker ......... project name <…>, task type <e.g. Feature>, default priority <e.g. Medium>
Google Chat webhook ..... I add it MYSELF in step 5. Do NOT ask me for it in this chat.

== Rules (non-negotiable) ==
- NEVER ask me to type, paste or confirm a webhook URL or token in this chat, and never
  accept one if I paste it — tell me to rotate it instead. Never read, copy or reuse
  another team's webhook (e.g. the "peopledesk" profile), even if it is on this machine.
- Follow peopledesk-ba-studio/CLAUDE.md: Bangla in chat, English files, fictional sample
  data only in mockups, approvals only by a named human, nothing writes to any database,
  Project Tracker tickets are created by a person.

== Read first, in full ==
1. peopledesk-ba-studio/README.md and peopledesk-ba-studio/CLAUDE.md
2. peopledesk-ba-studio/docs/mockup-spec.md
3. peopledesk-blueprint/ba-requirement-studio/implementation-plan.md (if available)

== Steps — report each result and wait where it says STOP ==

1. Install the studio: `bash peopledesk-ba-studio/install.sh`. If ~/.local/bin is not on
   PATH, tell me the line to add; until then use
   `node peopledesk-ba-studio/tools/ba-cli/bin/ba-studio.js`. Run `npm test` in
   peopledesk-ba-studio/tools/ba-cli and report the result.

2. Discovery (read-only). List the repository folders in this workspace and what each
   holds. Confirm the Knowledge Base path and the mode files exist; if there is no
   Knowledge Base, STOP and tell me — the studio is knowledge-first, so recommend
   building one before real requirements are written. Confirm the frontend repository
   exists and name its menu/route file so the mockups can match real screens.

3. Profile. Copy peopledesk-ba-studio/profiles/_template.json to profiles/<id>.json and
   fill it: id, productName, commandNamespace "ba:<id>", idPrefix, knowledgeBase, modes
   (newFeature / existingFeature / knowledgeDriven — only files that exist), frontendRepo,
   repoUrl https://github.com/peopledesk-app/peopledesk-ba-studio, defaultBranch, and
   projectTracker (mode "manual", baseUrl https://pt.ibos.io, projectName, taskType,
   defaultPriority). Check the id prefix is not used by another profile.
   Then generate the shared commands and check everything:
     node peopledesk-ba-studio/tools/ba-cli/bin/ba-studio.js profiles        # lists <id>
     node peopledesk-ba-studio/tools/ba-cli/bin/ba-studio.js sync-commands   # writes commands/ba/<id>/*.md
     npm test                                                                # in tools/ba-cli, incl. the drift check
   Show me the rendered /ba:<id>:new command and confirm it names OUR Knowledge Base and
   modes and contains no absolute path.

4. Share it — STOP before anything leaves this machine. Prepare a branch in
   peopledesk-ba-studio with profiles/<id>.json and commands/ba/<id>/*.md (plus any kit
   change I approved, with tests), show me the diff, and open a pull request only when I
   say so — the PeopleDesk team reviews and merges it. Then every member of our team gets
   the commands with `./update-repos` (or a pull and install.sh). Re-run install.sh here
   and tell me to reload Claude Code at the workspace root.

5. Google Chat — STOP and tell me: "Open ~/.config/peopledesk-ba-studio/config.json in
   your own editor and add, under "profiles": { "<id>": { "googleChatWebhookUrl":
   "<your team's BA space webhook>" } }. Reply 'done' here — do not paste the URL."
   Then run `node peopledesk-ba-studio/tools/ba-cli/bin/ba-studio.js --profile <id> config`:
   it must show profile "<id>" and googleChatWebhook "configured (secret)".

6. First requirement — a rehearsal. Ask me for a short, real set of business notes and run
   `/ba:<id>:new` with them. Check the result against the studio's rules: documents 01–04
   filled with no TODO left, every BR- rule covered by an AC- criterion, open questions as
   OQ- rows, the mockup builds (`/ba:<id>:mockup`) with fictional data and screens that
   match our frontend, and `ba-studio check <REQ>` passes. Report what is good and what is
   weak. Do not submit it for review unless I say so.

== Rules for every use ==
- State changes go through the CLI only (submit, approve, revise, publish, record-ticket).
- Approval is a human decision: the approver's name comes from me; never approve on your own.
- Never write to any database; never create the Project Tracker ticket yourself.
- If something in the studio does not fit our product (for example requirement listing per
  product), propose a change to the shared studio (with tests) instead of forking it.
```
