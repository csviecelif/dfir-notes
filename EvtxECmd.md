# EvtxECmd

---

**EvtxECmd**, part of the Zimmerman tools suite, is used to parse Windows Event Log files (`.evtx`) and convert them into `.csv` format for easier analysis.

---

## 🔧 Basic Command

```bash
EvtxECmd.exe -d "<directory_with_evtx_files>" --csv "<output_directory>" --csvf "<output_filename.csv>"
```

---

## 🧾 Parameters

- `-d` → Directory containing `.evtx` files  
- `--csv` → Output directory for the generated `.csv` file  
- `--csvf` → Output file name  

---

## 📊 Data Analysis

After conversion:

1. Open **Timeline Explorer** (Eric Zimmerman tool)  
2. Load the generated `.csv` file  
3. Use filtering and sorting features to analyze events  
```
