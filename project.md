# 🧩 Product Requirements Document (PRD) – SnapBacked (Windows MVP)
**Version:** 1.2  
**Owner:** Sander Lund Ejdesgaard  
**Date:** October 2025  

---

## 🧭 1. Product Overview

**Product Name:** SnapBacked  
**Tagline:** *Get your Snapchat memories backed up - with Snapbacked*  
**Platform:** Windows Desktop (Windows 10 and 11)  
**Preferred Tech:** Electron (TypeScript + React) or Tauri (Rust + React)

### Purpose
SnapBacked helps users easily **import, organize, and relive** their Snapchat memories — with a **ZIP-first** workflow that mirrors Snapchat’s export process. It processes Snapchat export ZIPs (and optionally folders or individual media), cleans and structures them, and displays a local gallery with “On This Day” flashbacks — completely offline and private.

> **Note:** The detailed UX steps live in `UserFlow_and_ProcessingFlow.md`. This PRD references that document and does not duplicate it.

### MVP Objective
Enable a user to:
1. Drag-and-drop a **Snapchat export ZIP** (primary path), and optionally **folders or individual media files** (MP4, MOV, JPG, JPEG, PNG, HEIC).  
2. Automatically extract, parse, convert (if needed), and organize media with correct metadata dates.  
3. Browse a simple, fast gallery.  
4. View “On This Day” flashbacks on startup.

---

## ⚙️ 2. MVP Scope

### In Scope
- ✅ Drag-and-drop **ZIP** imports (primary), plus **folder** or **file** imports (secondary)  
- ✅ Extraction and recursive folder scanning  
- ✅ Multi-format media support (ZIP, MP4, MOV, JPG, JPEG, PNG, HEIC)  
- ✅ Automatic format conversion and cleanup  
- ✅ Image/video thumbnail generation  
- ✅ Basic gallery view (grid layout)  
- ✅ “On This Day” flashback reminders  
- ✅ Local SQLite database for indexing  
- ✅ Local-only storage (no cloud)  
- ✅ Basic settings (theme, notifications)

### Out of Scope (Future Versions)
- ❌ Cloud sync (Google Drive, Dropbox, etc.)  
- ❌ AI highlight videos  
- ❌ macOS/mobile versions  
- ❌ Social sharing  
- ❌ Payments or Pro model  

---

## 🧱 3. Functional Requirements

### 3.1 Import & File Handling
**Goal:** Seamlessly import Snapchat export ZIPs and normalize media for a local gallery.

- **FR1 (Inputs):** User can drag and drop:
  - A **`.zip` Snapchat export** (primary)  
  - A **folder** containing mixed media  
  - Individual files: `.mp4`, `.mov`, `.jpg`, `.jpeg`, `.png`, `.heic`

- **FR2 (ZIP Handling):**
  - Extract ZIP to a staging folder, then organize to:
    ```
    Documents/SnapBacked/Imports/{username}_{date}
    ```
  - Recognize typical Snapchat export structure (e.g., `/html`, `/json`, `memories_history.html`, `memories_history.json`, `index.html`) and **parse metadata/links** where applicable.

- **FR3 (Downloader Integration):**
  - **Reuse the logic from “Snapchat-All-Memories-Downloader”** (vendored submodule or port):
    - Download assets referenced in the export (if present).
    - Preserve/restore **original creation dates** where possible.
  - Provide a **Node↔Python bridge** (Electron) or **Rust FFI/sidecar** (Tauri) to invoke the downloader pipeline.

- **FR4 (File Discovery & Types):**
  - Recursive discovery for folders and extracted ZIP contents.
  - Handle **extensionless image files** (reported as type “File” in exports): infer type via **magic bytes** and rename (e.g., `.jpg`).
  - Handle **nested/bundled ZIPs** per memory item; extract associated **MP4** and **PNG overlay** and keep their association.

- **FR5 (Duplicates & Integrity):**
  - De-dupe via **SHA-256** hash.
  - Detect **suspect MP4s** (e.g., ≲ ~60 KB) and mark as **error/corrupt** with logs and user notice in the import report.

- **FR6 (Conversion & Thumbnails):**
  - Auto-convert:
    - **HEIC → JPG**
    - **MOV → MP4**
  - Generate thumbnails for images and videos.

- **FR7 (Metadata Normalization):**
  - Persist normalized metadata to SQLite: filename, path, type, `date_created`, hash, source type, converted, thumbnail path.
  - If OS file `created/modified` reflects **download time**, correct `date_created` using (in priority order):
    1) EXIF,
    2) Snapchat JSON/HTML timestamps,
    3) filename date,
    4) file system timestamps (fallback).

- **FR8 (Progress & Completion):**
  - Show an import progress bar and queue.
  - On success, display the exact message:
    > **“Your Snapchat data has successfully and safely been downloaded. Press Next to continue”**  
    And render a **“Next”** button.

- **FR9 (DB Persistence):**
  - Use SQLite for all metadata; operations must be resilient to app restarts during import.

---

### 3.2 Gallery View
**Goal:** Let users browse all imported media intuitively.

- **FR10:** Display a responsive grid of image/video thumbnails with date labels.  
- **FR11:** Clicking a thumbnail opens fullscreen preview (lightbox).  
- **FR12:** Sorting options (Newest → Oldest, Year, File Type).  
- **FR13:** Optional search by filename, date, or media type.  
- **FR14:** Visual tag for source (e.g., “📦 ZIP Import” / “📁 Folder Import”).  
- **FR15:** Keyboard navigation (arrow keys to browse).

---

### 3.3 “On This Day” Flashbacks
**Goal:** Deliver nostalgic reminders from past years.

- **FR16:** On startup, query DB for media from the same date (±2 days) across past years.  
- **FR17:** If matches exist, show popup:
  - “🎞 You have _n_ memories from this day in _YYYY_!”  
- **FR18:** Clicking opens a mini-gallery (carousel).  
- **FR19:** Optional Windows desktop notification (toggleable in Settings).

---

### 3.4 Settings
**Goal:** Give users control over import and viewing preferences.

- **FR20:** Toggle “Show On This Day flashbacks.”  
- **FR21:** Choose default import folder.  
- **FR22:** Select file types to include by default (e.g., exclude screenshots).  
- **FR23:** Light/Dark mode toggle.  
- **FR24:** View app version and feedback link.  
- **FR25 (Guide/Reminder Controls):**
  - Toggle **“Show pre-import guide on startup”** (enabled by default until first successful import).
  - Toggle **“Email reminder after export request”** with configurable delay window; uses Windows Toast.

---

### 3.5 Pre-Import Guide & Reminders (from User Flow)
**Goal:** Mirror the pre-app steps (not performed by the app) and reduce friction.

- **FR26:** On first run (and when enabled), show a **modal guide** that visually explains how to request data on Snapchat (steps 1–3 from `UserFlow_and_ProcessingFlow.md`) with a CTA: **“I’m ready to continue”**.
- **FR27:** The guide includes:
  - **Export expiry note** (typically ~3 days) and a reminder that expiry appears in Snapchat under “Your exports”.
  - A **suggestion to start with “Last Week”** as the date range to get a small, fast test export.
- **FR28:** After the guide, optionally schedule a **Windows toast reminder** to check for the email **“Your Snapchat data is ready for download”**.

---

## 🧰 4. Technical Requirements

| Component | Description | Suggested Tools |
|---|---|---|
| **Framework** | Desktop runtime | Electron or Tauri |
| **Frontend** | UI layer | React + Tailwind CSS |
| **Database** | Local metadata | SQLite (via Prisma or better-sqlite3) |
| **File System** | Unzip & recursive scan | Node.js `fs-extra`, `adm-zip` |
| **File Type Detection** | MIME + magic bytes; handle extensionless | `file-type`, `mime-types`, low-level magic-byte check |
| **Media Tools** | Conversion & metadata | FFmpeg, ExifTool, Sharp |
| **Downloader Bridge** | Invoke Snapchat downloader pipeline | Electron: Node↔Python (`child_process`); Tauri: sidecar/FFI |
| **Notifications** | Flashback + reminders | Windows Toast API |
| **Installer** | Build packaging | Electron Builder or NSIS |
| **Storage Path** | User data | `%AppData%/SnapBacked/` |
| **Imports Path** | Default organized media | `Documents/SnapBacked/Imports/{username}_{date}` |

**Notes:**
- Vendor or port **Snapchat-All-Memories-Downloader** logic; expose as a callable pipeline.
- Implement **nested ZIP** extraction and overlay PNG association.
- Robust error logging for suspect/corrupt files; produce an import report.

---

## 🎨 5. User Interface Requirements

### 5.1 Navigation (Tabs)
- Bottom **3-tab** navigation (icons + labels, left→right):  
  **1) Memories  2) Gallery  3) Profile**
- Tabs persist across the app; default to **Memories** after a successful import, otherwise **Import (Home)**.

### 5.2 Home / Import Screen
- Large drag-and-drop zone (accepts **ZIP** primarily; folders/files also supported).  
- Secondary button: **“Select Folder or File”**.  
- Progress area showing extraction/conversion/indexing.  
- Display the **completion message** and **“Next”** button verbatim:  
  > “Your Snapchat data has successfully and safely been downloaded. Press Next to continue”
- After **Next**, route to **Gallery** (and show **Memories** when relevant).

### 5.3 Gallery View
- Responsive grid (3–6 columns).  
- Sidebar filters: **Year | Month | File Type | Search**.  
- Top bar: **Sort by Date**, **Import More**, **Settings**.  
- Source pill (e.g., “📦 ZIP Import”).  
- Double-click → fullscreen preview.

### 5.4 Memories (Flashbacks)
- Modal popup on matches with carousel and **Play All** option.  
- “View in Gallery” shortcut.

### 5.5 Profile
- Theme toggle, import defaults, flashback/reminder toggles, app info.

---

## 🧩 6. Data Model

**Table: `media_files`**

| Field | Type | Description |
|---|---|---|
| id | INTEGER (PK) | Unique ID |
| filename | TEXT | Original file name |
| file_path | TEXT | Absolute local path |
| file_type | TEXT | “image” or “video” |
| source_type | TEXT | “zip”, “folder”, or “file” |
| date_created | DATETIME | From EXIF/JSON/filename (normalized) |
| imported_at | DATETIME | Import timestamp |
| hash | TEXT | SHA-256 hash for duplicates |
| converted | BOOLEAN | Whether format was converted |
| thumbnail_path | TEXT | Cached thumbnail location |
| overlay_path | TEXT | Optional associated overlay (PNG) |
| import_job_id | TEXT | Batch/link back to import report |

---

## 🧠 7. User Flow
- The canonical step-by-step user flow is maintained in **`UserFlow_and_ProcessingFlow.md`** and referenced by the app’s **Pre-Import Guide** modal. This PRD defines implementation-level behavior and constraints.

---

## 🚀 8. Success Metrics

- ✅ User imports a Snapchat **ZIP** end-to-end without errors.  
- ✅ Gallery loads >1,000 items smoothly (<2s).  
- ✅ **Dates normalized correctly** (no “download time” pollution).  
- ✅ Flashbacks match correct dates from metadata.  
- ✅ No duplicate media (hash-based).  
- ✅ Runs 100% offline.  

---

## 🔮 9. Future Extensions (Post-MVP)

- Cloud sync (Google Drive, OneDrive).  
- AI-curated highlight videos and yearly summaries.  
- macOS and mobile versions.  
- “Pro” tier (e.g., merge overlays with base media).  
- AI face clustering and smart search.

---

## 🧩 10. Development Notes for AI Agent (important)

- Modules: `import`, `downloader`, `conversion`, `gallery`, `flashback`, `settings`.  
- The **pre-import guide** content is sourced from `UserFlow_and_ProcessingFlow.md`.  
- Importer accepts ZIPs, folders, and file arrays; **ZIP is the primary flow**.  
- Async everything; batch conversions; resilient to restarts.  
- Parse Snapchat HTML/JSON for timestamps and asset links.  
- If vendoring **Snapchat-All-Memories-Downloader**, wrap it with a stable CLI/IPC; otherwise port core logic to Node/Rust.  
- Log aggressively (conversion failures, suspect MP4s, overlay associations, duplicates skipped) and emit an **import report**.  
- Test with mixed datasets (ZIP + MP4 + JPEG + PNG + HEIC + extensionless images + nested ZIPs).  

---

**End of PRD**
