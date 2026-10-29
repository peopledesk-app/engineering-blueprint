# শেয়ারিং গাইড — অন্য প্রোডাক্ট টিমে সাপোর্ট অ্যানালাইসিস এজেন্ট চালু করা

> **কার জন্য:** Managerium, PrimeVAT, APON বা অন্য যেকোনো প্রোডাক্ট টিম, যারা PeopleDesk-এর মতো নিজেদের সাপোর্ট টিকিট AI দিয়ে বিশ্লেষণ করাতে চায়।
> **মূল ডকুমেন্ট:** [README.md](README.md) · **Claude Code-এ পেস্ট করার প্রম্পট:** [replication-prompt.md](replication-prompt.md)

---

## ১. এক নজরে — কী শেয়ার করা হচ্ছে

এজেন্টটি এখন **প্রোডাক্ট-নিরপেক্ষ**। প্রোডাক্টের সব নির্দিষ্ট তথ্য একটি **প্রোফাইল ফাইলে** থাকে; কোড, স্কিমা, রেন্ডারার, নিরাপত্তা নিয়ম ও টেস্ট সবার জন্য একই।

| অংশ | কোথায় | প্রতিটি টিমকে বদলাতে হবে? |
|---|---|---|
| CLI (ডেটাবেস পড়া, যাচাই, রেন্ডার, সংরক্ষণ, Google Chat) | `peopledesk-kb/scripts/support-analyst/`; অন্য টিমের কাছে `export-kit` প্যাকেজ হিসেবে যায় (ধাপ ৩) | না |
| প্রোডাক্ট প্রোফাইল | `profiles/<id>.json` — অন্য টিমের ক্ষেত্রে তাদের নিজস্ব কিট রিপোজিটরিতে | **হ্যাঁ — শুধু এটিই** |
| Golden reference (প্রতিটি রিপোজিটরিতে কী দেখতে হবে) | `golden-reference.md` | না (প্রয়োজনে নিজস্ব রিপোজিটরির নোট যোগ) |
| Slash command (`/support:<id>`, `/support:<id>-prod-debug`) | `commands/support-command.md.tmpl` থেকে `sync-commands` দিয়ে তৈরি, `commands/support/<id>*.md` হিসেবে **রিপোজিটরিতে commit করা** (শুধু ওয়ার্কস্পেস-আপেক্ষিক পাথ) | না — প্রোফাইল বদলালে শুধু `sync-commands` চালিয়ে commit |
| পেন্ডিং-টিকিট ভিউ | `view-sql` কমান্ড প্রোফাইল থেকে স্ক্রিপ্ট তৈরি করে | না (রিভিউ করে DBA চালাবেন) |
| সিক্রেট (DB লগইন, Chat webhook) | প্রতিটি মেশিনে `~/.config/peopledesk-support-analyst/config.json` | হ্যাঁ — কখনো শেয়ার/commit নয় |

সব প্রোডাক্টের টিকিট একই `VPT` ডেটাবেসে (`tblSupportTicket` + `tblProduct`), তাই ডেটাবেস অংশ ইতিমধ্যে প্রস্তুত।

### যাচাইকৃত প্রোডাক্ট তালিকা (`doctor`, 2026-10-22)

| প্রোডাক্ট | `strCode` | `intProductId` |
|---|---|---|
| Managerium ERP | `MGM` | 1 |
| PeopleDesk | `PD` | 2 ✅ চালু |
| PrimeVAT | `PV` | 3 |
| iBOS RMG | `RMG` | 4 |
| APON ERP | `APON` | 5 |
| iBOS Enterprise (Akij ERP) | `AERP` | 6 |
| Managerium Retail ERP | `POS` | 15 |
| MGM Health | `HMS` | 17 |

> প্রোফাইল বানানোর আগে নিজে `doctor` চালিয়ে আবার মিলিয়ে নিন — এই তালিকা সময়ের সাথে বদলাতে পারে।

---

## ১.১ ক্রেডেনশিয়াল — প্রতিটি টিমের নিজের, কখনো শেয়ার নয়

| ক্রেডেনশিয়াল | কে দেবে | কোথায় থাকবে |
|---|---|---|
| সাপোর্ট ডেটাবেস (`VPT`) লগইন | টিম লিড / DBA — **টিমের নিজস্ব লগইন** (সম্ভব হলে `sql/04-least-privilege-grant.sql` অনুযায়ী শুধু তিন কলামে লেখার অনুমতি) | প্রতিটি মেশিনে `~/.config/peopledesk-support-analyst/config.json`-এর `profiles.<id>` (chmod 600) — `setup` কমান্ড লেখে |
| Google Chat webhook | টিমের **নিজের** Chat space থেকে (Apps & integrations → Webhooks) | একই জায়গায়, `profiles.<id>.googleChatWebhookUrl` — `setup` কমান্ড লেখে |
| GitHub | প্রতিটি ডেভেলপারের নিজের `gh auth login` | GitHub CLI-র নিজস্ব স্টোর |
| Production শুধু-পড়ার DB (ঐচ্ছিক) | টিম লিড / DBA — **টিমের নিজস্ব** read-only লগইন | `profiles.<id>.debugDatabases.production` — `setup` লেখে (লুকানো ইনপুট) |
| Dev DB (ঐচ্ছিক) | কিছু দিতে হয় না — `setup`-এ শুধু `<appsettings ফাইল>#<key>` (যেমন `app-api/Api/appsettings.json#Development`) | রান-টাইমে রিপোজিটরি থেকে পড়া হয় |

**কীভাবে দেবেন:** নিজের terminal-এ (chat-এ নয়) চালান —

```bash
peopledesk-support-analyze --profile <id> setup
```

এটি host, port, database, **আপনার নিজের** SQL লগইন ও password (টাইপ করার সময় দেখা যায় না), টিমের Chat webhook ও workspace জিজ্ঞেস করে, আর শুধু `profiles.<id>`-এ লেখে।

**কোডে প্রয়োগ করা সুরক্ষা:**

- ক্রেডেনশিয়াল শুধু প্রোফাইল-ভিত্তিক — কনফিগের উপরের (shared) অংশে লগইন থাকলে CLI চলে না। তাই একই মেশিনে `apon` প্রোফাইল কখনো `peopledesk`-এর লগইন ব্যবহার করতে পারে না (যাচাই করা: credential ছাড়া `--profile apon` → `CONFIG_MISSING`)।
- `setup` অন্য প্রোফাইলে ইতিমধ্যে থাকা একই লগইন (`user` + `server`) বা একই webhook গ্রহণ করে না।
- `doctor` দেখায় কোন প্রোফাইলের ক্রেডেনশিয়াল ব্যবহার হচ্ছে (`credentialSource`), password ও webhook মাস্ক করা।
- Slash command ও replication prompt — দুটোই Claude-কে chat-এ ক্রেডেনশিয়াল চাওয়া বা নেওয়া নিষেধ করে।

> **PeopleDesk-এর DB লগইন, webhook বা টোকেন অন্য কোনো টিম কখনো ব্যবহার করবে না।** এগুলো শুধু প্রাইভেট `peopledesk-kb`-এর `shared-credentials.json`-এ (মালিকের সিদ্ধান্ত 2026-10-28) আর PeopleDesk টিমের মেশিনে থাকে। তাই অন্য টিমকে `peopledesk-kb`-এর অ্যাক্সেস দেওয়া হয় না — তারা পায় `export-kit` প্যাকেজ, যেটিতে এই ফাইল কখনো থাকে না এবং export নিজেই প্রতিটি জানা ক্রেডেনশিয়াল মান খুঁজে নিশ্চিত করে। কোনো সিক্রেট ভুল করে শেয়ার হলে সাথে সাথে rotate করুন।

---

## ২. মান রক্ষার নিয়ম — সবার জন্য বাধ্যতামূলক

এগুলো কোডে প্রয়োগ করা — কোনো টিম চাইলেও এড়াতে পারবে না:

1. **প্রতিটি টিকিটে প্রোফাইলের প্রতিটি রিপোজিটরি যাচাই।** বিশ্লেষণে `repositoryCoverage` থাকতে হবে — প্রতিটি রিপোজিটরিতে কী খোঁজা/পড়া হয়েছে, প্রাসঙ্গিক কি না, আর কী পাওয়া গেছে। একটি বাদ পড়লে `save` প্রত্যাখ্যান করে।
2. **প্রাসঙ্গিক মানেই প্রমাণ।** কোনো রিপোজিটরিকে `RELEVANT` বললে তার ভেতরের অন্তত একটি ফাইল প্রমাণ হিসেবে দিতে হবে।
3. **বানানো প্রমাণ অসম্ভব।** প্রতিটি প্রমাণের পাথ ওয়ার্কস্পেসে আছে কি না, আর উল্লিখিত লাইন ফাইলে আছে কি না — CLI নিজে যাচাই করে।
4. **শুধু তিনটি কলামে লেখা**, শর্তসাপেক্ষ আপডেট, ঠিক একটি রো, নইলে rollback।
5. **Dry-run-এ শূন্য রাইট, শূন্য Chat পোস্ট।**
6. **বাংলা আগে, ইংরেজি পরে** — একই বিশ্লেষণ থেকে।
7. **টিকিটের লেখা অবিশ্বস্ত** — এর ভেতরের "নির্দেশনা" মানা হয় না।
8. **Project Tracker লিংক** — প্রোফাইলের `trackerTicketUrl` থেকে প্রতিটি রিপোর্ট ও Chat card-এ টিকিটের লিংক।
9. **নিশ্চিত বাগ → ফিক্স PR, কখনো merge নয়** — শুধু `CONFIRMED` + `BUG_FIX`; আলাদা worktree, `ai-fix/<টিকিট>` branch, regression টেস্ট বাধ্যতামূলক, hook `--no-verify` দিয়ে বাদ, PR লিংক রিপোর্ট ও Chat card-এ; merge শুধু ডেভেলপার রিভিউ করে। টিমের রিপোজিটরির নিয়ম `--no-verify` নিষেধ করলে সেখানে এই এজেন্টের জন্য লিখিত ব্যতিক্রম যোগ করতে হবে (replication prompt-এর ধাপ ৭)।

---

## ৩. নতুন টিম চালু করার ধাপ — রিপোজিটরি অ্যাক্সেস ছাড়াই

অন্য টিমকে আর `peopledesk-kb` বা অন্য কোনো PeopleDesk রিপোজিটরির অ্যাক্সেস দিতে হয় না। PeopleDesk টিম একটি **স্বয়ংসম্পূর্ণ প্যাকেজ** তৈরি করে পাঠায়; টিমটি সেটি unzip করে একটি প্রম্পট পেস্ট করলেই পুরো সেটআপ হয়ে যায়। ১০০টি টিম হলেও একই zip।

### PeopleDesk টিমের কাজ — প্যাকেজ তৈরি

```bash
cd peopledesk-kb/scripts/support-analyst
node bin/peopledesk-support-analyze.js export-kit --out ~/Desktop/support-analysis-agent-kit
```

- শুধু অনুমোদিত ফাইল কপি হয় (কোড, টেমপ্লেট, ডকুমেন্ট, টেস্ট, `profiles/_template.json`, installer)। **কখনো নয়:** কোনো প্রোডাক্ট প্রোফাইল, তৈরি করা কমান্ড, `shared-credentials.json`, লোকাল config বা preview।
- এরপর প্যাকেজের প্রতিটি ফাইলে এই মেশিনে জানা প্রতিটি ক্রেডেনশিয়াল মান খোঁজা হয়; একটিও পাওয়া গেলে প্যাকেজ মুছে export ব্যর্থ হয় (`EXPORT_SECRET_FOUND`)।
- `KIT-VERSION.json`-এ source commit থাকে — commit করা অবস্থা থেকে export করুন (`uncommittedChanges: false`)।
- ফোল্ডারটি zip করে পাঠান, বা একটি শেয়ার্ড ড্রাইভে সর্বশেষ zip রাখুন।

### গ্রহীতা টিমের কাজ — তিন ধাপ

| # | কাজ | কে |
|---|---|---|
| 1 | প্যাকেজ unzip (যেকোনো জায়গায়) | টিম লিড |
| 2 | ওয়ার্কস্পেস রুটে (সব রিপোজিটরি পাশাপাশি) Claude Code খোলা | টিম লিড |
| 3 | `SETUP-PROMPT.md`-এর `<…>` মান (প্রোডাক্ট নাম, প্রোফাইল id, Project Tracker URL, ফিক্স-PR রিপোজিটরি, টিম লিড) বসিয়ে পেস্ট | টিম লিড |

এরপর Claude নিজে করে, আর যেখানে মানুষ লাগে সেখানে থামে:

1. Node.js / git / `gh` যাচাই।
2. কিট `<workspace>/<id>-support-kit/scripts/support-analyst`-এ বসানো — টিমের **নিজস্ব** git রিপোজিটরি।
3. Installer (`install.sh` / `install.ps1`) ও `npm test` (সব টেস্ট পাস বাধ্যতামূলক)।
4. রিপোজিটরি discovery, প্রতিটির build/lint/test একবার চালিয়ে যাচাই, আর git ইতিহাস থেকে টিম roster প্রস্তাব (টিম লিড অনুমোদন করেন)।
5. **থামে** — টিম লিড নিজের terminal-এ `peopledesk-support-analyze --profile <id> setup` চালিয়ে নিজের DB লগইন, টিমের webhook ইত্যাদি দেন। তারপর `doctor` দিয়ে প্রোডাক্ট রো যাচাই।
6. `profiles/<id>.json` তৈরি, `sync-commands`, আবার `npm test`।
7. **থামে** — ভিউ স্ক্রিপ্ট DBA রিভিউ করে চালান।
8. ঐচ্ছিক backfill (প্রিভিউ → অনুমোদন)।
9. টিমের রিপোজিটরির নিয়ম `--no-verify` নিষেধ করলে লিখিত ব্যতিক্রম প্রস্তাব।
10. কমান্ড ইনস্টল, VS Code reload, `/support:<id> 1 dry`।
11. কিট রিপোজিটরি commit (অনুমোদনে), টিমের GitHub-এ push (অনুমোদনে), আর সহকর্মীদের জন্য `TEAM-SETUP.md` — তারা কিট clone করে এক প্রম্পটেই যুক্ত হন।
12. লাইভ রান শুধু বললে।

### আপডেট

নতুন প্যাকেজ পাঠান; টিম একই `SETUP-PROMPT.md` আবার পেস্ট করে। Claude দুটি `KIT-VERSION.json` দেখিয়ে অনুমতি নেয়, টিমের প্রোফাইল, কমান্ড ও golden-reference সংযোজন রেখে বাকি কিট বদলায়, `sync-commands` ও টেস্ট চালায়, তারপর dry run।

### PeopleDesk টিমের নতুন সদস্য

কিছু বদলায়নি: `./update-repos` (Windows: `update-repos.ps1`), তারপর `install.sh` (Windows: `install.ps1`) — ক্রেডেনশিয়াল `shared-credentials.json` থেকে আসে, `setup` লাগে না (চাইলে ব্যক্তিগত লগইনের জন্য)। বিস্তারিত [replication-prompt.md](replication-prompt.md)-এর Part A। Claude Code অবশ্যই **ওয়ার্কস্পেস রুটে** খুলতে হবে।

উদাহরণ — `setup` চালানোর পর এক মেশিনে দুই প্রোডাক্ট (প্রতিটির নিজস্ব লগইন ও webhook):

```json
{
  "workspaceRoot": "/Users/<you>/peopledesk-ba-workspace",
  "defaultProfile": "peopledesk",
  "profiles": {
    "peopledesk": { "server": "<host>", "port": 32887, "database": "VPT", "user": "<PeopleDesk login>", "password": "…", "googleChatWebhookUrl": "<PeopleDesk space webhook>" },
    "managerium": { "server": "<host>", "port": 32887, "database": "VPT", "user": "<Managerium login>", "password": "…", "googleChatWebhookUrl": "<Managerium space webhook>", "workspaceRoot": "/Users/<you>/managerium-workspace" }
  }
}
```

> প্রোডাক্টের পরিচয় (কোড, আইডি, ভিউ, রিপোজিটরি) শুধু প্রোফাইল থেকে আসে — কনফিগের উপরের অংশে লিখলেও তা উপেক্ষিত হয়। ফলে এক প্রোডাক্টের webhook বা কোড অন্যটিতে ভুল করে যেতে পারে না। প্রতিটি প্রোফাইলের রান-লক আলাদা।

---

## ৪. প্রোফাইলে রিপোজিটরি লেখার নিয়ম

`repositories`-এ যা লেখা হবে, প্রতিটি টিকিটে ঠিক সেগুলোই বাধ্যতামূলকভাবে যাচাই হবে। তাই:

- **সব রিপোজিটরি দিন** — ব্যাকএন্ড, ফ্রন্টএন্ড, রিপোর্ট, শিডিউলার, রিয়েল-টাইম, Knowledge Base, SQA। একটি বাদ মানে প্রতিটি রিপোর্টে সেই অংশ অন্ধ।
- `name` = ওয়ার্কস্পেসের ভেতরের আসল ফোল্ডারের নাম।
- `role` = সেখানে বিশ্লেষক ঠিক কী দেখবেন (এটি slash command-এ টেবিল হিসেবে যায়)।
- Knowledge Base না থাকলে `knowledgeBase: null` — তবে PeopleDesk-এর মতো KB তৈরি করলে বিশ্লেষণের মান অনেক বাড়ে।

PeopleDesk-এর প্রোফাইল নমুনা হিসেবে দেখুন: `profiles/peopledesk.json`।

---

## ৫. মান যাচাই চেকলিস্ট (প্রথম ৫টি টিকিটের পর)

- [ ] প্রতিটি রিপোর্টে "রিপোজিটরি যাচাই" অংশে সব রিপোজিটরি আছে, প্রতিটিতে বাস্তব সার্চ/ফাইল।
- [ ] প্রমাণের ফাইল খুলে দেখলে দাবিটি সত্যি।
- [ ] শ্রেণিবিন্যাস `golden-reference.md`-এর গাইডের সাথে মেলে; অনুমান `LIKELY`-র বেশি দাবি করে না।
- [ ] বাংলা ও ইংরেজি একই কথা বলে।
- [ ] Google Chat card ঠিক space-এ গেছে; কাস্টমারের মূল বর্ণনা card-এ নেই।
- [ ] শুধু তিনটি কলাম বদলেছে (`strAIFeadbackWithHTML`, `strAIFeadback`, `IsReadFromPlatform`)।
- [ ] Chat card ও রিপোর্টে Project Tracker লিংক সঠিক টিকিট খোলে।
- [ ] কোনো ফিক্স PR থাকলে: শুধু নিশ্চিত বাগে, regression টেস্টসহ, `ai-fix/…` branch-এ, merge হয়নি।

---

## ৬. মালিকানা ও পরিবর্তন

| বিষয় | মালিক |
|---|---|
| CLI, স্কিমা, golden reference, টেস্ট | PeopleDesk টিম (এই রিপো) — পরিবর্তন PR দিয়ে, টেস্টসহ |
| প্রোডাক্ট প্রোফাইল | সংশ্লিষ্ট প্রোডাক্ট টিম |
| ভিউ, backfill, DB পারমিশন | DBA (রিভিউ করা স্ক্রিপ্ট) |
| Webhook ও DB পাসওয়ার্ড | প্রতিটি টিম নিজে; শেয়ার করা হলে rotate |

নতুন প্রয়োজন (যেমন নতুন সেকশন, নতুন enum) সব টিমের জন্য একসাথে আসে — তাই কোডের উৎস এক জায়গায় (`peopledesk-kb`)। প্রতিটি টিমের কাছে কিটের একটি কপি থাকলেও তারা শুধু নিজেদের প্রোফাইল ও golden-reference নোট বদলায়; কোড পরিবর্তনের প্রস্তাব (টেস্টসহ) PeopleDesk টিমে পাঠায়, যা পরের প্যাকেজে সবার কাছে পৌঁছায়। এতে আপডেট সবসময় পরিষ্কারভাবে বসে।
