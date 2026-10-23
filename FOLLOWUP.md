# Follow-up Plan

Items identified during review of the notification badge feature (2026-10-11).
Work through these in order; check off as completed.

## 1. Performance: Cache notification detection results ✅ DONE

**Problem:** `appHasNotification(_:)` in `WindowEnumerator.swift` iterates
`NSUserNotificationCenter.default.deliveredNotifications` for every window
during enumeration. This is O(windows × notifications) on the hot path.

**Fix applied (2026-10-11):**
- [x] Build `Set<String>` of bundle IDs with delivered notifications once per
      `enumerate(option:)` call via `deliveredNotificationBundleIDs()`
- [x] Per-PID cache (`notificationCache: [pid_t: Bool]`) so each unique app
      is checked only once
- [x] `appHasNotification` now takes the pre-built set and does O(1) lookup

**Result:** O(windows × notifications) → O(apps × notifications) once + O(1) per window.

**Files:** `Sources/Switch/WindowEnumerator.swift`

---

## 2. Test infrastructure — REMOVED

Tests cannot run under Termux (Android). The codebase is macOS-only
(AppKit, SwiftUI, XCTest). Testing is done manually on macOS.

**Manual test checklist for notification badge:**
- [ ] Build & run from Xcode (see build instructions below)
- [ ] Grant Accessibility permission (System Settings → Privacy & Security)
- [ ] Trigger a notification in an app (Mail, Messages, Slack, etc.)
- [ ] Verify dock icon shows a red badge number
- [ ] Press ⌘-Tab → verify red bell badge on the app's tile (top-right, grid mode)
- [ ] Enable Settings → Behavior → Vertical list → verify badge in trailing area
- [ ] Clear notifications → verify badge disappears on next Switch open
- [ ] Test with multiple apps (some with, some without notifications)

**Build from source:**
```
# 1. Install Xcode from App Store + XcodeGen
brew install xcodegen

# 2. Generate project
xcodegen generate

# 3. Open in Xcode, fix signing (set Team to your Apple ID or "None")
open Switch.xcodeproj

# 4. Build & Run (⌘R)
```

---

## 3. Deprecated API: `NSUserNotificationCenter`

**Problem:** `NSUserNotificationCenter` is deprecated since macOS 11.
The project targets macOS 14+, so `UNUserNotificationCenter` is available.

**Options:**
- [ ] Migrate to `UNUserNotificationCenter` (requires notification permission check)
- [ ] Drop the fallback entirely (rely on dock badge only) — simplest

**Files:** `Sources/Switch/WindowEnumerator.swift`

---

## 4. Feature documentation

**Problem:** The notification badge feature is not documented in `README.md`.

**Step:**
- [ ] Add a short section to `README.md` matching the existing concise style
  (e.g., "## Notification badges — Apps with pending notifications show a red bell badge on their tile.")

**Files:** `README.md`
