# Mac Storage Cleanup Guide (Developer Edition)

When macOS "System Data" swells to hundreds of gigabytes, it is rarely a simple cache issue. For developers, this space is almost always consumed by hidden IDE build files, virtual machines, and simulator images.

Follow these steps to safely reclaim massive amounts of storage space.

---

## 1. Hunt Down the Bloat

Before deleting things blindly, find out exactly which folders are hoarding your storage.

Open **Terminal** and run this command to list the 15 largest folders in your hidden Library directory (this may take a minute to calculate):
```bash
du -sh ~/Library/* 2>/dev/null | sort -hr | head -n 15
```

If you suspect your own projects are causing the issue, run this to check your main user directory:
```bash
du -sh ~/* 2>/dev/null | sort -hr | head -n 15
```

---

## 2. Xcode Cleanup (~50GB - 150GB+)

Xcode is notorious for keeping every build file and simulator image forever. You can safely delete these via Terminal. Xcode will simply generate fresh (and much smaller) ones the next time you build a project.

**Clear Derived Data (Temporary build files):**
```bash
rm -rf ~/Library/Developer/Xcode/DerivedData/*
```

**Clear old iOS Device Support files:**
```bash
rm -rf ~/Library/Developer/Xcode/iOS\ DeviceSupport/*
```

**Clear old Xcode caches:**
```bash
rm -rf ~/Library/Caches/com.apple.dt.Xcode
```

**Delete unavailable/disconnected simulators:**
```bash
xcrun simctl delete unavailable
```
*(Note: Active simulators live in `~/Library/Developer/CoreSimulator` and will naturally take up 15GB–30GB. Leave this folder alone so your active emulators continue to work).*

---

## 3. Android Studio Cleanup (~30GB - 100GB+)

Android Studio eats up space by downloading a new 5GB-10GB "System Image" (virtual hard drive) for every different API level you test against.

### The Best Practice: Consolidate APIs
Instead of having emulators running API 30, 33, and 36, **recreate all your emulators to use a single API level (e.g., API 37)**. Android Studio shares the system image across all emulators using that API, saving you dozens of gigabytes.

### How to clear old System Images & NDK safely:
1. Open **Android Studio**.
2. Go to **Tools > SDK Manager**.
3. Under the **SDK Platforms** tab, check the **Show Package Details** box (bottom right).
4. Uncheck any System Images for API levels you no longer use.
5. Switch to the **SDK Tools** tab, check **Show Package Details**, and uncheck older versions of **NDK (Side by side)**.
6. Click **Apply** to delete them.

*Nuclear Terminal Option (Will break current emulators until re-downloaded):*
```bash
rm -rf ~/Library/Android/sdk/system-images/*
rm -rf ~/Library/Android/sdk/ndk/*
```

---

## 4. General macOS Cache Clear

It is good practice to occasionally clear your everyday application caches (browsers, Spotify, Discord, etc.).

1. Open **Finder**.
2. Press `Cmd + Shift + G` and enter exactly: `~/Library/Caches`
3. Press `Cmd + A` to select all contents.
4. Press `Cmd + Delete` to move them to the Trash.
5. **Empty the Trash and immediately Restart your Mac.** 
*(Restarting is critical so macOS can build fresh cache files cleanly).*

---

## 5. Check Time Machine Snapshots

If you use Time Machine but haven't plugged in your drive recently, macOS hides backups on your internal drive.

Check for local snapshots:
```bash
tmutil listlocalsnapshots /
```

If a list of dates appears, plug in your Time Machine drive and let it back up, OR force delete them with:
```bash
sudo tmutil thinlocalsnapshots / 10000000000 4
```