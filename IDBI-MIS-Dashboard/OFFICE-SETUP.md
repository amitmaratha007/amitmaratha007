# Office Setup — first run on a new PC

How to make **Start-Dashboard.bat** open the dashboard on an office computer, including
**exactly where to paste the Node.js path**. Do this once per PC.

---

## Step 0 — do you even need a path? (30‑second check)

The launcher can run on **Node.js** *or* **Python**. If either is already installed on the PC,
**you don't need to paste any path** — just double‑click `Start-Dashboard.bat`.

To check: open **Command Prompt** (press `Win`, type `cmd`, Enter) and type each of these:
```
node -v
python -v
```
- If **`node -v`** or **`python -v`** prints a version number → you're done, skip to **Step 3**.
- If both say *"not recognized"* → you need portable Node. Continue to **Step 1**.

---

## Step 1 — get portable Node.js (no admin install)

On a PC **with internet** (or ask IT):
1. Go to **nodejs.org → Downloads**.
2. Download the **Windows Binary (.zip)**, **64‑bit** (file looks like
   `node-v20.x.x-win-x64.zip`). *Not* the .msi installer — the **.zip** needs no admin rights.
3. Copy that .zip to the office PC (USB / shared drive).
4. **Right‑click the .zip → Extract All…** You'll get a folder like `node-v20.x.x-win-x64`
   that contains **`node.exe`**.

---

## Step 2 — tell the dashboard where Node is (pick ONE way)

The dashboard folder (the one with `Start-Dashboard.bat`) is your working folder. Pick **any one**
of these three ways — the launcher checks them in this order:

### ✅ Way A — rename the folder to `node` (easiest, no typing)
Put the extracted Node folder **next to `Start-Dashboard.bat`** and **rename it to exactly
`node`**, so that this path exists:
```
…\IDBI-MIS-Dashboard\node\node.exe
```
Your folder should look like:
```
IDBI-MIS-Dashboard\
├─ Start-Dashboard.bat
├─ server.js   server.py   index.html   data.json   assets\
└─ node\
    └─ node.exe        ← this is what matters
```
Done — go to **Step 3**. (No path to paste anywhere.)

### ✅ Way B — paste the path into `node-path.txt`  ← *this is the "where to paste the path"*
1. In the dashboard folder (next to `Start-Dashboard.bat`), create a plain text file named
   **exactly `node-path.txt`**.
   - Right‑click empty space → **New → Text Document**, then rename it to `node-path.txt`.
   - ⚠️ Windows hides extensions — make sure it is **`node-path.txt`**, **not**
     `node-path.txt.txt`. (View tab → tick **File name extensions** to be sure.)
2. Open it in Notepad and paste **one line** — either the **folder** that contains `node.exe`
   **or** the full path to `node.exe`. Both work:
   ```
   D:\Tools\node-v20.11.0-win-x64
   ```
   *…or…*
   ```
   D:\Tools\node-v20.11.0-win-x64\node.exe
   ```
3. Save. **Important:** **no quotes**, no extra spaces, nothing else on the line.
   - Tip to copy a path: hold **Shift**, right‑click `node.exe` → **Copy as path**, paste,
     then **delete the surrounding `"` quotes** that Windows adds.

Done — go to **Step 3**.

### ✅ Way C — set it inside the app (for next time)
If the app is already open once (via Way A/B or Python), go to **⚙ Settings → "Portable Node.js
path"**, paste the same path, Save. The app writes it into `node-path.txt` for you, so the **next**
launch finds Node automatically. (This can't be the very first step, because the app has to be
running to show Settings.)

---

## Step 3 — run it

1. **Double‑click `Start-Dashboard.bat`.**
2. A black window opens and prints:
   - `Runtime detected : node` (or `python`) and the Node path it used — good signs.
   - Two links: `http://localhost:8080` (this PC) and `http://<your-IP>:8080` (for the team).
3. Chrome opens the dashboard automatically. *(If Chrome isn't installed it opens your default
   browser — that's fine.)*
4. If Windows shows a **firewall** popup the first time → click **Allow access** (needed only so
   teammates on the office LAN can open the network link).
5. **Keep the black window open** while anyone uses the dashboard. Closing it stops the server.

If you instead see **`ERROR: Node.js was not found`**, the path isn't right yet — re‑check Step 2
(usually the `node` folder isn't named exactly `node`, or `node-path.txt` has quotes / a wrong
name / `.txt.txt`).

---

## Other prerequisites & paths (the full list)

| Thing | Needed? | What to do |
|---|---|---|
| **Node.js path** | Only if Node/Python aren't already on the PC | Step 2 (Way A / B / C). |
| **Python** (alternative to Node) | Optional | If `python -v` works in cmd, the launcher uses it automatically — **no path needed**. Portable Python only works if `python` is on the system PATH. |
| **Google Chrome** | Optional | Auto‑detected in the 3 normal install locations. If not found, the default browser is used. No path to set. |
| **Port 8080 free** | Yes | If another program uses 8080, edit `Start-Dashboard.bat`, change `set "PORT=8080"` to e.g. `8090`, save, and use that number in the link. |
| **Firewall** | Only for team/LAN access | Click **Allow access** on first run (Private networks). Not needed if only this PC opens it. |
| **Same PC / same link** | Yes (for data) | Each person's personal workspace + theme are saved in **their browser**; the shared team data lives in `data.json` on the **host PC**. Always open from the **same PC and same link** so your data is there. Use **⬇ Backup / ⬆ Restore** to move data between PCs. |

---

## Quick checklist for a new office PC
1. `node -v` or `python -v` works in cmd? → if yes, just run the .bat (skip 2–3 below).
2. Else: extract portable Node, then **either** rename its folder to `node` next to the .bat
   (Way A) **or** put its path in `node-path.txt` (Way B).
3. Double‑click `Start-Dashboard.bat` → it shows `Runtime detected` → Chrome opens the dashboard.
4. Click **Allow access** on the firewall popup (only if sharing on the LAN).
5. Keep the black window open. Bookmark `http://localhost:8080`.

> Tip: once one PC is set up as the **host**, teammates usually **don't** install anything — they
> just open the **network link** (`http://<host-IP>:8080`) the host window shows. Only the host PC
> needs Node/Python.
