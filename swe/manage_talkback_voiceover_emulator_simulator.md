# Enable and Disable TalkBack / VoiceOver in Android Emulator and iOS Simulator

This guide covers quick ways to turn screen readers on and off while testing mobile apps.

> **Note:** Android's ADB method is generally reliable. iOS Simulator does not provide a documented `simctl` command for toggling VoiceOver. Treat the iOS commands below as experimental and verify them on your installed runtime.

## Android Emulator — TalkBack

### Option A: Android Settings

1. Open **Settings** in the emulator.
2. Go to **Accessibility → TalkBack**.
3. Turn **Use TalkBack** on or off.

The exact menu names can vary by Android version.

### Option B: Enable TalkBack with ADB

Run from your Mac terminal:

```bash
adb shell settings put secure enabled_accessibility_services \
  com.google.android.marvin.talkback/com.google.android.marvin.talkback.TalkBackService

adb shell settings put secure accessibility_enabled 1
```

If TalkBack is installed and its service component is available, this enables it.

### Disable TalkBack with ADB

```bash
adb shell settings put secure accessibility_enabled 0
adb shell settings delete secure enabled_accessibility_services
```

`settings delete` removes the service setting. This avoids passing an empty value to `settings put`.

### Check the setting

```bash
adb shell settings get secure accessibility_enabled
adb shell settings get secure enabled_accessibility_services
```

The first command should return `1` when accessibility is enabled and `0` when disabled. The second shows the configured accessibility service(s).

### TalkBack shortcut

On supported Android versions, configure **Settings → Accessibility → TalkBack → TalkBack shortcut**.

The common shortcut is holding **Volume Up + Volume Down** together for a few seconds. The emulator toolbar may expose only one volume control, so the shortcut can be awkward to trigger there. Use ADB or the Settings screen instead.

---

## iOS Simulator — VoiceOver

### Option A: Simulator UI (recommended)

1. Boot the iOS Simulator.
2. Open the simulated **Settings** app.
3. Go to **Accessibility → VoiceOver**.
4. Turn VoiceOver on or off.

Menu availability can vary by iOS runtime.

### Option B: Keyboard / accessibility shortcut

On a physical iPhone, VoiceOver can be assigned to the Accessibility Shortcut (typically triple-click the side or Home button). The iOS Simulator does not consistently emulate that hardware shortcut, so use Settings when possible.

### Option C: Experimental command-line workaround

There is **no documented, supported `xcrun simctl` command to enable or disable VoiceOver**.

Some runtime-specific experiments write a preference or attempt to launch the VoiceOver service. These are undocumented and may fail or change across Xcode/iOS versions. A preference change alone does not prove VoiceOver is active.

If experimenting, first identify the booted simulator:

```bash
xcrun simctl list devices booted
```

Avoid relying on undocumented `defaults`, `launchctl`, or service commands in team scripts. Prefer the Simulator Settings UI and verify actual VoiceOver behavior.

---

## Quick reference

| Platform | Enable | Disable |
|---|---|---|
| Android Emulator | ADB commands above, or Settings → Accessibility → TalkBack | ADB commands above, or turn TalkBack off in Settings |
| iOS Simulator | Settings → Accessibility → VoiceOver | Settings → Accessibility → VoiceOver |
| iOS `simctl` | No documented supported toggle | No documented supported toggle |

## Testing checklist

- Verify the screen reader actually speaks; a setting value alone is not enough.
- Navigate using screen-reader gestures, not only mouse clicks.
- Check labels, roles, reading order, focus, and dynamic announcements.
- Test important flows on a physical device before release.
