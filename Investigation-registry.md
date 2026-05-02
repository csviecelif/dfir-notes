# 🧠 Windows Registry Forensic Triage Guide

---

## 🧭 Overview

The Windows Registry is a hierarchical database that stores operating system, user, hardware, and application configuration data. From an investigative perspective, it is one of the most valuable artifact sources because it preserves evidence of persistence, execution, security configuration, user activity, device usage, and system manipulation.

This guide is focused on **what to inspect during an investigation**. It is not a Registry internals reference or malware development note. The goal is to highlight Registry locations that frequently provide high-value evidence during triage, incident response, and forensic review.

---

## 🗂️ Registry root keys

These root keys are the logical views most commonly referenced during analysis:

- `HKCR` — File associations and COM/class registration data.
- `HKCU` — Settings for the currently logged-on user.
- `HKLM` — System-wide configuration that applies to the local machine.
- `HKU` — All loaded user profiles, typically organized by SID.
- `HKCC` — Current hardware profile information.

These are logical paths presented by Windows. In forensic work, the underlying hive files are often more important than the logical aliases.

---

## 📁 Hive files

Registry hive files are structured on-disk containers that back the logical Registry paths.

### Core system hives

These are commonly stored under `%SystemRoot%\System32\Config`:

- `SYSTEM` — Services, drivers, control sets, hardware configuration, mounted devices, and boot-related settings.
- `SOFTWARE` — Installed software, system policies, persistence locations, application configuration, and shell-related settings.
- `SAM` — Local account database information.
- `SECURITY` — Local security policy data and secret material.
- `DEFAULT` — Default profile settings used before user logon and as a template context in some scenarios.

### User hives

These are typically stored inside each user profile:

- `%UserProfile%\NTUSER.DAT` — User-specific settings such as Explorer history, Run keys, and many per-user execution artifacts.
- `%LocalAppData%\Microsoft\Windows\UsrClass.dat` — Per-user class registration, shell integration, and COM-related artifacts.

When investigating user activity or per-user persistence, `NTUSER.DAT` and `UsrClass.dat` are often just as important as the system hives.

---

## 🔍 Why malware uses the Registry

The Registry is commonly abused because it offers stable storage, automatic execution points, policy control, and extensive integration with the operating system.

Common malicious use cases include:

- **Persistence** — Adding autorun entries, service definitions, shell modifications, or boot-time execution values.
- **Configuration storage** — Saving C2 addresses, campaign IDs, mutex names, execution flags, or cryptographic material.
- **Defense evasion** — Disabling or weakening Defender, firewall rules, UAC, logging, or credential protections.
- **Payload staging** — Storing encoded, compressed, or binary data that can later be decoded in memory.
- **Credential theft support** — Modifying settings that expose or preserve credentials.
- **Rootkit or stealth support** — Adjusting service, driver, or boot-related values to maintain stealth or early execution.
- **System manipulation** — Changing policy or shell behavior to alter normal user workflows or weaken security controls.
- **Data theft or temporary storage** — Saving stolen values, exfiltration queues, or operator notes under obscure keys.

Because of this, Registry analysis is not only about persistence. It is also about identifying **intent**, **scope**, and **post-compromise behavior**.

---

## 📌 Investigation priorities

A practical Registry review during an investigation usually follows this order:

1. Check high-confidence persistence locations.
2. Check service and driver-related keys.
3. Check security control disablement and evasion.
4. Check execution artifacts and user activity traces.
5. Check scheduled task, COM, and shell-related hijacks.
6. Check device usage, credential-related material, and suspicious custom keys.

This order helps reduce time-to-answer during triage while still preserving breadth.

### 🧰 Eric Zimmerman’s Registry Explorer

When analyzing extracted hive files, use **Registry Explorer** for fast navigation, value comparison, and deleted artifact review. Pay special attention to **Associated deleted records** and **Unassociated deleted values**, since attackers often remove persistence or configuration keys after use.

---

## 🚀 Persistence

### Run and RunOnce keys

Review these common autostart locations for suspicious command lines, renamed binaries, script launchers, LOLBins, or executables placed in user-writable directories.

- `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`  
  Most frequently abused by droppers, loaders, and RATs because it is per-user, simple, and reliable.
- `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce`
- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce`
- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run`
- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`  
  Higher-impact because it affects the machine globally and often requires elevated rights.

### What to look for

- Executables in `%TEMP%`, `%APPDATA%`, `%LOCALAPPDATA%`, `C:\ProgramData`, or unusual subfolders.
- `powershell.exe`, `cmd.exe`, `wscript.exe`, `cscript.exe`, `mshta.exe`, `rundll32.exe`, or `regsvr32.exe` launching external content.
- Base64 blobs, encoded parameters, long command lines, or unusual quoting.
- File names designed to resemble legitimate software.
- Empty values, orphaned paths, or entries pointing to deleted files.

---

## 🛠️ Services

Malicious services remain one of the most reliable persistence mechanisms because they can execute at boot, run as privileged accounts, and blend into normal system behavior.

Review:

- `HKLM\SYSTEM\CurrentControlSet\Services\<name>`
- `HKLM\SYSTEM\CurrentControlSet\Services\<name>\Parameters`

### What to inspect

- `ImagePath` — The actual executable, DLL, or driver path.
- `Start` — Startup mode. `0x02` means automatic start and deserves close review.
- `Type` — Helps distinguish service types such as user-mode service versus driver.
- `ServiceDll` under `Parameters` — Critical when the service runs through `svchost.exe`.
- `DisplayName`, `Description`, and `ObjectName` — Useful for detecting impersonation and odd execution context.

### Red flags

- Paths under `%TEMP%`, `%APPDATA%`, profile folders, or archive extraction locations.
- Services using vague names that imitate Microsoft components.
- Broken paths, replaced binaries, or services with no legitimate vendor relationship.
- Unexpected autostart services installed close to the intrusion timeframe.

---

## 🛡️ Defense evasion

### Windows Defender and protection settings

Check whether security controls were weakened through policy or configuration changes.

- `HKLM\SOFTWARE\Policies\Microsoft\Windows Defender`
- `HKLM\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection`

### Important values

- `DisableAntiSpyware` — Historically significant and suspicious if set to disable protection, though interpretation depends on OS version and policy model.
- `DisableRealtimeMonitoring` — Indicates real-time monitoring was disabled or modified.

### UAC and elevation behavior

- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System`

Review:

- `EnableLUA`
- `ConsentPromptBehaviorAdmin`

Changes here may indicate attempts to reduce prompts, weaken elevation boundaries, or normalize malicious administrative actions.

### Firewall configuration

- `HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy`

Look for broad changes that reduce filtering or alter expected enforcement behavior.

### WDigest

- `HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest`

Review:

- `UseLogonCredential`

If set to `1`, this is a major finding because it can allow plaintext credential material to be retained in memory under compatible configurations. Even on newer systems, the presence of this value may still indicate attacker intent, legacy hardening drift, or prior credential access activity.

---

## 🎯 IFEO abuse

Image File Execution Options can be used legitimately for debugging, but they are also widely abused to hijack execution flow through the `Debugger` value.

Review:

- `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\<target.exe>`

### Important value

- `Debugger`

A malicious `Debugger` value can cause Windows to launch a different binary when the target executable starts.

### Common targets

- `taskmgr.exe`
- `procmon.exe`
- `procexp.exe`
- `sethc.exe`
- `utilman.exe`
- Security or administrative tools that an attacker wants to suppress, replace, or monitor

This technique should also be reviewed alongside related execution redirection scenarios such as `SilentProcessExit`-based monitoring or trigger behavior when relevant to the case.

---

## 🪟 Winlogon and AppInit abuse

### Winlogon

Review:

- `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`
- `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Notify`

### Important values

- `Userinit`
- `Shell`

These values are extremely sensitive. Extra binaries, appended command lines, or non-standard shells are strong indicators of persistence or interactive session hijacking.

### AppInit DLLs

Review:

- `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows`
- `HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Windows`

### Important values

- `AppInit_DLLs`
- `LoadAppInit_DLLs`

AppInit DLLs can force DLL loading into processes that load `user32.dll`, making them a classic persistence and injection mechanism.

### What to look for

- Unsigned or unknown DLLs.
- DLLs in user-writable directories.
- DLLs with random names or masquerading names.
- 32-bit and 64-bit mismatches used to selectively target processes.

---

## 🧩 COM hijacking

Per-user COM registration is an important review area because `HKCU`-based class entries can override or redirect expected component loading behavior.

Review:

- `HKCU\Software\Classes\CLSID\{<CLSID>}\InprocServer32`
- Compare against `HKLM\SOFTWARE\Classes\CLSID\{<CLSID>}\InprocServer32`

### What to look for

- Unexpected per-user overrides that point to a user-profile DLL.
- COM server paths under suspicious directories.
- CLSIDs associated with Explorer, Task Scheduler, MMC, Control Panel items, or commonly abused shell components.
- Empty default values or paths to missing DLLs that indicate incomplete cleanup.

A per-user COM override is often stealthier than a machine-wide persistence point because it blends into user-specific configuration and may only trigger under certain execution paths.

---

## 🏁 Boot execution and early-start persistence

Review these locations for signs of execution before normal user activity begins.

- `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\BootExecute`
- `HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs`
- `HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot\Minimal\<ServiceName>`

### What to look for

- Additional commands in `BootExecute` beyond the expected baseline.
- Unexpected KnownDLLs changes that may support DLL replacement or load-path abuse.
- Service entries added to Safe Mode-related branches.

`BootExecute` is especially critical because it can affect startup flow before the system reaches a normal interactive state.

---

## 🔐 Credential-related artifacts

Review credential-bearing or credential-supporting areas carefully.

- `HKLM\SAM\SAM\Domains\Account\Users`
- `HKLM\SECURITY\Policy\Secrets`
- `HKLM\SECURITY\Cache`

### Investigative value

- Local account structures can reveal account presence and credential-related material.
- LSA secret storage may contain service account secrets and other sensitive values.
- Cached domain logon data can indicate prior domain access and credential exposure scope.

These paths are sensitive and high-value. Treat any unauthorized access or tampering as a major escalation indicator.

---

## 🧬 C2 and RAT configuration

Attackers often store lightweight configuration in obscure Registry locations because the data survives reboot and does not require a separate config file.

Review:

- `HKCU\Software\<random or spoofed key>`
- `HKLM\SOFTWARE\<spoofed vendor name>`
- `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks\{<GUID>}`
- `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Custom`

### What to look for

- Base64, hexadecimal, binary blobs, compressed content, or unusually long strings.
- Misspelled vendor names or keys imitating legitimate software.
- Encoded URLs, IPs, ports, mutex names, campaign identifiers, or execution flags.
- Task action content that points to scripts, LOLBins, or user-profile payloads.

Do not ignore custom subkeys simply because they appear application-specific. Attackers frequently hide in “boring” software-like naming.

---

## 🗂️ Activity artifacts

Registry activity artifacts help reconstruct what the user or malware interacted with, even when the payload is no longer present.

Review:

- `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist`
- `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`
- `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePidlMRU`
- `HKCU\Software\Microsoft\Windows\CurrentVersion\Search\RecentApps`
- `HKCU\Software\Microsoft\Windows\Shell\Bags`
- `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache`

### What each area can reveal

- `UserAssist` — GUI-based program execution history; entries are ROT13-encoded and useful for identifying launched applications.
- `RecentDocs` — Recently opened documents and file interaction traces.
- `OpenSavePidlMRU` — File dialog interaction artifacts, including recently accessed or downloaded items.
- `RecentApps` — Application usage traces on supported systems.
- `Shell\Bags` — Folder access and Explorer view history, useful for interaction reconstruction.
- `TaskCache` — Scheduled task definitions and metadata that may reveal persistence, execution timing, or deleted task remnants.

These artifacts are often strongest when correlated with surrounding timestamps and other host evidence.

---

## 📎 Physical interaction and removable media

Device history can show physical access, staging activity, or data movement.

Review:

- `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR`
- `HKLM\SYSTEM\CurrentControlSet\Enum\USB`
- `HKLM\SYSTEM\MountedDevices`
- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer`

### What to look for

- USB storage device identifiers and serial numbers.
- Evidence of previously connected removable media.
- Drive-letter assignments and volume-to-device mapping.
- Policy changes intended to restrict or alter expected removable media behavior.

`USBSTOR` is especially useful because it often provides enough detail to tie device presence to a specific piece of removable hardware.

---

## ⚠️ Practical red flags

Across all Registry areas, prioritize these findings:

- Autoruns or services pointing to user-writable folders.
- PowerShell, script hosts, LOLBins, or chained command lines.
- Empty masquerading keys with one suspicious value.
- Encoded data where plain text would normally be expected.
- Security settings weakened shortly before or during suspicious activity.
- Per-user overrides for COM or shell behavior.
- Machine-wide persistence added without an expected software installation context.
- Paths to deleted files, network shares, or transient directories.
- Registry changes that align with login, beaconing, staging, or lateral movement timelines.

---

## 🧠 Interpretation notes

A single suspicious Registry value is not always enough to prove compromise. Some keys are modified by administrators, installers, endpoint software, accessibility features, or legacy troubleshooting workflows.

The key question is whether the Registry content is **consistent with the host’s expected role, user behavior, software inventory, and incident timeline**. The strongest findings are the ones that combine suspicious pathing, suspicious execution semantics, and suspicious timing.

---

## 📍 Scope reminder

This guide is intentionally focused on **what to inspect** during a Windows Registry investigation. It is designed for triage and investigative review, with emphasis on persistence, evasion, activity reconstruction, credential exposure, and system manipulation.
