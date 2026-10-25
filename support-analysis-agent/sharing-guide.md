# শেয়ারিং গাইড — অন্য প্রোডাক্ট টিমে সাপোর্ট অ্যানালাইসিস এজেন্ট চালু করা

> **কার জন্য:** Managerium, PrimeVAT, APON বা অন্য যেকোনো প্রোডাক্ট টিম, যারা PeopleDesk-এর মতো নিজেদের সাপোর্ট টিকিট AI দিয়ে বিশ্লেষণ করাতে চায়।
> **মূল ডকুমেন্ট:** [README.md](README.md) · **Claude Code-এ পেস্ট করার প্রম্পট:** [replication-prompt.md](replication-prompt.md)

---

## ১. এক নজরে — কী শেয়ার করা হচ্ছে

এজেন্টটি এখন **প্রোডাক্ট-নিরপেক্ষ**। প্রোডাক্টের সব নির্দিষ্ট তথ্য একটি **প্রোফাইল ফাইলে** থাকে; কোড, স্কিমা, রেন্ডারার, নিরাপত্তা নিয়ম ও টেস্ট সবার জন্য একই।

| অংশ | কোথায় | প্রতিটি টিমকে বদলাতে হবে? |
|---|---|---|
| CLI (ডেটাবেস পড়া, যাচাই, রেন্ডার, সংরক্ষণ, Google Chat) | `peopledesk-kb/scripts/support-analyst/` | না |
| প্রোডাক্ট প্রোফাইল | `profiles/<id>.json` | **হ্যাঁ — শুধু এটিই** |
| Golden reference (প্রতিটি রিপোজিটরিতে কী দেখতে হবে) | `golden-reference.md` | না (প্রয়োজনে নিজস্ব রিপোজিটরির নোট যোগ) |
| Slash command (`/support:<id>`) | `commands/support-command.md.tmpl` থেকে স্বয়ংক্রিয় তৈরি | না |
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

> **PeopleDesk-এর DB লগইন, webhook বা টোকেন অন্য কোনো টিম কখনো ব্যবহার করবে না।** এগুলো শুধু PeopleDesk টিমের মেশিনে থাকে, কোনো রিপোজিটরিতে নয় — অন্য টিম prompt চালালেও তাদের মেশিনে এগুলো পৌঁছায় না। কোনো সিক্রেট ভুল করে শেয়ার হলে সাথে সাথে rotate করুন।

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

## ৩. নতুন টিম চালু করার ধাপ

### পথ A — একই ওয়ার্কস্পেস কাঠামো (সুপারিশকৃত)

টিমের নিজস্ব ওয়ার্কস্পেস ফোল্ডারে তাদের সব রিপোজিটরি পাশাপাশি থাকবে (যেমন `managerium-workspace/mgm-api`, `mgm-front`, ...)।

| # | কাজ | কে | কমান্ড / ফাইল |
|---|---|---|---|
| 1 | `support-analyst` ফোল্ডারটি টিমের ওয়ার্কস্পেসে আনা (এই রিপো clone, অথবা ফোল্ডার কপি) | টিম লিড | — |
| 2 | প্রোডাক্ট রো যাচাই | টিম লিড | `peopledesk-support-analyze doctor` |
| 3 | প্রোফাইল তৈরি — `profiles/_template.json` কপি করে `profiles/<id>.json`; প্রোডাক্ট কোড/আইডি, ভিউ নাম, `trackerTicketUrl` (`{ticketId}` সহ), **সব** রিপোজিটরি ও প্রতিটির ভূমিকা, আর ফিক্স PR-যোগ্য কোড রিপোজিটরিতে `github` + `baseBranch` | টিম লিড | `profiles/managerium.json` |
| 4 | প্রোফাইল থেকে ভিউ স্ক্রিপ্ট তৈরি, রিভিউ, DBA দিয়ে চালানো | টিম লিড + DBA | `peopledesk-support-analyze --profile <id> view-sql > view.sql` |
| 5 | পুরনো টিকিট বাদ দিতে চাইলে backfill (প্রিভিউ → backup → আপডেট) | DBA | `sql/03-backfill-…sql` (প্রোডাক্ট কোড বদলে) |
| 6 | ইনস্টল — প্রতিটি প্রোফাইলের জন্য `/support:<id>` কমান্ড তৈরি হয় | প্রতিটি সদস্য | `PDSA_WORKSPACE_ROOT=<ওয়ার্কস্পেস> bash install.sh` |
| 7 | ক্রেডেনশিয়াল — নিজের terminal-এ `setup` চালিয়ে **নিজের** DB লগইন ও **টিমের নিজস্ব** webhook; `gh auth status` দিয়ে GitHub লগইন যাচাই | প্রতিটি সদস্য | `peopledesk-support-analyze --profile <id> setup` |
| 8 | প্রথম রান — dry-run, preview দেখে মান যাচাই | টিম লিড | `/support:<id> --limit 1 --dry-run` |
| 9 | লাইভ রান | টিম | `/support:<id> --limit 5` |

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

### পথ B — Claude Code দিয়ে স্বয়ংক্রিয় সেটআপ

টিমের ওয়ার্কস্পেসে Claude Code খুলে [replication-prompt.md](replication-prompt.md)-এর প্রম্পটটি পেস্ট করুন (প্রোডাক্টের নাম বসিয়ে)। Claude উপরের ধাপগুলো যাচাইসহ করবে এবং যেখানে মানুষের অনুমোদন দরকার (ভিউ তৈরি, backfill, webhook) সেখানে থেমে জিজ্ঞেস করবে।

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

নতুন প্রয়োজন (যেমন নতুন সেকশন, নতুন enum) সব টিমের জন্য একসাথে আসে — তাই কোড এক জায়গায় রাখা হয়েছে, কপি করে আলাদা সংস্করণ বানানো নিরুৎসাহিত।
