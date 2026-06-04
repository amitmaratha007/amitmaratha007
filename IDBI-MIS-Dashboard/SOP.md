# SOP — IDBI MIS Work Command Center

Standard Operating Procedure for running and using the dashboard. Keep this file in the dashboard folder.

---

## A. One-time setup (host PC — the computer that "runs" the dashboard)

You need **Node.js** (portable/extracted is fine — no admin install required).

**Option 1 — Portable Node (recommended, no install):**
1. Download "Windows Binary (.zip)" from nodejs.org and extract it.
2. Copy the extracted folder next to `Start-Dashboard.bat` and **rename it to `node`** (so `node\node.exe` exists). Done.
   - *Or* keep it anywhere and create a file `node-path.txt` next to the `.bat` containing the full path to `node.exe` (or its folder), e.g. `D:\tools\node-v20.11.0-win-x64`.
   - *Or* set it later inside the app: **⚙ Settings → Portable Node.js path** (saves it for next launch).

**Option 2 — Installed Node or Python:** if either is installed system-wide, the `.bat` finds it automatically.

## B. Start it
1. Double-click **`Start-Dashboard.bat`**.
2. It opens Chrome at **http://localhost:8080** and shows a **network link** (e.g. `http://10.123.210.37:8080`).
3. If Windows asks about the firewall the first time → click **Allow access**.
4. **Keep the black window open** while the team uses it. Close it to stop.

## C. Give the team access
- Share the **network link** with colleagues on the same office LAN.
- As Superadmin (the host PC is admin automatically): **⚙ Settings → Map IP → person** — type each person's office IP next to their name (their current IP shows at the top of the box when *they* open it, or use `ipconfig`).
- After mapping, each person sees & edits **their own work + people reporting to them**. Unmapped users get **view-only**.
- Want everyone to simply view everything? **⚙ Settings → "Give everyone view-only access"**.

## D. Daily use
- **Overview** = drill-down pie. Click a manager slice → their team; click a person → their works; click the centre to go up.
- **Work Pendency** = full team work list. Turn on **✎ Edit** to add/update/complete items (KPIs are clickable filters).
- **Team Directory / Org Chart** = contacts & reporting structure (Org Chart is editable as Superadmin with ✎ Edit on).
- **Your workspace** (click your own profile card) = your projects, sub-tabs (Firewall CRF / i-Smart / Service / Change), Projects (UAT/Go-Live), Tasks, and **custom Trackers** (e.g. Regulatory Returns). Use **＋ Tracker** to build your own with custom columns.
- **Search** (Ctrl + K) finds requests, work items, people, projects, tasks.

## E. Backups (do this!)
- **⚙ Settings → Backup folders**: set a **local** path (e.g. `D:\MIS-Backups`) and a **shared** path (e.g. `\\server\share\MIS`). Tick **Auto-sync** to copy on every change.
- Click **💾** (sidebar) any time to write a timestamped copy to both folders + the local `backups\` folder.
- **⬇ Backup** downloads a full `.json`; **⬆ Restore** loads one back. Use these to move data between PCs.
- Personal workspace data is stored in **your browser** — always open the dashboard from the **same PC / same link** to see it.

## F. Themes
Sidebar theme menu: **Dark, Light, IDBI Dark, IDBI Light, GenZ Pink**, plus accent colour dots.

## G. Troubleshooting
- **Page looks broken after an update** → hard refresh: **Ctrl + Shift + R**.
- **"node not found"** → see section A (portable node / `node-path.txt`).
- **Network link doesn't open for others** → allow it through Windows Firewall (Private networks).
- **Port 8080 busy** → edit `set "PORT=8080"` in `Start-Dashboard.bat`.

---
*Default Superadmin password: `1234` — change it in ⚙ Settings. IP login is a LAN convenience gate, not strong security.*
