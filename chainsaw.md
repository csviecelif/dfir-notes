# Chainsaw

**Chainsaw** is a fast tool for hunting through Windows Event Logs using Sigma detection rules. It allows you to quickly identify suspicious activity from `.evtx` files.

---

## 📥 Download Sigma Rules

Chainsaw relies on Sigma rules for detections. You can download them from the official repository:

- GitHub: https://github.com/SigmaHQ/sigma

After downloading, use the `rules` folder as your Sigma path (or adjust as needed).

---

## 🔧 Basic Command

```bash
chainsaw.exe hunt "<path_to_evtx_or_directory>" -s "<sigma_rules_path>" --mapping "<mapping_file>" --output "<output_file.csv>" --csv
```

---

## 📌 Example (Windows)

```bash
.\chainsaw.exe hunt "...\\AD-01\\Evtx" -s "...\\sigma\\rules" --mapping "...\\chainsaw\\mappings\\sigma-event-logs-all.yml" --output "...\\AD-01\\chainsaw_hunt.csv" --csv
```

---

## 🧾 Parameters

- `hunt` → Mode used for detection via Sigma rules  
- `-s` → Path to Sigma rules directory  
- `--mapping` → Mapping file to normalize event fields  
- `--output` → Output file path  
- `--csv` → Exports results in `.csv` format  

---

## 🧠 Understanding the Mapping File

Sigma rules are written in a **generic format**, while Windows Event Logs (`.evtx`) use specific field names.

The mapping file bridges this gap by translating Windows event fields into the format expected by Sigma rules.

---

### Why use `sigma-event-logs-all.yml`?

- Designed for **Windows Event Logs (.evtx)**  
- Covers multiple sources (Security, Sysmon, PowerShell, etc.)  
- Improves compatibility and detection accuracy  

Without mapping, many Sigma rules may fail to match correctly.

---

## 📊 Analysis Workflow

1. Run the `hunt` command against `.evtx` files or directories  
2. Review detections directly in the terminal output  
3. Open the generated `.csv` file in **Timeline Explorer (Eric Zimmerman)**  
4. Filter and pivot on detections for deeper investigation  

---

## ⚡ Notes

- Ensure your Sigma rules are up to date  
- Use the correct mapping file for Windows Event Logs (`sigma-event-logs-all.yml`)  
- Chainsaw supports both single `.evtx` files and directories  
```
