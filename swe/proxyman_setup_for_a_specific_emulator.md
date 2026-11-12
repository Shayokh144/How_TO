# How to Install Proxyman Certificate on a Specific Android Emulator

When running multiple Android emulators (e.g., one for automated testing, one for debugging), Proxyman's default "Override All Emulators" button will modify all of them. 

Follow this guide to isolate Proxyman's proxy settings and system certificate to a single target emulator.

## Prerequisites
* Proxyman is running on your Mac (default port `9090`).
* Your target Android emulator must be running a **Google APIs** image (Google Play images cannot run `adb root`).

---

## Step 1: Identify Your Target Emulator

First, list all running emulators to find the ID of your target device:

```bash
adb devices
```

To verify which ID corresponds to which virtual device name, run:
```bash
for d in $(adb devices | grep emulator | cut -f1); do echo "$d -> $(adb -s $d emu avd name | head -n 1)"; done
```

*For the rest of this guide, replace `emulator-5556` with your target emulator's ID.*

---

## Step 2: Route HTTP Traffic

Point the global HTTP proxy of your specific emulator to Proxyman. `10.0.2.2` is a special alias inside the Android emulator that routes directly to your Mac's `localhost`.

```bash
adb -s emulator-5556 shell settings put global http_proxy 10.0.2.2:9090
```

---

## Step 3: Install the System CA Certificate

Android 7+ requires certificates to be installed at the system level for apps to trust them. Create a bash script (e.g., `install_cert.sh`) with the following code to mount Proxyman's root certificate into your specific emulator.

```bash
#!/bin/bash

# Define your target emulator
TARGET_EMU="emulator-5556"

echo "Targeting $TARGET_EMU..."

# 1. Restart adb as root on the target emulator
adb -s $TARGET_EMU root
sleep 2

# 2. Locate Proxyman's root certificate
CERT="$HOME/Library/Application Support/com.proxyman.NSProxy/app-data/proxyman-ca.pem"
if [ ! -f "$CERT" ]; then
    echo "Error: Proxyman cert not found at $CERT"
    exit 1
fi

# 3. Hash the certificate for Android
HASH=$(openssl x509 -inform PEM -subject_hash_old -in "$CERT" | head -n 1)
cp "$CERT" "/tmp/$HASH.0"

# 4. Push the certificate to the emulator
adb -s $TARGET_EMU push "/tmp/$HASH.0" "/data/local/tmp/$HASH.0"

# 5. Mount the certificate into the system CA store
adb -s $TARGET_EMU shell "
  mkdir -p /data/local/tmp/cacerts
  cp /apex/com.android.conscrypt/cacerts/* /data/local/tmp/cacerts/ 2>/dev/null || cp /system/etc/security/cacerts/* /data/local/tmp/cacerts/
  cp /data/local/tmp/$HASH.0 /data/local/tmp/cacerts/
  chmod 644 /data/local/tmp/cacerts/$HASH.0
  chcon u:object_r:system_file:s0 /data/local/tmp/cacerts/$HASH.0
  mount --bind /data/local/tmp/cacerts /apex/com.android.conscrypt/cacerts 2>/dev/null || mount --bind /data/local/tmp/cacerts /system/etc/security/cacerts
"

echo "Successfully mounted Proxyman certificate to $TARGET_EMU system store."
```

Run the script from your terminal:
```bash
bash install_cert.sh
```

---

## Step 4: Important Note for Flutter Developers

By default, Dart's `HttpClient` ignores the Android system `global http_proxy`. Even with the system certificate installed, Flutter API calls will bypass Proxyman unless you do one of the following:

**Option A (Code - Debug Only):** Add an `HttpOverrides` block to your `main.dart` to force traffic to the proxy.
```dart
class DevProxyHttpOverrides extends HttpOverrides {
  @override
  HttpClient createHttpClient(SecurityContext? context) {
    return super.createHttpClient(context)
      ..findProxy = (uri) => 'PROXY 10.0.2.2:9090';
      // badCertificateCallback is no longer needed because the System CA is installed!
  }
}
```

**Option B (Proxyman VPN APK):** Install Proxyman's VPN app onto your target emulator to capture traffic at the network level.
```bash
adb -s emulator-5556 install /Applications/Proxyman.app/Contents/Frameworks/ProxymanCore.framework/Resources/*.apk
```
Open the newly installed app on the emulator and hit "Start".

---

## Step 5: Reverting Changes

When you are done debugging, remove the proxy settings so the emulator can access the internet normally. Rebooting the emulator will automatically unmount the temporary system certificate.

```bash
adb -s emulator-5556 shell settings put global http_proxy :0
adb -s emulator-5556 reboot
```