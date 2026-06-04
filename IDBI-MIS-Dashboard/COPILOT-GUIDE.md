# Copilot Guide — safely changing the IDBI MIS Dashboard

This file is for **GitHub Copilot** (or any AI assistant) and for **you**. It lists the safe,
copy‑paste recipes for the changes you'll actually ask for — change a theme, add a task or a
custom tab for a specific person, add people/links/contacts — **without breaking the dashboard**.

> **How to use it with Copilot:** open this file and the file the recipe points to, then tell
> Copilot in plain English what you want, e.g. *"Using COPILOT‑GUIDE.md recipe R5, add a task
> 'Prepare GSTR‑9' due 2026‑07‑15 to Shivesh Kumar's workspace."* Ask for **one change at a
> time**, then run the **Verify** step before moving on.

---

## 0. Two ways to change things — try the app first

**Most changes need NO code.** The dashboard already lets you do these in the running app:

- **Change your theme / accent colour** → sidebar **theme dropdown** + colour dots (each person
  chooses their own).
- **Add your own tasks, projects, trackers, project tabs, requests, links, contacts** → open
  your workspace and use **＋ Add**, **＋ Project tab**, **＋ Tracker**, **⚙ Request types**.
- **Add / edit people, org chart, shared links, work items** → log in as Superadmin, turn on
  **✎ Edit**.

**Use Copilot (code/data edits) only for things the app can't do**, such as:
- changing the **default** theme everyone starts on, or **creating a new** theme / colours;
- adding tasks/tabs **into another person's workspace for them** (you can't do that from the UI —
  "view‑as" is read‑only);
- pre‑loading data, renaming built‑in labels, or layout tweaks.

---

## 1. GOLDEN RULES (Copilot must follow these)

1. **Offline only.** No new libraries, no `npm install`, no internet/CDN/Google Fonts. Plain
   HTML/CSS/JS + the Python/Node server. If a change needs a download, it's wrong.
2. **One change at a time**, then verify (section 9). Don't refactor unrelated code.
3. **Never rename or restructure the data keys** in `data.json` or the workspace objects
   (`workspaces`, `people`, `links`, `vendors`, `workItems`, `requests`, `tasks`, `trackers`,
   `trackedProjects`, `projects`, `reqTypes`). The app depends on these exact names.
4. **`data.json` must stay valid JSON.** No trailing commas, no comments, double‑quotes only.
5. **IDs must be unique and must match across files.** A person's `id` (e.g. `p-shivesh`) is the
   same in `people`, in `access.ipMap`, in `workspaces`, and in any `workItems.assignee`.
6. **Keep `server.js` and `server.py` identical in behaviour.** If you change one endpoint,
   change the other the same way.
7. **Don't touch these load‑bearing pieces** unless that's the whole point of the task:
   the per‑user workspace logic (`seedWorkspace`, `syncWorkspace`, `normalizeWorkspace`,
   `workspaceOwnerId`), the save routing (`savePersonal`→`/api/workspace`, `saveShared`→
   `/api/shared`, `saveTeam` which **must keep stripping** `workspaces`/`links`/`vendors`),
   the `!viewAs` read‑only guards, and `DATA_LOCK` in `server.py`.
8. **After editing any file in `assets/` or `index.html`, bump the cache number:** change every
   `?v=NN` in `index.html` to the next number (currently `?v=13` → use `?v=14`).
9. **Back up first.** Before editing `data.json`, copy it to `data.json.bak` (or click **⬇ Backup**
   in the app). See section 10 to restore.

---

## 2. The data model in 60 seconds

- **`data.json`** on the server is the single source of truth. It holds the shared team data
  **and** every person's private workspace under `workspaces`:
  ```
  data.json
  ├─ meta, access (incl. access.ipMap: which PC IP = which person)
  ├─ workTags[]            ← tags for team Work Pendency items
  ├─ people[]             ← the staff directory
  ├─ workItems[]          ← team "Work Pendency" list (has assignee)
  ├─ links[]   / vendors[]← SHARED links & tech contacts (everyone sees)
  └─ workspaces{ }        ← one entry PER PERSON, keyed by person id:
       "p-shivesh": { meta, projects[], reqTypes[], requests[], trackedProjects[],
                      tasks[], trackers[], links[], vendors[] }
  ```
- **Identity = IP.** A person sees their own workspace because their PC's IP is mapped to their
  `id` in `access.ipMap` (set via ⚙ Settings → Map IP → person). The host PC (`127.0.0.1`) is
  Superadmin and is mapped to `p-amit`.
- **Only `p-amit` is pre‑seeded** with the default project tabs (Oracle GL, ASP‑GSP, GST
  Litigation, Miscellaneous). Everyone else starts blank and builds their own.
- **Editing `data.json` takes effect on the next browser refresh** — no server restart needed
  (the server re‑reads the file each load). Tell the affected person to refresh, and don't edit
  their workspace in `data.json` while they're actively editing it in the app (their next save
  would overwrite your edit).

---

## 3. Recipe R1 — Change the DEFAULT theme everyone starts on

Themes are: `dark`, `light`, `idbi-dark`, `idbi-light`, `genz` (shown as "🌸 Pink").

**Edit 2 spots:**
1. `index.html`, line 2: `<html lang="en" data-theme="dark">` → change `dark` to e.g. `idbi-dark`.
2. `assets/app.js`, in `loadUI()` (search `UI.theme = UI.theme`): change `|| "dark"` to
   `|| "idbi-dark"`.

Then bump `?v=` (rule 8). *(This only affects people who haven't already picked a theme; their
own choice is remembered per browser.)*

---

## 4. Recipe R2 — Change an EXISTING theme's colours (e.g. the Pink theme)

**File:** `assets/style.css`. Search for the theme block, e.g. `[data-theme="genz"]` (Pink),
`[data-theme="idbi-dark"]`, or `[data-theme="idbi-light"]`.

Each block sets CSS variables. The ones that change the look most:
- `--text` / `--muted` / `--faint` — text colours.
- `--glass`, `--glass-2`, `--glass-3` — the frosted panel tints (use `rgba(r,g,b,alpha)`).
- `--brd`, `--brd-2` — border tints.
- `--wall` — the big background gradient (this drives the "vibrancy"; bump the alpha values up
  to make it more vivid, down to calm it).
- The matching `[data-theme="…"] { --surface: #…; }` line lower down = solid colour for menus.

Also update the theme's **accent** (buttons/active tabs) in `assets/app.js` →
`const THEME_ACCENT = { … }` (line 49). Tell Copilot the hex you want.

> Tip: IDBI brand colours = green `#12b766`, orange `#f26522`, white. Keep IDBI themes on those.

---

## 5. Recipe R3 — Add a BRAND‑NEW theme

Add the theme in **4 places** (use a lowercase id with no spaces, e.g. `ocean`):
1. `assets/style.css` — add a block `[data-theme="ocean"] { --text:…; --glass:…; --wall:…; … }`
   (copy an existing block and recolour), **and** a `[data-theme="ocean"] { --surface:#…; }` line.
2. `assets/app.js` — add `"ocean"` to `const THEMES = [ … ]` (line 48).
3. `assets/app.js` — add `ocean: "#hexAccent"` to `const THEME_ACCENT = { … }` (line 49).
4. `index.html` — add an option in the theme dropdown:
   `<option value="ocean">🌊 Ocean</option>` (next to the other `<option value="…">` lines).

Bump `?v=`. Verify the new name appears in the dropdown and switching to it recolours the app.

---

## 6. Recipe R4 — Change the accent colour dots

**File:** `index.html`. Search `data-action="accent"`. Each dot is one line:
`<span class="sw" data-action="accent" data-c="#0a84ff" style="background:#0a84ff"></span>`.
Change both the `data-c` and the `background` to the **same** new hex (they must match). Add or
remove `<span>` lines to add/remove dots. Bump `?v=`.

---

## 7. Recipe R5 — Add a TASK / project / tracker entry to a SPECIFIC person

Do this in **`data.json`** under that person's `workspaces["p-…"]`. Add an object to the right
array. **Give it a unique `id`** (any unique string, e.g. `t-shivesh-1`).

**Find the person's id** in `people[]` (e.g. Shivesh = `p-shivesh`). If that person has **no
`workspaces` entry yet**, create one first by copying this skeleton (blank, no Amit tabs):
```json
"p-shivesh": {
  "meta": { "owner": "Shivesh Kumar", "version": 5 },
  "projects": [], "reqTypes": [], "requests": [], "trackedProjects": [],
  "tasks": [], "trackers": [], "links": [], "vendors": []
}
```

**Add a task** → push into `tasks`:
```json
{ "id": "t-shivesh-1", "title": "Prepare GSTR-9", "project": "", "priority": "high", "due": "2026-07-15", "done": false }
```
- `priority`: `high` | `medium` | `low`. `project`: `""` (General) or the id of one of *their*
  `projects` tabs. `due`: `YYYY-MM-DD` (or `""`).

**Add a tracked project** → push into `trackedProjects`:
```json
{ "id": "tp-shivesh-1", "name": "ENachos Migration", "status": "UAT", "uat": "2026-08-01", "goLive": "2026-09-01", "owner": "Shivesh Kumar", "progress": 50, "notes": "" }
```
- `status`: `Requirement` | `Development` | `UAT` | `Pre-Prod` | `Go-Live` | `Live` | `On Hold` | `Closed`.

**Add a "Returns"‑style tracker** → push into `trackers`:
```json
{ "id": "tk-returns-shivesh", "name": "Regulatory Returns", "icon": "📑",
  "columns": [
    { "id": "name", "label": "Return name", "type": "text" },
    { "id": "due", "label": "Due date", "type": "date" },
    { "id": "status", "label": "Status", "type": "select", "options": ["Pending","Filed","Overdue"] }
  ],
  "rows": [ { "id": "row1", "values": { "name": "GSTR-3B May", "due": "2026-06-20", "status": "Pending" } } ] }
```
- Column `type`: `text` | `date` | `select` (with `options`). Each row's `values` keys must match
  the column `id`s.

Then: validate JSON, refresh that person's browser.

---

## 8. Recipe R6 — Add custom PROJECT TABS (and their sub‑types) for a person

In `data.json` → `workspaces["p-…"]`:

**Add a project tab** → push into `projects` (id must be a lowercase‑hyphen slug; it's referenced
by `requests.project` and `tasks.project`):
```json
{ "id": "regulatory", "name": "Regulatory Work", "icon": "🏛️", "color": "#12b766" }
```
**Add request sub‑types** (the sub‑tabs inside every project tab) → push into `reqTypes`:
```json
{ "id": "filing", "name": "Filing" }
```
**Add a request inside a tab** → push into `requests` (link it with `project` + `type` ids):
```json
{ "id": "r-shivesh-1", "project": "regulatory", "type": "filing", "ref": "REF-001",
  "title": "File monthly return", "raised": "2026-06-01", "deadline": "2026-06-20",
  "status": "pending", "owner": "Shivesh Kumar", "approver": "", "notes": "" }
```
- `status`: `raised` | `pending` | `in-progress` | `approved` | `completed` | `rejected` | `on-hold`.

> **Give someone Amit‑style tabs automatically (R7):** open `assets/app.js`, find `const AMIT_ID
> = "p-amit"`. Two options — (a) simplest & safest: just copy Amit's `projects`/`reqTypes`
> arrays from his `workspaces["p-amit"]` into the other person's workspace in `data.json`; or
> (b) change the seeding rule so the listed ids get defaults — tell Copilot: *"In seedWorkspace,
> seed default projects when ownerId is one of \[\"p-amit\",\"p-vivek\"], else empty."*

---

## 9. Recipe R8–R10 — People, shared links/contacts, team work items

- **R8 Add/edit a person** → easiest in the app (Superadmin → ✎ Edit → Team Directory/Org →
  ＋ Add person). Or in `data.json` `people[]`:
  ```json
  { "id": "p-newhire", "name": "New Person", "salutation": "Shri", "roleLevel": "Manager",
    "role": "Manager, MIS", "email": "x@idbi.co.in", "mobile": "", "pax": "", "location": "Mumbai", "reportsTo": "p-ankit" }
  ```
  `roleLevel`: `DGM` | `AGM` | `Manager` | `AM`. `reportsTo`: another person `id` or `null`.
  Then map their office PC: ⚙ Settings → Map IP → person (so they get their own workspace).
- **R9 Shared link / tech contact** → anyone can add these in the app (Links / Tech Contacts →
  ＋ Add, "Shared" section). Or in `data.json` `links[]` / `vendors[]`:
  ```json
  // links[]:   { "id": "l1", "title": "GST Portal", "url": "https://…", "category": "GST", "desc": "" }
  // vendors[]: { "id": "v3", "name": "Support", "org": "TCS", "role": "L1", "area": "Oracle GL", "email": "", "phone": "", "notes": "" }
  ```
- **R10 Team Work Pendency item** → app: Work Pendency → ＋ Add. Or `data.json` `workItems[]`:
  ```json
  { "id": "w7", "title": "Quarterly review", "tag": "misc", "assignee": "p-shivesh", "status": "pending", "priority": "medium", "due": "2026-07-01", "notes": "" }
  ```
  `tag` must be an id from `workTags[]`; `assignee` an id from `people[]`.

---

## 10. Recipe R11 — Rename a visible label

- **A theme's display name** (e.g. "🌸 Pink") → `index.html`, the `<option value="genz">…</option>`
  line. **Keep `value="genz"`** (the internal id) so saved preferences don't break — change only
  the text between the tags.
- **A nav tab / section heading wording** → it's usually a string in `assets/app.js`. Tell Copilot
  the exact current text to find and the new text. Don't change `data-action` / `data-tab` /
  `id` values — only the human‑readable words.

---

## 11. Verify after EVERY change (don't skip)

Open a terminal in the project folder and run:
```bash
node --check assets/app.js
node --check server.js
python -m py_compile server.py
node -e "JSON.parse(require('fs').readFileSync('data.json','utf8')); console.log('data.json OK')"
```
All four must succeed. Then:
1. (Re)start the dashboard with **Start‑Dashboard.bat** if it isn't running.
2. **Hard‑refresh** the browser: **Ctrl + Shift + R**.
3. Click around the area you changed; for a per‑person data edit, open/view‑as that person and
   confirm the new task/tab/etc. shows and nothing else disappeared.

If Copilot can't run these, ask it to *"double‑check the JSON is valid and the JS has balanced
brackets."*

---

## 12. If something breaks — restore

- **Data problem** (blank lists, "Save failed", JSON error): restore `data.json` from your
  `data.json.bak`, or from the newest file in the **`backups/`** folder, or use the app's
  **⬆ Restore** with a downloaded backup. Refresh.
- **Page looks broken after a code edit:** undo Copilot's last change (Ctrl+Z in the file), or
  ask Copilot to revert that file to the previous version, then hard‑refresh.
- **Golden rule:** if a change makes the four checks in section 11 fail, **revert it** — don't
  pile more edits on top.

---

## 13. Quick reference — allowed values & shapes

| Thing | Allowed values |
|---|---|
| Task / work‑item / tracker status | `pending`, `in-progress`, `completed` |
| Request status | `raised`, `pending`, `in-progress`, `approved`, `completed`, `rejected`, `on-hold` |
| Tracked‑project status | `Requirement`, `Development`, `UAT`, `Pre-Prod`, `Go-Live`, `Live`, `On Hold`, `Closed` |
| Priority | `high`, `medium`, `low` |
| Person `roleLevel` | `DGM`, `AGM`, `Manager`, `AM` |
| Tracker column `type` | `text`, `date`, `select` (with `options:[…]`) |
| Dates | `"YYYY-MM-DD"` or `""` |
| Theme ids | `dark`, `light`, `idbi-dark`, `idbi-light`, `genz` (+ any you add) |

**Key file map:** `index.html` (shell, themes dropdown, accent dots, `?v=` cache number) ·
`assets/app.js` (all behaviour: `THEMES`, `THEME_ACCENT`, `seedWorkspace`, save routing) ·
`assets/style.css` (all styling + theme colour blocks) · `data.json` (all data incl. per‑person
`workspaces`) · `server.js` / `server.py` (offline servers, keep in parity) · `backups/` (restore
points).
