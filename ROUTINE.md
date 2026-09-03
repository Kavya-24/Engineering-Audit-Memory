# Weekly Engineering Audit & Contribution Workflow

---

## 1. Setup

### 1.1 Repository Access
- Ensure that you have access to <your_repo_name>
- Confirm before the first run: **do not** attempt to create, store, or request credentials yourself — if auth isn't already set up, stop and tell the me to set it up first as the final output

### 1.2 Connector Access
- [ ] Ensure you have access to Slack
- [ ] Ensure you have access to Notion
- [ ] Ensure you have access to Jira / Rovo / Atlassian
- [ ] Ensure you have access to Google Drive

If any of these are missing, stop and report exactly which one(s) — don't run a partial pass silently. Note which sections of the output will be affected by the gap (e.g. missing Slack access means `references.md` and the comms portion of `next_action_items.md` can't be produced this week).

### 1.3 Folder layout

1. Call the Google Drive connector and check for a folder named `<your_brain_folder>`.
2. List the folder's contents and verify it contains three subdirectories: `updates`, `projects`, `notable_highlights`.
3. If the top-level folder or any of the three subdirectories don't exist, **stop and confirm with me before creating them** — don't silently scaffold a new structure into someone's Drive.

Schema (for reference):

```
<your_brain_folder>/
├── updates/                        ← append-only, one file per week, never edited after creation
│   ├── 2026-08-04.html
│   └── 2026-08-11.html
├── notable_highlights/             ← append-only, one file per week, personal record
│   ├── 2026-08-04.md
│   └── 2026-08-11.md
└── projects/                       ← overwritten each week, current state only
    ├── <project_name>/
    │   ├── context.md              ← auto-generated, then auto-updated by this workflow (see below)
    │   ├── next_action_items.md    ← overwritten each week, current state only
    │   └── references.md           ← append-only, accumulates Notion doc links, PR links, Google Doc links
    └── ...
```

**How `context.md` is created and maintained:**

- **First creation**: if a project folder under `projects/` doesn't have a `context.md`, generate one from the best available inputs — inferred activity, the tracked Epic/ERD/PRD in Notion, the Jira epic description. Don't finalize it silently: show me the draft once and get a one-time confirmation before it's saved as the project's baseline.
- **Every run after that**: this workflow can modify `context.md` directly, no confirmation needed — including the description. When Analysis (§3) surfaces something that belongs in it — a new Epic/ERD/PRD, a PROJECT_STATUS change, or activity that refines what the project is actually about — write the change directly into the file **and** append a dated line to the file's own `Activity Log` section, so every automated edit leaves a visible trail inside the file itself, not just in a chat message.

`context.md` template:
```markdown
# <Project Name>

## Description
2-4 sentences, plain words, what this project is and why it exists.

## Tracking
- Epic: <link or "none yet">
- ERD: <link or "none yet">
- PRD: <link or "none yet">
- Slack channel: <#channel>

## PROJECT_STATUS: <active | completed | deprioritized>

## Activity Log
- <date>: PROJECT_STATUS active → completed (Epic LEAD-123 marked Done)
- <date>: Added ERD link (<url>) — new doc found in Notion
```
Only append an Activity Log entry when something actually changed — don't pad it with a weekly "no change" line. The Description is auto-updated like everything else in this file: each run, refine it if this week's data adds real signal about what the project is or where it's headed — a one-line summary of what changed goes into the Activity Log alongside it, so a prose edit is just as traceable as a status flip.

### 1.4 Compute the variables
- [ ] Note that Github UserName = <GITHUB_USER_NAME>
- [ ] Note that FullName = <FULL_NAME>
- [ ] Note that Email ID = <EMAIL_ADDRESS>
- [ ] Compute the time window - (From 7 Days Ago - Today) (Keep Consistent 00:00 am GMT Format Timing).
- [ ] Ensure you have a <start_date, 00:00> and <end_date, 00:00> Start date is 7 days ago, and End Date is Today's Date.

### 1.5 Prepare scratchpads and Read Current State
- [ ] Create an empty <end_date>.md file for notable_highlights
- [ ] Read the context.md of each project — note PROJECT_STATUS and last week's next-steps, to know what was I working on up until last week.

---

## 2. Data Gathering

Pure collection only, one source at a time. **Pipe each source's raw pull to a local scratch file as it's gathered — do not hold the raw data in context.** These are transient working files for this run only (not part of the Drive schema in §1.3); §3 reads them back in for classification. Suggested naming: `scratch/pr_reviews_raw.md`, `scratch/self_prs_raw.md`, `scratch/slack_raw.md`, `scratch/notion_jira_raw.md`.

### 2.1 PR Review Data — raw pull
github filter (is:pr is:closed reviewed-by:<GITHUB_USER_NAME> merged:start_date..end_date -author:<GITHUB_USER_NAME>)

Steps -
 i. Pull every PR matching the filter, for all mentioned repos
 ii. For each PR, read the PR description + my comments on it — store the raw comment text against the PR, don't classify yet
 iii. Correlate each PR to a Project by matching its repo against `projects[].github_repos`

- [ ] Done completely for Repository
- [ ] Every PR has its project correlated (or marked unmapped)
- [ ] Written to `scratch/pr_reviews_raw.md`, not held in context

### 2.2 Self PR Data — raw pull
github filter (is:pr author:<GITHUB_USER_NAME> (created:start_date..end_date OR merged:start_date..end_date))

Steps -
 i. Pull every PR matching the filter, for both repos
 ii. Read the PR title, description, files changed, and any linked issues — store raw, don't classify yet
 iii. Correlate the PR to a Project by matching its repo against `projects[].github_repos`. If the PR description references a Jira key, cross-check it against that project's `jira_keys` to confirm the match. If a PR's repo doesn't map to any configured project, mark it **PROJECT: unmapped** — don't guess

- [ ] Done completely for Repository
- [ ] Every PR has its project correlated (or marked unmapped)
- [ ] Written to `scratch/self_prs_raw.md`, not held in context

### 2.3 Slack Data — raw pull

Privacy boundary — applies here absolutely, no exceptions: **never read, search, quote, or summarize direct messages** (1:1 DMs or private group MPIMs). Scope to the following filter - 
from:@<SLACK_HANDLE/FULL_NAME> -is:dm before:(end_date)after:(start_date)

Steps -
 i. If the message is in a thread, pull the full thread context
 ii. Search for all threads exhaustively.
 iii. For each pulled thread, store a raw 1-2 line gist plus who said what — don't classify my role yet

- [ ] Done for every project's configured slack_channels
- [ ] Privacy boundary confirmed — no DM/MPIM content pulled anywhere
- [ ] Every pulled thread has its raw gist stored
- [ ] Written to `scratch/slack_raw.md`, not held in context

### 2.4 Notion and Jira Data — raw pull

Steps -
 i. For each project, search its `notion_roots` for pages edited in the window — capture title, link, last-edited-by, last-edited date, and a one-line gist of what changed
 ii. For each project, run the Jira filter over its `jira_keys`: `project in (<keys>) AND updated >= start_date AND updated <= end_date ORDER BY updated DESC` — capture key, summary, status, assignee, updated date

- [ ] Done for every project's `notion_roots`
- [ ] Done for every project's `jira_keys`
- [ ] All doc/ticket metadata stored raw, no classification yet
- [ ] Written to `scratch/notion_jira_raw.md`, not held in context

---

## 3. Analysis

Read each scratch file from §2 back in, one source at a time, and judge it here. This is where CATEGORY, TYPE, and Ref get assigned, and where notable_highlights actually get written.

### 3.1 Classify PR Review Data
For each PR in `scratch/pr_reviews_raw.md`, classify my comments on it into the following categories -
 * if my comment materially would end up changing the implementation - CATEGORY - **material_change**
 * if my comment is an optimization (database, prompt etc) - CATEGORY - **optimization**
 * if my comment identifies a security / vulnerability - CATEGORY - **vulnerability**
 * for all others - CATEGORY - **other**

A notable_highlight is where category is not other. For EACH material_change / vulnerability / optimization, append a pr_review_notable_highlight into the <end_date>.md scratchpad created in §1.5:
```
<1-2 lines of the comment and its impact>
CATEGORY: <optimization | material_change | vulnerability>
TYPE: pr_review_notable_highlight
PROJECT: <project_name>
Ref: (Link to the PR comment)
```

- [ ] Every PR from §2.1 classified
- [ ] Ensure correct format
- [ ] Each pr_review_notable_highlight appended to the scratchpad

### 3.2 Classify Self PR Data
For each PR in `scratch/self_prs_raw.md`, classify it into the following categories -
 * if the PR closes out a production incident (label matches one of `incident_labels`) - CATEGORY - **incident_fix**
 * if the PR states its own quantifiable impact (latency, error rate, cost, throughput) in the description - CATEGORY - **impact**
 * for all others - CATEGORY - **other**

A notable_highlight is where category is not other. For EACH incident_fix / impact, append a pr_self_notable_highlight into the scratchpad:
```
<1-2 lines of what shipped and its stated impact>
CATEGORY: <incident_fix | impact>
TYPE: pr_self_notable_highlight
PROJECT: <project_name>
Ref: (Link to the PR)
```
If a PR is CATEGORY incident_fix or impact but states no actual number, write the highlight anyway with "impact not specified in PR" — never estimate a number that isn't written down.

Regardless of category, every self PR's link gets appended to its project's `references.md` in §4.1 — notability only gates the highlight, not the reference.

- [ ] Every PR from §2.2 classified
- [ ] Ensure correct format
- [ ] Each pr_self_notable_highlight appended to the scratchpad

### 3.3 Classify Slack Data
For each thread in `scratch/slack_raw.md`, classify <YOUR_NAME>'s role in it -
 * if <YOUR_NAME> proposed or landed the resolution - CATEGORY - **decision_driving**
 * if <YOUR_NAME> resolved a blocker someone else raised - CATEGORY - **unblocking**
 * for all others - CATEGORY - **other**

A notable_highlight is where category is not other. For EACH decision_driving / unblocking, append a comms_notable_highlight into the scratchpad:
```
<1-2 lines: what was discussed and my's role in it>
CATEGORY: <decision_driving | unblocking>
TYPE: comms_notable_highlight
PROJECT: <project_name>
Ref: (Link to the thread)
```
Regardless of category, every pulled thread's raw gist still goes into `updates/<end_date>.html` in §4.4 — notability only gates the highlight, not the raw log.

- [ ] Every thread from §2.3 classified
- [ ] Ensure correct format
- [ ] Each comms_notable_highlight appended to the scratchpad

### 3.4 Classify Notion and Jira Data
For each Notion doc in `scratch/notion_jira_raw.md` -
 * if the doc is an Epic, ERD, or PRD (by title or Notion property) - it's a **context.md update**: queue it to be added to that project's Tracking list
 * for all others - it's a normal reference, no context.md change needed

For each project, determine PROJECT_STATUS from this week's Jira data -
 * if the project's tracked Epic (per `context.md`) has Jira status Done/Closed - PROJECT_STATUS -> **completed**
 * if the project's tracked Epic has Jira status Won't Do/deprioritized, or has had no ticket movement for several consecutive weeks despite previously being active - PROJECT_STATUS -> **deprioritized**
 * if there was any PR, Jira, Notion, or Slack activity for the project this week - PROJECT_STATUS -> **active**
 * if none of the above give a clear signal - leave PROJECT_STATUS unchanged and flag it as ambiguous rather than guessing

Every Notion doc link and relevant Jira ticket link still gets appended to that project's `references.md` in §4.1, regardless of whether it also triggered a context.md update.

- [ ] Every doc/ticket from §2.4 classified
- [ ] PROJECT_STATUS determined for every project (or explicitly left unchanged with a reason)
- [ ] New Epic/ERD/PRD entries and status changes queued for §4.5

---

## 4. Synthesis & Output Finalization

### 4.1 Finalize project `references.md`
For each project, read the existing `references.md`, append only the genuinely new PR links (§3.2), Notion doc links, and Jira ticket links (§3.4) found this week, and save. Never remove or rewrite prior entries — this file only grows.

### 4.2 Rewrite project `next_action_items.md`
For each project, using `context.md` (read in §1.5, and updated in §4.5) plus everything gathered and classified in §2-3 for that project:
```markdown
# <Project Name> — Week of <start_date>

## Status
2-3 sentences: what shipped, what's blocked, trajectory — grounded in this week's PRs/Jira/Notion/Slack data.

## Next actions
- Owned where possible: "- @person: do X (blocked on Y)"

## Open questions / risks
- Anything unresolved needing a decision.
```
If a project had no activity across any source this week, say so plainly. This file is fully replaced each run — it is never appended to.

### 4.3 Finalize `notable_highlights/<end_date>.md`
This file was created empty in §1.5 and appended to throughout §3. Before saving, add a short header and make sure every entry follows its TYPE's exact template:
```markdown
# Notable Highlights: <YOUR_NAME> — <start_date> → <end_date>

## Summary
- PRs reviewed with notable comments: N
- Self PRs with notable impact: N
- Notable Slack contributions: N

## Highlights
<all pr_review_notable_highlight, pr_self_notable_highlight, and comms_notable_highlight
blocks collected during §3, in the exact format they were written>
```
Never edit a prior week's file in this folder.

### 4.4 Write `updates/<end_date>.html`
The raw, unfiltered record from §2 — every PR (reviewed and self, notable or not), every Jira ticket touched, every Notion doc touched, every pulled Slack thread gist — organized by source, not by project, no editorializing:
```html
<h1>Weekly Update Log: <start_date> to <end_date></h1>
<h2>GitHub</h2>
<h3>Repo-Name-1</h3>
<ul><li>#123 [MERGED] Title — @author — <a href="...">link</a></li></ul>
<h3>Repo-Name-2</h3>
<ul>...</ul>
<h2>Jira</h2>
...
<h2>Notion</h2>
...
<h2>Slack</h2>
...
<h2>Sources unavailable this run</h2>
<ul><li>&lt;source&gt;: &lt;reason&gt;, skipped</li></ul>
```
Create this as a new file — never edit a prior week's file in this folder.

### 4.5 Apply context.md updates
For each project, write directly into `context.md`: any new Epic/ERD/PRD/slack-channel entries, the computed PROJECT_STATUS from §3.4, and a refined Description if this week's data adds real signal about what the project is or where it's headed. Append a dated `Activity Log` line for each change (per §1.3's template) — status flips, new docs, and description refinements all get logged the same way. If a project had no `context.md` at all, generate it now per the first-creation rule in §1.3 — draft it, then hold it for my one-time confirmation before treating it as final. If PROJECT_STATUS was left ambiguous in §3.4, note that in §4.6 rather than silently leaving it out.

### 4.6 Report back
Short summary in chat: link to `updates/<end_date>.html`, link to `notable_highlights/<end_date>.md`, which projects had `next_action_items.md` updated, every `context.md` change made in §4.5 (including any new file awaiting my confirmation), and anything skipped this run (missing connector, unavailable source, zero-activity project, an unmapped self PR, an ambiguous PROJECT_STATUS).

---

## 5. Edge cases

| Situation | Handling |
|---|---|
| `<your_brain_folder>` or a subfolder doesn't exist | Stop, confirm with me before creating anything. |
| A project folder has no `context.md` | Generate a draft per §1.3, hold for my one-time confirmation, note it in §4.6. |
| PROJECT_STATUS signals are conflicting or missing | Leave PROJECT_STATUS unchanged, flag as ambiguous in §4.6 — don't guess a status. |
| A self PR's repo doesn't map to any configured project | Mark **PROJECT: unmapped**, still log it, don't guess. |
| Zero activity for a project/source | Say so plainly in `next_action_items.md` rather than padding. |
| One source's connector/auth fails | Log under "Sources unavailable" in `updates/<end_date>.html`, keep going. |
| Incident/impact PR with no stated number | "impact not specified in PR" — never infer one. |
| Ambiguous DM vs. channel result in Slack | Drop it. Never guess in the direction of including it. |

---

## 6. Maintenance notes

- `updates/` and `notable_highlights/` are append-only by design — corrections get a new dated file, never a rewrite of history.
- `references.md` only grows; `next_action_items.md` is the only file fully replaced each week.
- `context.md` is fully auto-maintained by this workflow every run — PROJECT_STATUS, the Tracking list, and the Description all update directly, with every change logged in the file's own Activity Log. First-time creation of a project's `context.md` still needs a one-time confirmation — everything after that is automatic.
- CATEGORY/TYPE/Ref is the fixed shape for every notable_highlight — keep new categories additive, don't repurpose existing ones.
- §2 (Data Gathering) and §3 (Analysis) are intentionally separate passes, and §2's raw pulls belong in scratch files, not in context — resist folding classification back into the pull steps, and resist holding raw pulls in-context "just for now."
- Pair this with a scheduled task if you want it to fire weekly without typing a trigger phrase.
