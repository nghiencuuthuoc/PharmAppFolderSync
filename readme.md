# PharmAppFolderSync

A lightweight, GUI-based folder synchronization tool (SRC → DST) with profile management and symlink-aware behavior.  
Built with Python + Tkinter/ttk using the PharmApp 2026 theme.

> **Primary goal:** keep two folders consistent (backup / mirror / sync) in a simple, auditable workflow—without requiring command-line tools.

---

## Highlights

- **One-way sync (SRC → DST)** with optional **Mirror** mode (delete extra items in DST).
- **Symlink-aware** behavior:
  - **preserve**: replicate symlinks as symlinks
  - **follow**: copy the contents behind symlinks (dereference)
  - **skip**: ignore symlinks
- **Profiles (MVP)**: save multiple configurations and switch quickly.
- **Dry-run** support: simulate actions without writing to disk.
- **Checksum mode (SHA-256)** for stronger change detection.
- **Exclude patterns** (one per line) using wildcard matching (glob-like).
- **Logging** to a file (per-profile or per-run), plus live GUI log.
- Packaging-friendly: works well as **single-file EXE** (PyInstaller `--onefile`) and can be packaged for macOS.

---

## What this tool is (and is not)

### Intended uses
- Project folder backup (e.g., dev folder → backup drive)
- Keeping a “golden copy” of a folder updated
- Mirroring a folder structure with optional exclusions

### Not intended as
- A two-way merge/sync tool (no conflict resolution)
- A version control system
- A network replication system with remote diffing

It is a **deterministic one-way sync**: SRC is the source of truth.

---


## Screenshot

![Screenshot](docs/screenshot.png)


---

## Installation / Download

### Option A — Use the prebuilt EXE (Windows)
1. Download the latest `.exe` from **Releases** (if provided in this repo).
2. Place the executable anywhere you like.
3. Ensure the icon assets are included only if your build expects them (see Packaging notes below).

### Option B — Run from source (Python)
1. Clone this repo:
   ```bash
   git clone https://github.com/nghiencuuthuoc/PharmAppFolderSync
   cd PharmAppFolderSync
   ```
2. Create and activate a virtual environment:
   ```bash
   python -m venv .venv
   # Windows
   .venv\Scripts\activate
   # macOS/Linux
   source .venv/bin/activate
   ```
3. Run:
   ```bash
   python folder_sync_gui_v1.py
   ```

---

## Quick Start (GUI)

1. **Profile**
   - Select an existing profile from the dropdown, or click **New**.
2. **SRC**
   - Choose the source folder (the folder you want to copy from).
3. **DST**
   - Choose the destination folder (the folder you want to update).
4. Choose your options:
   - **Symlink mode**
   - **Mirror**
   - **Dry-run**
   - **Checksum**
   - **Exclude patterns**
   - **Log file**
5. Click **Start Sync**.

---

## Profiles (how they work)

Profiles let you save multiple sync configurations (e.g., “Project backup”, “Media mirror”, “Docs archive”).

### Profile buttons
- **New**: create a new profile
- **Save**: overwrite the current profile with the current settings
- **Save As**: save current settings into a new profile name
- **Delete**: remove a profile
- **Import / Export**: move profiles between computers (JSON)

### Where profiles are stored (per user)
The app stores profiles in the OS user configuration directory so it works well with “onefile” EXEs.

- **Windows:** `%APPDATA%\PharmAppFolderSync\profiles.json`
- **macOS:** `~/Library/Application Support/PharmAppFolderSync/profiles.json`
- **Linux:** `~/.config/PharmAppFolderSync/profiles.json`

You can open this folder from the GUI using **Profiles Dir**.

---

## Sync Options Explained

### Symlink mode

#### 1) `preserve`
- DST will contain symlinks corresponding to SRC symlinks.
- Good when you want DST to remain “structurally identical” to SRC.
- **Windows note:** creating symlinks may require Administrator privileges or Windows Developer Mode.

#### 2) `follow` (copy symlink targets)
- The tool walks into symlinked directories and copies their real contents into DST.
- DST will contain normal files/folders (not symlinks).
- Use this when you want a “real data copy” even if SRC contains symlinks.

**Important:** if a symlink points outside the SRC tree, `follow` will copy from that external location as well. Use excludes to control scope.

#### 3) `skip`
- Ignore symlinks.
- Useful to avoid external folders or risky loops.

### Mirror (delete extra in DST)
- When enabled, the tool will delete files/folders in DST that do not exist in SRC (after applying excludes).
- Use with care—this is intentionally destructive in order to mirror SRC.

### Dry-run
- Simulates actions without writing changes.
- Recommended before enabling **Mirror** or before large sync operations.

### Checksum (SHA-256)
- Stronger detection for file changes (compares file hashes).
- Slower on large datasets.
- Good for auditing and ensuring correctness across unreliable filesystems.

### Exclude patterns
Exclude patterns are matched against **relative POSIX-style paths** (with `/` separators), using wildcard matching.

Examples:
```text
**/.git/**
**/__pycache__/**
**/*.tmp
**/node_modules/**
**/dist/**
**/*.log
```

Tips:
- One pattern per line
- Lines starting with `#` are treated as comments
- Use `**` to match any depth

---

## Logging

- You can set a **Log file** path manually.
- If left empty, the app may default to:
  - `DST/sync.log`

Logs include:
- Start/end summary
- Actions (COPY/UPDATE/SKIP/DELETE)
- Warnings and errors

The GUI also shows a live log and a recent-actions table.

---

## Safety Notes (recommended reading)

1. **Mirror deletes data**
   - Always use **Dry-run** first if you are unsure.
2. **Symlink follow can expand scope**
   - A symlink may point outside SRC; `follow` will copy from that target.
3. **Large folders**
   - For very large trees, consider starting without checksum and enabling it only for audits.
4. **Permissions**
   - Some files may fail to copy due to ACL/permissions; check the log for errors.

---

## Build a Single-File EXE (Windows)

### Requirements
- Python 3.10+ recommended
- PyInstaller

Install PyInstaller:
```bash
pip install pyinstaller
```

Build one-file GUI EXE:
```bash
pyinstaller --noconfirm --clean --onefile --windowed \
  --name PharmAppFolderSync \
  --icon "./nct_logo.ico" \
  --add-data "./nct_logo.png;." \
  folder_sync_gui_v1.py
```

Output:
- `dist/PharmAppFolderSync.exe`

---

## macOS Packaging Notes

You can package with PyInstaller on macOS as well, but:
- Consider using an `.icns` icon for the app bundle
- Use `--add-data` for `nct_logo.png` so the window icon works
- Test symlink behavior under macOS permissions and Gatekeeper constraints

Example:
```bash
pyinstaller --noconfirm --clean --windowed \
  --name PharmAppFolderSync \
  --add-data "./nct_logo.png:." \
  folder_sync_gui_v1.py
```

---

## Recommended Workflow (best practice)

1. Create a profile (e.g., `pa_sync`)
2. Set **SRC** and **DST**
3. Add excludes (at least: `.git`, `__pycache__`, build folders)
4. Run **Dry-run** once
5. Disable Dry-run, run real sync
6. Enable **Mirror** only when you truly need DST to be an exact mirror

---

## Roadmap Ideas (optional)

- Run queue (run multiple profiles sequentially)
- Hotkeys (Ctrl+1..9 to run profiles)
- “Follow dirs only” mode (follow symlinked directories but preserve symlinked files)
- Better reporting (CSV summary, HTML report)
- Two-way sync mode (with conflict resolution) — more complex by design

---

## Donate / Support

If this project helps your work, you can support the ecosystem:

- Donate NCT: https://www.nghiencuuthuoc.com/p/donate.html  
- Donate PharmApp: https://www.pharmapp.vn/Donate

---

## License

See the `LICENSE` file in this repository.

---

## Author / Ecosystem

- Nghiên Cứu Thuốc: https://www.nghiencuuthuoc.com  
- PharmApp: https://www.pharmapp.vn
