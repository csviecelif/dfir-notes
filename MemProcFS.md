# 🧠 MemProcFS DFIR Guide

---

## ⚡ Command to mount

```powershell
MemProcFS.exe -device ..\memdump.mem -forensic 1 -license-accept-elastic-license-2-0
```

## 🧭 Overview

MemProcFS is a memory forensics framework that mounts a memory image as a virtual filesystem. This makes investigation faster because artifacts can be explored directly in a read-only structure instead of being pulled only through command output.

This guide focuses on **what to inspect**, **where to look**, and **how to use the mounted view during an investigation**. It is meant for incident response, triage, and offline memory review.

---

## 🗂️ Why use it

MemProcFS is especially useful when you already have a full memory dump and want rapid access to processes, network data, modules, handles, registry material, and YARA hits in one place.[cite:45][cite:46]

It is a strong fit for cases where you want to browse the evidence like a filesystem, keep the analysis read-only, and correlate process artifacts with memory-backed traces from the same snapshot.[cite:46][cite:50][cite:57]

---

## 📁 Mounting model

A memory dump is mounted to a drive letter such as `M:\`, and the filesystem remains available while the MemProcFS process is running.[cite:50]

If forensic mode is enabled, additional output is generated under `M:\forensic\`, including progress and evidence-oriented output such as YARA hits and timeline-style data.[cite:49][cite:50]

A zero-filled progress file at the start is normal; it updates as the image is processed.[cite:49]

---

## 🔍 Core areas to inspect

### Processes

Start with the process tree and process folders. These usually provide command lines, parent-child relationships, memory maps, loaded modules, handles, and process-specific dumps.

Look for suspicious command lines, orphaned or unusual parentage, empty module lists, private executable memory, or processes that appear only in certain views.

### Network

Check active and historical network artifacts to identify remote control, beaconing, or unexpected listeners.

Prioritize established connections, odd destination IPs, uncommon ports, and long-lived sessions that correlate with suspicious processes.

### Modules and memory regions

Review loaded modules and memory maps for anomalous DLLs, unsigned code, injected regions, private executable pages, and mismatched file-versus-memory behavior.

### Handles and named objects

Open handles, named pipes, events, and mutexes can reveal coordination between processes, persistence, or operator tooling.

### Registry material

MemProcFS can expose Registry-related data from memory snapshots, which is useful for checking autostarts, services, configuration, and recent user activity without leaving the mounted view.

---

## 🧩 What to look for

### Process clues

- Unusual parent/child chains.
- Command lines with encoded content, scripts, or LOLBins.
- Processes with suspicious names in user-writable paths.
- Missing or inconsistent module visibility.
- Private executable memory regions.

### Network clues

- Repeated connections to the same external host.
- Beacon-like timing.
- Connections from unexpected system processes.
- Traffic tied to processes that should not normally communicate externally.

### Injection clues

- RWX or RX private regions.
- Module mismatch between what is loaded and what exists on disk.
- Hidden or unlinked artifacts.
- Suspicious thread start addresses.

### Configuration clues

- Embedded C2 data.
- Base64, hex, or compressed payload strings.
- Tooling references such as AnyDesk, TeamViewer, ScreenConnect, or PowerShell abuse patterns.
- Artefacts that suggest staged execution or operator persistence.

---

## 🛠️ YARA use

YARA is useful when you want to scan memory content for known tools, strings, payload markers, or malware families.

MemProcFS can run with built-in Elastic rules if the license is accepted, and it can also use external rule sets through forensic YARA options.[cite:49][cite:54]

For broader coverage, external YARA collections can be added, but the tradeoff is longer processing time and more noisy matches.[cite:54]

---

## 📊 Forensic output

When forensic mode is enabled, review the generated output rather than relying only on the mounted browser view.

Useful outputs include:

- progress indicators
- YARA hits
- timeline-style artifacts
- CSV summaries
- process-specific evidence folders

The `findevil.csv` and YARA-related output are especially useful for triage because they help separate signature hits from heuristic findings.[cite:49][cite:54]

---

## 🧰 Eric Zimmerman tooling

For Registry-oriented work that comes from memory or extracted hives, Eric Zimmerman’s Registry Explorer is a strong companion because it makes hive browsing, comparison, and deleted artifact review much faster.

That is especially useful when you need to inspect deleted values, compare registry paths, or review user and system persistence locations more comfortably than with raw text output.

---

## 🚩 High-value hunting targets

- Suspicious processes and command lines.
- Remote access tooling.
- Private executable regions.
- Named pipes and mutexes.
- Odd network destinations.
- Injected or hollowed processes.
- Registry persistence and service tampering.
- Evidence of credential access or lateral movement support.

---

## 🧠 Practical reading order

A fast investigation usually works best in this order:

1. Identify suspicious processes.
2. Check their command lines and parents.
3. Review modules, VADs, and handles.
4. Correlate with network activity.
5. Check YARA and forensic output.
6. Review registry and user activity artifacts.

This sequence reduces noise and gets you to the highest-value evidence first.

---

## 📍 Scope reminder

This guide is intentionally focused on **how to inspect a mounted memory image** using MemProcFS. It is designed for offline analysis, incident response, and evidence review on a full memory dump.
