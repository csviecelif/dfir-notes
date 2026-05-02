# 🧠 Volatility 3 DFIR Notes

***

## ⚙️ Dependency installation

```bash
py -3.12 -m pip install -U volatility3
py -3.12 -m pip install pycryptodome
py -3.12 -m pip install yara-python
```

> If any generated `.txt` file is 0 KB, the plugin did not run successfully. Check the terminal output — a required dependency is likely missing.

***

## 🔧 Environment variables

```powershell
$Mem="<MEMORY_FILE>.mem"; $Out="<OUTPUT_FOLDER>"; mkdir $Out -Force | Out-Null
```

***

## 🚀 Main extraction script (single run)

```powershell
$plugins=@("windows.info.Info","windows.shimcachemem.ShimcacheMem","windows.netscan.NetScan","windows.netstat.NetStat","windows.pstree.PsTree","windows.cmdline.CmdLine","windows.envars.Envars","windows.privileges.Privs","windows.getsids.GetSIDs","windows.svcscan.SvcScan","windows.getservicesids.GetServiceSIDs","windows.malware.malfind.Malfind","windows.callbacks.Callbacks","windows.dlllist.DllList","windows.symlinkscan.SymlinkScan","windows.sessions.Sessions","windows.modules.Modules","windows.truecrypt.Passphrase","windows.modscan.ModScan","windows.registry.hivelist.HiveList","windows.hashdump.Hashdump","windows.lsadump.Lsadump","windows.registry.amcache.Amcache","windows.ssdt.SSDT","windows.driverirp.DriverIrp","windows.ldrmodules.LdrModules","windows.bigpools.BigPools","windows.desktops.Desktops","windows.skeleton_key_check.Skeleton_Key_Check","windows.joblinks.JobLinks","windows.verinfo.VerInfo","windows.filescan.FileScan","windows.mbrscan.MBRScan","windows.mutantscan.MutantScan","timeliner.Timeliner","windows.psscan.PsScan","windows.handles.Handles","windows.orphan_kernel_threads.Threads","windows.psxview.PsXView","windows.deskscan.DeskScan","windows.devicetree.DeviceTree","windows.malware.svcdiff.SvcDiff","windows.malware.pebmasquerade.PebMasquerade"); foreach($p in $plugins){$n=$p-replace'[\\.]','_'; py -3.12 vol.py -f $Mem $p | Out-File "$Out\\$n.txt" -Encoding UTF8}
```

***

## ⏳ Slow plugins (run after main collection)

```powershell
$plugins=@("windows.hollowprocesses.HollowProcesses","windows.vadinfo.VadInfo","windows.registry.printkey.PrintKey","windows.mftscan.MFTScan","windows.registry.certificates.Certificates","windows.registry.userassist.UserAssist"); foreach($p in $plugins){$n=$p-replace'[\\.]','_'; py -3.12 vol.py -f $Mem $p | Out-File "$Out\\$n.txt" -Encoding UTF8}
```

***

## 🔍 YARA memory scan (Example)

```powershell
py -3.12 vol.py -f $Mem windows.vadyarascan.VadYaraScan --yara-string "AnyDesk|TeamViewer|ScreenConnect|Cobalt Strike|Metasploit|Sliver|password|powershell -enc|cmd /c|certutil|VirtualAlloc|CreateRemoteThread|bitcoin|decrypt" | Out-File "$Out\\yara_hits.txt" -Encoding UTF8
```

***

## 🧩 YARA file-based scan

```powershell
py -3.12 vol.py -f $Mem windows.vadyarascan.VadYaraScan --yara-file "<RULES_PATH>.yar" | Out-File "$Out\\yara_file_hits.txt" -Encoding UTF8
```

***

## 🧲 Process dump (procdump)

> Use only in authorized and isolated environments.

```powershell
py -3.12 vol.py -f $Mem -o "$Out" windows.pslist --pid <PID> --dump
```

***

## 📌 Plugins documented outside batch execution

### Cachedump
`windows.cachedump.Cachedump` is useful for extracting cached credential material, but it is intentionally left out of the batch scripts because it is not part of the default triage collection set and may depend on specific registry-backed artifacts or target conditions.

### Strings
`windows.strings.Strings` is also intentionally left out of the batch scripts because, in Volatility 3, it is not a standalone memory scanner. It consumes output from an external `strings` command and maps recovered strings back to processes.

***

## 📊 Analysis workflow

1. Define the memory image path.
2. Run the main extraction script.
3. Run the slow plugins separately.
4. Perform YARA scanning.
5. Dump suspicious processes if needed.
6. Review the results with PEStudio, DIE, IDA, and VirusTotal if permitted.

***

## 📖 Plugin reference

- `windows.info.Info` — Collects basic system metadata from the memory image.
- `windows.shimcachemem.ShimcacheMem` — Extracts ShimCache data from memory to identify program execution artifacts.
- `windows.pstree.PsTree` — Displays parent-child process relationships to help analyze execution chains.
- `windows.psscan.PsScan` — Scans for EPROCESS objects to identify active, hidden, and terminated processes.
- `windows.psxview.PsXView` — Compares multiple process enumeration methods to detect hidden or inconsistent process visibility.
- `windows.cmdline.CmdLine` — Retrieves full process command-line arguments.
- `windows.envars.Envars` — Lists process environment variables.
- `windows.privileges.Privs` — Enumerates process token privileges.
- `windows.getsids.GetSIDs` — Lists security identifiers associated with processes.
- `windows.handles.Handles` — Enumerates open handles such as files, registry keys, processes, events, and mutexes.
- `windows.netscan.NetScan` — Scans memory for network objects such as TCP endpoints, UDP endpoints, and sockets.
- `windows.netstat.NetStat` — Lists active network connections using OS networking structures.
- `windows.hollowprocesses.HollowProcesses` — Detects possible process hollowing and image replacement activity.
- `windows.svcscan.SvcScan` — Enumerates Windows services from memory.
- `windows.getservicesids.GetServiceSIDs` — Lists service SIDs associated with services.
- `windows.dlllist.DllList` — Lists DLLs loaded in process address space.
- `windows.ldrmodules.LdrModules` — Identifies discrepancies between loader-linked modules and mapped memory regions.
- `windows.modules.Modules` — Lists loaded kernel modules.
- `windows.modscan.ModScan` — Scans memory for kernel module objects to detect hidden or unlinked drivers.
- `windows.filescan.FileScan` — Scans memory for file objects.
- `windows.mftscan.MFTScan` — Scans for NTFS MFT-related artifacts in memory.
- `windows.registry.hivelist.HiveList` — Lists registry hives present in memory.
- `windows.registry.printkey.PrintKey` — Prints registry keys and values from hive data.
- `windows.registry.amcache.Amcache` — Extracts Amcache execution and file metadata artifacts.
- `windows.registry.userassist.UserAssist` — Extracts UserAssist execution artifacts from user hives.
- `windows.registry.certificates.Certificates` — Extracts certificate-related registry data.
- `windows.hashdump.Hashdump` — Attempts to recover local account password hashes from memory-resident registry data.
- `windows.lsadump.Lsadump` — Attempts to extract LSA secrets from memory-resident registry data.
- `windows.ssdt.SSDT` — Enumerates the System Service Descriptor Table to help spot syscall table anomalies.
- `windows.driverirp.DriverIrp` — Lists driver IRP handlers to help identify IRP hook anomalies.
- `windows.callbacks.Callbacks` — Enumerates kernel callbacks that may indicate persistence or monitoring mechanisms.
- `windows.devicetree.DeviceTree` — Displays driver and device object relationships.
- `windows.desktops.Desktops` — Enumerates desktop objects.
- `windows.deskscan.DeskScan` — Scans for desktop objects in memory, including artifacts not shown by standard enumeration.
- `windows.sessions.Sessions` — Enumerates user logon sessions and session-related structures.
- `windows.symlinkscan.SymlinkScan` — Scans for object manager symbolic links.
- `windows.malware.malfind.Malfind` — Detects suspicious memory regions commonly associated with code injection.
- `windows.vadinfo.VadInfo` — Lists virtual address descriptors for process memory regions.
- `windows.vadyarascan.VadYaraScan` — Scans process VAD regions with YARA rules or strings.
- `windows.malware.svcdiff.SvcDiff` — Compares service information to highlight suspicious service tampering.
- `windows.malware.pebmasquerade.PebMasquerade` — Detects possible process masquerading through suspicious PEB metadata.
- `windows.bigpools.BigPools` — Enumerates large kernel pool allocations.
- `windows.mbrscan.MBRScan` — Scans for Master Boot Record artifacts to help identify bootkits or boot-level tampering.
- `windows.mutantscan.MutantScan` — Scans for mutant (mutex) objects in memory.
- `windows.joblinks.JobLinks` — Enumerates job objects and linked processes.
- `windows.verinfo.VerInfo` — Extracts version information from PE files mapped in memory.
- `windows.orphan_kernel_threads.Threads` — Identifies orphaned kernel threads that may indicate kernel-level anomalies.
- `windows.truecrypt.Passphrase` — Attempts to locate TrueCrypt passphrase-related artifacts in memory.
- `windows.skeleton_key_check.Skeleton_Key_Check` — Checks for indicators associated with Skeleton Key attacks.
- `timeliner.Timeliner` — Produces a unified forensic timeline from supported artifacts.
- `windows.pslist.PsList` — Lists processes linked in the active process list and supports process dumping with `--dump`.
- `windows.strings.Strings` — Reads output from an external `strings` command and indicates which process each string belongs to.
- `windows.cachedump.Cachedump` — Attempts to extract cached credential data when the required artifacts are available.
```
