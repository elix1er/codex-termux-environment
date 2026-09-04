# Fire TV blue-mic button — Mac/ADB investigation brief

Use this as the prompt for a Codex session on a Mac that has ADB access to this Fire TV Stick.

```text
You have authorized ADB access to my own Amazon Fire TV Stick. I want a focused, evidence-based reverse-engineering investigation of the blue Alexa microphone button on its Bluetooth remote.

Goal:
Determine whether the blue mic button can be remapped or intercepted so spoken text can reach a local Termux/Codex workflow. Do not overengineer a production solution. I want a practical POC verdict.

Hard boundaries:
- Do not root, unlock, exploit, downgrade, boot-modify, flash, bypass signatures, modify `/system`, or tamper with Amazon/Alexa packages.
- Do not disable or alter the existing Alexa button behavior.
- Do not send fuzzed broadcasts/intents, brute-force private APIs, or make changes that could break the stick/remote.
- Read-only inspection is preferred. A single clearly safe, non-persistent test is okay only if it directly validates a hypothesis.
- Stop if the only path requires root, a platform-signed app, Amazon’s private APIs, or modifying system software. Report that clearly.

Known device facts from Termux running on this same Fire TV:
- Fire OS 7 / Android 11, SDK 30
- Build fingerprint: `Amazon/karat/karat:11/RS8182.3811N/0032548774788:user/amz-p,release-keys`
- Production build: `ro.debuggable=0`
- Termux is unprivileged: `uid=u0_a248`, SELinux context `u:r:untrusted_app_27:s0:...`
- ADB is configured for TCP port 5555 and `adbd` reports running, but localhost ADB from Termux did not connect.
- This is a Bluetooth remote, and the system contains BLE voice-search code. Treat the microphone audio path as potentially separate from ordinary Android key events.

Relevant system packages and APKs:
- `/system/system_ext/priv-app/BluetoothKeyMapLib/BluetoothKeyMapLib.apk` (`com.amazon.device.bluetoothkeymaplib`)
- `/system/system_ext/priv-app/com.amazon.fireinputdevices/com.amazon.fireinputdevices.apk` (`com.amazon.fireinputdevices`)
- `/system/system_ext/priv-app/alexadirectivebrokerservice/alexadirectivebrokerservice.apk` (`com.amazon.alexadirectivebrokerservice`)
- Alexa runtime: `com.amazon.alexamediaplayer.runtime.ftv`

Static evidence already found:
- BluetoothKeyMapLib contains `KeyMapService`, `KeyMapSyncManager`, `KeyMapSyncTask`, and `KeyMetricService`.
- It contains BLE voice strings including `BT_BLE_VOICE_SEARCH`, `BT_BLE_VOICESEARCH_IDLE`, and `bleVoiceSearchCount`, plus a remote-button mapping table with a distinct `Voice` entry.
- Its exported service listens for `com.amazon.device.bluetoothkeymap.action.KEYMAP_SYNC`.
- It requests privileged permissions including `android.permission.BLUETOOTH_PRIVILEGED`, `android.permission.AMAZON_REMOTE_DFU`, and `com.amazon.tv.devicecontrol.WRITE`.
- `alexadirectivebrokerservice` receives Alexa directives only behind `amazon.speech.permission.SEND_ALEXA_DIRECTIVE`.
- These facts suggest the blue button may start a privileged Bluetooth/Alexa voice session rather than dispatching a remappable ordinary key event.

Work plan:

1. Confirm ADB and capture only useful device metadata:
```sh
adb devices -l
adb shell getprop ro.build.fingerprint
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk
adb shell getprop ro.debuggable
adb shell getprop ro.adb.secure
adb shell id
adb shell getenforce
```

2. Inspect actual input behavior while I press and hold the blue mic button. Start each capture, tell me exactly when to press/release it, then stop after 10–15 seconds:
```sh
adb shell getevent -lt
adb logcat -c
adb logcat -v threadtime
adb shell dumpsys input
```
Compare a normal D-pad key press against the blue mic button. Establish:
- Does the blue button emit any Linux/Android key code?
- If yes, identify its exact event device, scan code, Android key code, and dispatch target.
- If no, identify logs/services showing BLE voice capture or Alexa session startup.
- Do not expose account tokens, identifiers, or unrelated private logs in the report.

3. Inspect the current input-policy and relevant package components:
```sh
adb shell dumpsys input
adb shell dumpsys window policy
adb shell dumpsys activity services | grep -i -E 'alexa|voice|speech|bluetooth|input'
adb shell pm path com.amazon.device.bluetoothkeymaplib
adb shell pm path com.amazon.alexamediaplayer.runtime.ftv
adb shell pm path com.amazon.alexadirectivebrokerservice
adb shell pm path com.amazon.tv.keypolicymanager
```

4. Pull only the relevant system APKs to the Mac and inspect them statically:
```sh
adb pull /system/system_ext/priv-app/BluetoothKeyMapLib/BluetoothKeyMapLib.apk .
adb pull /system/system_ext/priv-app/alexadirectivebrokerservice/alexadirectivebrokerservice.apk .
adb pull /system/system_ext/app/KeyPolicyManagerUI/KeyPolicyManagerUI.apk .
```
Use `aapt/aapt2`, JADX, apktool, or strings to answer:
- Where is the `Voice` mapping resolved?
- Is it server-synced/config-signed, or configurable locally?
- Does a public/exported component genuinely accept a third-party remap, and what permission guards it?
- Are there documented/public intents or settings to assign the key to another app?
- Does the microphone stream pass through an ordinary Android `AudioRecord`/intent path accessible to third-party apps?

5. Check non-invasive standard options:
- Fire TV settings related to Alexa, accessibility shortcuts, remotes, button customization, and developer options.
- Whether an accessibility service can observe this specific key. Do not install anything unless we decide together after the findings.
- Whether an Android app can become the assistant/voice handler on this Fire OS build through a supported role/setting.

Deliverable:
Give me a short evidence table and a decisive conclusion in one of these forms:

A. “Userland POC feasible” — exact supported mechanism, what app/service to build, and the smallest next action.

B. “Button event visible but not safely remappable” — exact key code/path and why it stops there.

C. “System-owned BLE/Alexa path; no userland takeover” — cite the logs/static evidence and recommend the smallest alternative:
   `blue mic → private Alexa Skill with SearchQuery slot → authenticated queue/webhook → Termux/Codex`
   or another concrete, low-complexity option.

Do not confuse “ADB can observe the device” with “ADB can capture Alexa’s transcription.” The key question is whether the audio/text is ever exposed to an ordinary app before it reaches Amazon’s privileged Alexa stack.
```
