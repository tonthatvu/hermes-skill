---
name: release-summary
description: Syncs the Confluence "Summary" page (GP2 release history) with new fixVersions/releases from Jira. Use when the user says "update the Summary with new releases", "sync release notes", "update release summary", or mentions Confluence pageId 455213122 / Jira project GP2. Automatically detects new versions, computes feature count/story points/contributors, builds and pushes the HTML table in the agreed format, and verifies the push didn't corrupt the table.
---

# Release Summary — sync Jira GP2 releases into Confluence

This skill replaces having to re-explain the full context every time the Confluence release
summary needs updating. Reading this file is enough to re-run the process without re-asking
the user about decisions that are already settled (unless a genuinely new, ambiguous case
comes up).

## Fixed context

- **Confluence "Summary" page**: `pageId 455213122`, space `98306`
  (key `~712020a3dcebcbe93646fea464f119dd38b08c`), parent `451313773` ("Release notes"),
  cloudId `goopay.atlassian.net`.
- **Jira project**: `GP2` (id 10040). Story Points field = `customfield_10032`.
- **Legacy epic**: `GP2-9` "Release build" (id 13177) — holds every manual release task from
  **Phase 1**.
- **Phase-transition date**: 2026-08-06. Before that = Phase 1 (manual tasks under GP2-9,
  features listed as bullets in the task description). From that date = Phase 2 (native Jira
  Release/fixVersion, each Story/Bug tagged directly with a version, each version has its own
  release-notes page).
- The Summary page has 3 sections: (1) Overview, (2) Phase 1 stats (a static table, does not
  change when new Phase 2 releases are added), (3) **the full release list table** — this is
  the one that needs updating. Columns: Tháng (Month) | Ngày (Date) | Release | Version |
  Trạng thái (Status) | Feature | Story Points | Contributor | Link. Sorted **newest first**.
  (Note: the table's own column headers stay in Vietnamese — that's the page's existing
  language, don't translate the live content, only this skill's instructions are in English.)

## Process when the user asks to "update with new releases"

### Step 1 — Fetch the page's current live state (don't trust memory)
```
getConfluencePage(pageId=455213122, contentFormat="html")
```
If it 404s transiently, retry — this has happened before due to a temporary glitch, not the
page being deleted (confirm with `searchConfluenceUsingCql(cql: title ~ "Summary" AND space =
"...")` if the 404 repeats).

Read section 3's table to see which versions are already present (Version column).

### Step 2 — Pull every issue with a fixVersion from Jira
```
searchJiraIssuesUsingJql(
  jql: 'project = GP2 AND fixVersion is not EMPTY ORDER BY fixVersion DESC',
  fields: ["summary","issuetype","status","fixVersions","customfield_10032","parent"],
  maxResults: 100
)
```
- First run with `searchResultMode: "count"` to get the total and compare against last time,
  to know whether a new version exists at all.
- If `isLast: false`, paginate further with `nextPageToken`.
- Large results auto-save to a tool-result file — use `jq` on it instead of reading it raw.

### Step 3 — Group by version, classify what needs handling
```
jq -r '.issues[].fields.fixVersions[] | [.name,.releaseDate,.released] | @tsv' | sort -u
```
Classify each version:
1. **Already in the table, nothing changed** → skip.
2. **Already in the table but data drifted** (story points increased because an issue got the
   fixVersion attached later, or — importantly — **the version may have been RENAMED in Jira**,
   e.g. "v1.5.1" was used before but got renamed to "v1.6.0" while still pointing at the same
   release-notes page). How to detect a rename: if a version already in the table suddenly
   returns 0 issues on its own (`fixVersion = "vX.Y.Z"` → empty), re-fetch the release-notes
   Confluence page linked in that row's Link column and read the "Version" field in its
   "1. Release Information" table — that's the current real name. **Fix the label in the
   Summary table** and recompute feature count/points.
3. **Brand-new version, `released:true` or already has real content** → add a new row.
4. **`released:false`, no `releaseDate`, most issues still To Do/In Progress** → this is a
   future/planned version (backlog). **Do NOT add a row for it** — just mention it exists to
   the user so they're aware, but it's not release history yet.

### Step 4 — Compute the data for each new/changed version

Two possible row shapes:

**A. Normal "Phase 2" row** (version with real Story/Bug work):
- **Feature** = count of top-level (`hierarchyLevel 0`) issues of type `Story` + `Bug`
  (do NOT count `Task`/`Sub-task` — a top-level Task is usually a tracker/QC item, not a
  feature).
- **Story Points** = sum of `customfield_10032` across top-level `Story` issues (and `Task` if
  it has points — but Tasks usually don't count).
- **Contributor** = the **unique** set of assignees across every Story/Bug/Sub-task issue
  (excluding pure Task trackers), in order of first appearance (not alphabetical). Fetch the
  `assignee` field separately — the Step 2 JQL fields don't include it.
- **Status**: if 100% of issues are Done → `Done` badge, color **green**. If some issues aren't
  Done yet (e.g. still `READY TO RELEASE`) → use that exact status text, color **blue** (to
  distinguish it from green Done without inventing a color outside Confluence's palette).
- **Link**: if the version has its own Confluence release-notes page (usually named
  "Release Notes Sprint NN - ddmmyy" — find it via `searchConfluenceUsingCql` title contains
  "Sprint" in the same space) → link as
  `https://goopay.atlassian.net/wiki/pages/viewpage.action?pageId=<id>`.
  If no page exists yet → link straight to a representative Jira issue.

**B. "Hybrid hotfix" row** (version has only `Task`-type issues, usually `parent = GP2-9`,
description is a single plain paragraph rather than a bullet list, name like
"Hot fix - dd/mm/yyyy") — this is a Phase-1-style carryover. Handle it **exactly like the
existing Hot fix rows** in the table:
- **Release**: display name in red `<span style="color: #de350b">Hot fix - dd/mm/yyyy</span>`.
- **Feature** = 1 (no leaf-bullet counting since the description has no bullets).
- **Story Points** = `—`, **Contributor** = `—` (leave unpopulated, matching the convention for
  the other Phase-1 hotfix rows).
- **Link**: straight to the Jira issue (no dedicated release-notes page for this type).

**Feature progress bar** (applies to both row types): width 10 characters, scale anchor **23**
(historical max, from GP2-2574):
```
filled = round(feature_count / 23 * 10), but minimum 1 if feature_count > 0
bar = "█" * filled + "░" * (10 - filled)
```

**Contributor badge colors**: cycle in order of first appearance **within that row** (not
globally per person): `red, yellow, blue, neutral, purple, green`, repeating if there are more
than 6 people. This is purely decorative — colors don't need to stay consistent for the same
person across different rows.

### Step 5 — Build & push, following the anti-corruption protocol

The tbody has previously ended up silently empty from hand-typing/paraphrasing long HTML —
**don't repeat that**:

1. Save the HTML fetched in Step 1 to a scratchpad file **verbatim** (copy exactly, never
   retype).
2. Verify `grep -c '<tr>'` == `grep -c '</tr>'` on that file before changing anything.
3. Use a **Python script** (not manual character-by-character edits) to replace/insert the new
   `<tr>...</tr>` rows in the right place (newest at the top of tbody), based on
   `content.replace(old_row_string, new_rows)`.
4. After building, re-verify `<tr>` == `</tr>` on the new file, **Read the entire file in one
   call** (no offset/limit), and copy that exact output verbatim into the `body` of
   `updateConfluencePage(contentFormat="html")`.
5. After pushing, **always** call `getConfluencePage` again to confirm: the version number
   incremented by exactly 1, the tbody isn't empty, the row count matches expectations, and the
   new/changed rows render correctly.

### Step 6 — Report back to the user

Give a short summary: which version(s) were added, which were relabeled/recomputed (and why,
e.g. a Jira rename), which future version(s) were detected but not added to the table. No need
to re-explain the whole process — only ask if a case genuinely doesn't fit either pattern A or
B above.

## Things that don't need re-confirming every time (already agreed with the user)

- The "Phase 1 stats" table (section 2) is a fixed snapshot of Phase 1 — it does **not** get
  updated when a new Phase 2 release is added.
- The Feature column in section 3's table consistently uses the word **"feature"** (not "tính
  năng"), even though section 2 still uses "tính năng" since that's an old snapshot that stays
  unchanged.
- No explanatory paragraph describing what each column means under the section 3 heading (the
  user explicitly asked to remove that earlier).
- A version that isn't really released yet (`released:false`, no `releaseDate`) → don't add it
  to the table, just mention it.
