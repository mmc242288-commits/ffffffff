# حياة (Hayat) v2.0 — Android source

This project provides:
- Installed-app target selection.
- Overlay control panel.
- User-authorized MediaProjection screenshot capture.
- Screenshot point editor (tap to add, drag to reposition).
- Saved normalized point coordinates.
- Draggable numbered overlay points.
- One master mode: normal or long press, applied to all points.
- Repeating multi-stroke Accessibility gesture dispatch in one GestureDescription.
- Target-package foreground gating based on Accessibility window events.

## Build
Push the contents of this folder to a GitHub repository on branch `main`, then open Actions and run **Hayat Android APK**. Download the `hayat-debug-apk` artifact.

## Setup on device
1. Install the debug APK.
2. Select the target app in TouchPilot.
3. Grant overlay permission.
4. Enable TouchPilot in Android Accessibility settings.
5. Grant screen-capture permission.
6. Open the target app, use the floating panel to capture a screenshot, tap/drag points in the editor, and save.
7. Start the master control.

## Changes in v1.2
- Tap/hold strokes now use an explicit same-coordinate line, improving compatibility with Android gesture dispatch.
- Foreground-target tracking ignores unrelated accessibility content events.
- Start is blocked when there are no points or the Accessibility service is not connected.
- A rejected gesture dispatch stops the run loop and restores point touchability.
- The master normal/long-press control updates its displayed label immediately.

## Important verification status
This source has not been compiled in this environment because Android SDK/Gradle are unavailable here. The GitHub workflow is provided to perform a real build.
The OS Accessibility API accepts multiple gesture strokes in a single GestureDescription with the same start time, but actual multi-touch delivery is not guaranteed across all devices or target apps. Validate on the intended device/app.
Some apps prohibit capture or automated input. Respect their rules. This prototype does not bypass app protections.

## Changes in v1.6
- Overlay windows now opt into display-cutout areas, so chips sit exactly where gestures are sent on notched screens.
- Screenshot, saved points and gesture coordinates all use the real display size.
- Hidden overlays (during capture/editor) are also untouchable, so they no longer swallow taps in the point editor.
- Opening the keyboard no longer pauses the run loop (IME windows are ignored in foreground tracking).
- Old screenshots are deleted on each new capture instead of piling up.

## Changes in v1.7 (manual T button)
- No more automatic repeating taps. Nothing taps by itself.
- A floating round **T** button: drag it to any spot on screen.
- Tap **T** and it taps at its own position.
- From the floating panel, **إضافة نسخ من T** adds copies (T2, T3, ...). Drag each copy where you want.
- When you tap **T**, all copies tap at the same moment at their own positions. Tapping a copy directly does nothing.
- **حذف كل النسخ** removes all copies. Normal/long-press mode still applies to all.

## Changes in v1.8
- Three press types (cycle from the panel): single tap, long press, very fast repeat (no extra gap between rounds; tap T to start, tap the red circle to stop).
- "التزامن" switch: all points in one multi-touch gesture (same instant), or very quick one-after-another taps (works in apps that ignore multi-touch).
- Overlay windows use alpha 0.75 so Android 12+ does not block touches that pass through them.
- New copies get their own free slot, so copies added later never land on top of older ones.

## Changes in v1.9 (several T buttons)
- You can have up to 5 independent T buttons (T, T2, T3, ...), each with its own colour and its own copies.
- Pressing a T taps only that T and its own copies; other groups stay still.
- In the panel menu, pick a T in the top row (selected one has a thick ring), then add copies, capture a screenshot for it, or delete its copies. The "+" circle adds a new T.
- Copies are numbered by group (1.1, 1.2, 2.1, ...) and fan out next to their own T.
- Older saved data (one T plus points) is converted automatically into the first group.

## Changes in v2.0 (حياة)
- App renamed to **حياة** with a new round logo: green-to-teal disc, the word "حياة" and a small leaf. The same logo is the launcher icon (adaptive + themed/monochrome), the floating panel logo, and the main-screen header.
- Clear naming: main buttons are **T1..T5** (solid circle); copies are dashed "ghost" circles showing the owner on top and their number below (`T2 / 3`), so copies can never be mistaken for main buttons. The panel uses the same T1..T5 chips and colours.
- New **إخفاء الأزرار / إظهار الأزرار** tile: hides the look of every T and copy (they stay in place and let touches through) while the floating panel stays. Small T chips appear under the logo to press each group.
- Floating panel redesigned as a dark card with sections (الأزرار، النسخ، الضغط، الشاشة والأدوات), uniform tiles, scrolls on short screens and is kept on-screen.
- Main screen redesigned: app picker, live status for each permission (overlay, accessibility, capture), one main action button, short usage guide.
- Point editor restyled, with cancel and undo.
- Package name and class names are unchanged (`com.touchpilot`), so the build workflow keeps working.

## Changes in v2.1 (smoothness and accuracy)
- **Fast repeat no longer freezes the screen.** Several rounds of taps are packed into one gesture (max about 0.4 s long) instead of one system call per tap, so the device is called a few times per second rather than dozens. Fast repeat is now 30 ms touch + 20 ms pause (about 20 taps per second per point). Tune it with `FAST_PRESS_MS` / `FAST_GAP_MS` at the bottom of `TouchService.kt`.
- **Stopping is instant and reliable.** The red stop circle (and any T) now stops the repeat the moment the finger touches it, not when it is released. A real touch cancels an injected gesture in Android; the old code immediately fired the next gesture and fought the finger. Now a cancelled gesture pauses the loop for 350 ms so the touch goes through cleanly.
- **Presses no longer get lost.** Only the pressed group's buttons switch to pass-through (fewer window updates), and the first gesture waits 120 ms so the system has applied the change before the tap is sent. Taps no longer land on the button itself.
- **"Together off" is now truly quick.** Points are sent as one gesture with strokes a few ms apart instead of one system call per point.
- **No crash with many points or off-screen buttons.** Strokes per gesture are capped to the system limit and coordinates are never negative.
- Buttons and copies are kept fully on screen while dragging, so the position that is drawn is exactly the position that is tapped.
- `versionCode 8`, `versionName 2.1`.

Not compiled in this environment (no Android SDK). Build through the GitHub workflow and test on the device.

## Also in v2.1 (works only inside the selected app)
- The T buttons, copies and the floating logo are shown, touchable and active **only while the app selected on the main screen is in front**. When you leave it (Home, another app, Hayat's own main screen) they disappear, a running fast repeat stops, and no gesture is sent anywhere else on the phone.
- When the selected app comes back to the front, everything reappears in the same places.
- Notification shade / volume panel (System UI) and the keyboard do not count as leaving the app.
- To hide the look of the buttons while keeping them in place and working: panel menu → **إخفاء الأزرار**; use the small T chips under the logo to press.

## Hide mode (v2.1)
- **إخفاء الأزرار** hides everything except the floating Hayat logo. No extra chips are shown.
- While hidden, each T stays exactly where it was and still works: **press its (invisible) place** and it runs, as if it were visible. It cannot be dragged while hidden, and copies let touches go straight through to the game.
- The logo only opens the menu (it no longer runs anything). A finger on the logo, or on a T, stops a running fast repeat.
- **إظهار الأزرار** shows every T and copy exactly where it was.

## Hold to repeat (fast mode, v2.1)
- With press type **متكرر سريع**: put a finger on a T and **keep it there** = the taps keep repeating; **lift the finger** = it stops. Works with the buttons hidden too.
- The finger covers the T's own spot, so while holding, the taps go to that T's **copies** (a T without copies shows a hint). Normal tap and long-press modes are unchanged.
- The menu tile "تنفيذ الآن" still starts a toggle-style repeat that includes the T's own spot; touch the logo to stop it.

## Press-and-hold (press type "ممسوك", v2.1)
- Cycle **نوع الضغط** to **ممسوك**. Example: a red T at the top with two copies, one on the fire button and one on the prone button. **Put a finger on T and keep it there** = both copies are pressed and stay pressed (prone + fire); **lift the finger** = both are released.
- A **quick tap** on T (under 0.25 s) latches the hold instead; touch T again, or the red logo, to release.
- The finger covers T's own spot, so a hold presses the **copies** (a T without copies shows a hint).
- Held touches are chained 200 ms segments (`HOLD_SEGMENT_MS`) so the fingers never lift between segments. Leaving the selected app releases everything.
- Android cancels an injected gesture when a real finger touches the screen. If that happens during a hold, the copies are pressed again after 40 ms, so a hold may flicker while you use other fingers. Test on the device.
- **متكرر سريع** (hold = repeated taps on the copies) is unchanged.
