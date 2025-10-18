# 🧩 Product Requirements Document (PRD) – SnapBacked (Windows MVP)
**Version:** 1.1  
**Owner:** Sander Lund Ejdesgaard  
**Date:** October 2025  

---

## 🧭 1. Product Overview

**Product Name:** SnapBacked  
**Tagline:** *Save it. Own it. Relive it.*  
**Platform:** Windows Desktop (Windows 10 and 11)  
**Preferred Tech:** Electron (TypeScript + React) or Tauri (Rust + React)

### Purpose
SnapBacked helps users easily **import, organize, and relive** their Snapchat memories — whether downloaded as ZIPs, individual videos, or image files.  
It processes mixed-format Snapchat exports, cleans and structures them, and displays a local gallery with “On This Day” flashbacks — completely offline and private.

### MVP Objective
Enable a user to:
1. Drag-and-drop **ZIP files, folders, or individual media files** (MP4, MOV, JPG, JPEG, PNG, HEIC).  
2. Automatically extract, convert (if needed), and organize media.  
3. Browse their memories via a simple gallery.  
4. View “On This Day” flashbacks on startup.  

---

## ⚙️ 2. MVP Scope

### In Scope
- ✅ Drag-and-drop **ZIP, folder, or individual file** imports  
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
**Goal:** Seamlessly import and process mixed media from Snapchat exports or local files.

- **FR1:** User can drag and drop:
  - A `.zip` file  
  - A folder containing mixed media  
  - Individual `.mp4`, `.mov`, `.jpg`, `.jpeg`, `.png`, or `.heic` files  
- **FR2:** The app identifies file type automatically and performs the correct operation:
  - If ZIP → extract to a temporary staging folder  
  - If Folder → recursively scan and collect supported files  
  - If individual files → process directly  
- **FR3:** Default extraction path:  
  `Documents/SnapBacked/Imports/{username}_{date}`  
- **FR4:** Handle duplicates via hash comparison (SHA-256).  
- **FR5:** Auto-convert unsupported or high-friction formats:
  - HEIC → JPG  
  - MOV → MP4  
- **FR6:** Generate thumbnails for both images and videos.  
- **FR7:** Save normalized metadata (filename, path, type, creation date, hash) to SQLite DB.  
- **FR8:** Allow batch imports and progress tracking (import queue).  

---

### 3.2 Gallery View
**Goal:** Let users browse all imported media intuitively.

- **FR9:** Display grid of image/video thumbnails with date labels.  
- **FR10:** Clicking a thumbnail opens fullscreen preview (lightbox modal).  
- **FR11:** Sorting options (Newest → Oldest, Year, File Type).  
- **FR12:** Optional search by filename, date, or media type.  
- **FR13:** Keyboard navigation (arrow keys to browse).  
- **FR14:** Show import source label (ZIP name or folder).  

---

### 3.3 “On This Day” Flashbacks
**Goal:** Deliver nostalgic reminders from past years.

- **FR15:** On startup, query DB for media from the same date (±2 days) across past years.  
- **FR16:** If matches exist, show popup:  
  “🎞 You have 4 memories from this day in 2018!”  
- **FR17:** Clicking opens a mini-gallery (carousel view).  
- **FR18:** Optional Windows desktop notification (toggleable in Settings).  

---

### 3.4 Settings
**Goal:** Give users control over import and viewing preferences.

- **FR19:** Toggle “Show On This Day flashbacks.”  
- **FR20:** Choose default import folder.  
- **FR21:** Select file types to include by default (e.g., exclude screenshots).  
- **FR22:** Light/Dark mode toggle.  
- **FR23:** View app version and feedback link.  

---

## 🧰 4. Technical Requirements

| Component | Description | Suggested Tools |
|------------|--------------|------------------|
| **Framework** | Desktop runtime | Electron or Tauri |
| **Frontend** | UI layer | React + Tailwind CSS |
| **Database** | Local metadata | SQLite (via Prisma or better-sqlite3) |
| **File System** | Unzip & recursive file scan | Node.js `fs-extra`, `adm-zip` |
| **File Type Detection** | Identify MIME and extension | `file-type`, `mime-types` |
| **Media Tools** | Conversion & metadata | FFmpeg, ExifTool, Sharp |
| **Notifications** | Flashback reminders | Windows Toast API |
| **Installer** | Build packaging | Electron Builder or NSIS |
| **Storage Path** | User data | `%AppData%/SnapBacked/` |

---

## 🎨 5. User Interface Requirements

### 5.1 Home / Import Screen
- Drag-and-drop zone accepts **ZIP, folders, or media files**.  
- Button: “Select Folder or File.”  
- Progress bar showing import and conversion status.  
- Queue list of current imports with file counts and completion %.
- “Open Gallery” button once import finishes.

### 5.2 Gallery View
- Responsive grid (3–6 columns).  
- Sidebar filters: Year | Month | File Type | Search.  
- Top bar: “Sort by Date,” “Settings,” “Import More.”  
- Double-click → fullscreen preview.  
- Visual tag for source (e.g., “📦 ZIP Import” or “📁 Folder Import”).

### 5.3 Flashback Popup
- Modal with carousel of matching “On This Day” items.  
- “Play All” slideshow option.  
- “View in Gallery” shortcut.  

### 5.4 Settings
- Toggles:  
  - Enable/disable flashbacks  
  - Include/exclude file types  
- Folder picker for import directory.  
- Light/Dark theme.  
- App info and changelog.

---

## 🧩 6. Data Model

**Table: `media_files`**

| Field | Type | Description |
|--------|------|--------------|
| id | INTEGER (PK) | Unique ID |
| filename | TEXT | Original file name |
| file_path | TEXT | Absolute local path |
| file_type | TEXT | “image” or “video” |
| source_type | TEXT | “zip”, “folder”, or “file” |
| date_created | DATETIME | Extracted from EXIF or metadata |
| imported_at | DATETIME | Import timestamp |
| hash | TEXT | SHA-256 hash for duplicates |
| converted | BOOLEAN | Whether format was converted |
| thumbnail_path | TEXT | Cached thumbnail location |

---

## 🧠 7. User Flow

1. Launch app → Home/import screen loads.  
2. User drags in ZIP, folder, or media files.  
3. App detects file types and begins extraction/processing.  
4. Conversion & indexing run automatically (progress shown).  
5. When done, “Open Gallery” button activates.  
6. User browses organized media.  
7. On future startups, “On This Day” runs and displays flashbacks.  

---

## 🚀 8. Success Metrics

- ✅ User can import any combination of ZIP, folder, or file types successfully.  
- ✅ Conversion and indexing complete without manual steps.  
- ✅ Gallery loads >1,000 items smoothly (<2s).  
- ✅ Flashbacks match correct dates from metadata.  
- ✅ Application runs 100% offline.  
- ✅ No data duplication — hashes are unique.  

---

## 🔮 9. Future Extensions (Post-MVP)

- Cloud sync integrations (Google Drive, OneDrive).  
- AI-curated highlight videos and yearly summaries.  
- macOS and mobile versions.  
- “Pro” tier with smart albums and deduplication analytics.  
- AI face clustering and smart search.  

---

## 🧩 10. Development Notes for AI Agent (important)

- Structure code into **modules**: `import`, `conversion`, `gallery`, `flashback`, `settings`.  
- Importer should accept ZIPs, folders, and file arrays.  
- Use asynchronous file handling for speed; batch convert in background.  
- Handle media discovery recursively — skip unsupported formats silently.  
- All operations should work fully offline.  
- Document setup, dependencies, and build commands in README.  
- Include robust error handling and logs (conversion failures, duplicates skipped).  
- Test with mixed datasets (ZIP + MP4 + JPEG + PNG).  

---

**End of PRD**
