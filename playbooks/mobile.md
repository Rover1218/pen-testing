# Mobile / Thick-Client Pen-Test Playbook

For Android/iOS apps and desktop thick clients. Aligned with the **OWASP MASVS/MASTG**. Most of
this needs a device/emulator and the app binary; some steps (static analysis) Claude can help with.

## Android

### Static
- [ ] Pull the APK; decompile: `apktool d app.apk`, `jadx app.apk` for source.
- [ ] `AndroidManifest.xml` — exported components (activities/services/receivers/providers),
      `debuggable`, `allowBackup`, custom permissions, deep links / intent filters.
- [ ] Hardcoded secrets, API keys, endpoints (`strings`, `grep`, MobSF).
- [ ] Insecure storage — `SharedPreferences`, SQLite, external storage of sensitive data.
- [ ] Certificate pinning present? Crypto usage (weak algos, hardcoded keys).
- [ ] Automated: **MobSF** static scan.

### Dynamic
- [ ] Proxy traffic (Burp) — install CA, defeat pinning if needed (Frida/objection).
- [ ] Inspect API calls → hand the API to [`api.md`](api.md).
- [ ] Runtime tampering / hooking with **Frida** / **objection**: bypass root/jailbreak/pinning,
      dump keychain/prefs, hook auth checks.
- [ ] Local data at rest after use (logs, caches, databases).

## Flutter apps (special handling)

Flutter compiles Dart to **native code** (`lib/<abi>/libapp.so`) and ships its own TLS stack, so
two things differ from normal Android testing:

1. **Secrets/URLs hide in `libapp.so`, not in decompiled Java.** Extract printable strings from the
   native libs to find hardcoded API endpoints and keys.
2. **Flutter ignores the system proxy AND user-installed CA certs.** Burp won't see Flutter traffic
   by default. To intercept it you must either patch the app or hook it at runtime.

### Static (automated)
- [ ] Run the helper: `./scripts/mobile/flutter-scan.ps1 -Apk app.apk -Engagement <name>`
      It unzips the APK, runs apktool/jadx, extracts strings from `libapp.so`, and writes:
      `endpoints.txt` (API URLs to test), `secrets.txt` (potential hardcoded secrets), plus the
      decompiled tree. **Feed `endpoints.txt` straight into [`api.md`](api.md).**
- [ ] Review `AndroidManifest.xml` flags: `debuggable`, `allowBackup`, `usesCleartextTraffic`,
      exported components, deep links.
- [ ] Run MobSF for a full static report (`-RunMobsf` with a running MobSF instance, or `mobsfscan`).

### Intercepting Flutter traffic (the hard part)
Pick one approach to defeat Flutter's built-in TLS validation:
- [ ] **reFlutter** — patches the APK to route traffic through your proxy + trust your CA. Repackage,
      re-sign (`apksigner`), install, then proxy with Burp. Easiest for most apps.
- [ ] **Frida + objection** — on a rooted device/emulator, hook the native SSL functions to disable
      pinning/validation at runtime (`objection -g <pkg> explore`, then `android sslpinning disable`).
- [ ] **proxychains/iptables/ProxyDroid** — force traffic through the proxy at the OS level.
- [ ] Once intercepting, every API call you capture goes into the API methodology — that's where the
      real bugs are.

### Flutter-specific checks
- [ ] Hardcoded secrets/endpoints in `libapp.so` (from the scan above).
- [ ] Certificate pinning present? (most Flutter apps don't pin by default = interceptable).
- [ ] Sensitive data in local storage (shared_preferences, sqflite, hive, path_provider files).
- [ ] Business logic enforced server-side, not just in the Dart UI (it's trivially bypassed).

## iOS

### Static
- [ ] Decrypt/obtain IPA; inspect `Info.plist`, entitlements, `ATS` exceptions.
- [ ] `class-dump` / Hopper/Ghidra for symbols; hardcoded secrets & endpoints.
- [ ] Keychain usage, Data Protection class, insecure `NSUserDefaults`/plist storage.

### Dynamic
- [ ] Frida/objection on jailbroken device — pinning bypass, keychain dump, method hooking.
- [ ] Proxy & analyze API traffic.

## Desktop thick clients
- [ ] Identify framework (Electron, .NET, Java, native C/C++).
- [ ] **Electron:** check `nodeIntegration`, `contextIsolation`, `asar` unpack for source,
      exposed IPC, loaded remote content.
- [ ] **.NET/Java:** decompile (dnSpy/ILSpy, JD-GUI/CFR) — secrets, logic, crypto.
- [ ] Local files, registry, config for secrets & insecure permissions.
- [ ] Intercept network/IPC traffic; check update mechanism (signed? TLS?).
- [ ] DLL/dylib hijacking, insecure file/registry ACLs, running as admin.

## Cross-cutting checks
- [ ] Transport security (TLS, pinning, ATS/cleartext traffic).
- [ ] Authentication & session handling (token storage, biometric bypass).
- [ ] Logging of sensitive data; screenshots/backgrounding leaks; clipboard.

## Tooling
| Platform | Tools |
|---|---|
| Android | `apktool`, `jadx`, MobSF, Frida, objection, `adb` |
| iOS | `class-dump`, Hopper/Ghidra, Frida, objection, MobSF |
| Desktop | dnSpy/ILSpy, JD-GUI/CFR, `asar`, Ghidra, Process Monitor |
| Traffic | Burp Suite, mitmproxy |
