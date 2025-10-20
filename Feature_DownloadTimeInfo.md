# 📡 Feature Specification: Download Time Information (Static Version)

**Product:** SnapBacked  
**Feature Name:** Download Time Information  
**Version:** 1.0  
**Owner:** Sander Lund Ejdesgaard  
**Last Updated:** October 2025  

---

## 🎯 Goal
Provide users with a simple, clear message **before starting their Snapchat data download**, showing how long large downloads might take depending on internet speed.  
This helps users prepare properly — ensuring a **stable connection**, **sufficient time**, and a **powered device** — reducing failed or incomplete downloads.

---

## 💡 Feature Summary
Before initiating the **Turbo Download** or “Download All Memories” process, SnapBacked displays a **static informational message** explaining estimated download durations for various internet speeds.  

This lightweight addition sets user expectations and increases trust, while adding zero technical complexity.

**Example Message:**
> 💡 *The time to download your Snapchat data depends on your internet speed.*  
> For example:  
> - At **1 Mbps**, it could take around **115 hours** (nearly 5 days).  
> - At **10 Mbps**, about **11.5 hours**.  
> - At **50 Mbps**, roughly **2.5 hours**.  
> - At **100 Mbps**, about **1 hour and 15 minutes**.   
>
> ⚡ *Tip: For best results, connect to a reliable Wi-Fi or Ethernet network and keep your computer plugged in.*

---

## 🧩 Functional Behavior

### Trigger Point
- Display automatically **before** the user begins any download operation (Turbo Download or manual import).
- Appears as a pre-download **info card or modal popup** on the “Download Setup” screen.

### Behavior
- Shows a **static, pre-written message** (no input required).
- Contains estimated times for multiple internet speeds (1 Mbps → 1 Gbps).
- Includes one or more **practical tips** (stable connection, plugged-in power, patience).
- Provides a **“Proceed to Download”** button below the message.

---

## 🧱 Example UI Layout


- The card appears centered with a soft background highlight (purple or yellow accent).
- Text uses friendly tone and concise formatting.
- Clicking “Proceed with Download” continues to the normal download workflow.

---

## ⚙️ Implementation Notes

| Item | Description |
|------|--------------|
| **Complexity** | 🟢 Very Low |
| **Effort** | < 1 day |
| **Dependencies** | None |
| **Placement** | Download Setup screen (before initiating download) |
| **Behavior** | Static text display, dismissible by proceeding |

**Implementation Steps (Electron / Tauri):**
1. Add new React component: `DownloadTimeInfoCard.tsx`
2. Render component on download setup screen (before Turbo Downloader starts).
3. Include predefined text block (as shown above).
4. Optionally add “Learn More” tooltip linking to FAQ.
5. “Proceed” button triggers existing download handler.

---

## 🧠 Why This Feature Matters

| Benefit | Description |
|----------|--------------|
| **Transparency** | Builds trust by explaining how long downloads might take. |
| **Preparation** | Reduces incomplete downloads by reminding users to ensure Wi-Fi and power stability. |
| **Professionalism** | Makes SnapBacked feel polished, thoughtful, and user-friendly. |
| **Low Effort, High Impact** | Adds value without new backend logic or user inputs. |

---

## ⚠️ Design Considerations

| Aspect | Recommendation |
|--------|----------------|
| **Accuracy** | Use “approximately” wording; speeds and times vary by connection and server. |
| **Tone** | Friendly, helpful, non-technical. |
| **Visual Style** | Card or popup with soft warning colors (amber, violet). |
| **Placement** | Shown before or above “Start Download” button. |
| **Persistence** | Optional: allow “Don’t show this again” checkbox in later versions. |

---

## ✅ Acceptance Criteria

- [x] Static informational message is shown before any download begins.  
- [x] Message includes at least 4 speed tiers (1 Mbps → 100 Mbps).  
- [x] Message includes at least one practical preparation tip.  
- [x] “Proceed with Download” button continues to next step.  
- [x] Works fully offline (no internet dependency).  
- [x] Layout fits desktop window responsively.

---

## 🔮 Future Enhancements (Post-MVP)
- Dynamically estimate file size and calculate per-user time.
- Allow user to input or auto-detect their internet speed.
- Display real-time **progress-based ETA** during downloads.
- Add speed test integration for more accurate pre-download planning.
- Offer “Best Time to Download” suggestions based on network conditions.

---

## 📘 Summary
This **Download Time Information** feature:
- Sets clear expectations before a long process,  
- Helps users plan for optimal conditions,  
- Adds polish and professionalism to SnapBacked’s download flow,  
- And can be implemented quickly with **zero backend complexity**.

It’s a small but impactful feature that directly improves **user satisfaction and trust** — perfectly aligned with SnapBacked’s mission to make data ownership simple and stress-free.
