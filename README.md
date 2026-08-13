# QNX IFS Memory Map Stacked Cake Visualizer

A lightweight Python tool to parse QNX `dumpifs` output logs and generate an interactive **Stacked Cake Memory Map Visualization** in a single standalone HTML file.

---

## 🚀 Features

- **🍰 Stacked Cake Memory Diagram**: Color-coded visual layers representing QNX IFS memory regions ordered from Base Address (`0x80000`) up to Top Address (`0x2904408`).
- **🏷️ Smart Categorization**: Automatic classification into core memory sections:
  - 🔴 **Boot & Header** (`*.boot`, `startup.*`, `Image-header`, `Image-directory`)
  - 🟣 **Kernel & OS Core** (`procnto-smp-instr`, `startup-script`)
  - 🟠 **Drivers & Modules** (`devs-qwdi...`, `devs-em`, `gpio-*`, `mbox-*`)
  - 🔵 **Shared Libraries** (`libc.so`, `libcrypto.so`, `libc++.so`, `libssl.so`, `libz.so`)
  - 🟢 **Executables & Utilities** (`toybox`, `wpa_supplicant`, `hostapd`, `sshd`, `ksh`, `pidin`)
  - 🟢 **Firmware & Configs** (`firmware.bin`, `pcidatabase...`, `services`, `wpa_supplicant.conf`)
- **🔍 Interactive Search & Filter**: Instant search filtering (e.g. search `sshd`, `crypto`, `kernel`) and category pill filters.
- **📊 Summary Footprint Metrics**: Total IFS size, module count, largest module breakdown, and SHA512/Checksum validation info.
- **⚡ Lightweight & Zero Dependencies**: Built using Python 3 standard library (`sys`, `re`, `json`, `argparse`, `html`). **No third-party packages or server setup required.**

---

## 📁 Files Included

- [`visualize_ifs.py`](file:///home/cp/QNX_Practice/visualize_ifs.py) — Lightweight Python visualizer script
- [`dumpifs.log`](file:///home/cp/QNX_Practice/RequirementPSM/dumpifs.log) — Raw output from QNX `dumpifs` command on `ifs-rpi5.bin`
- [`ifs_memory_map.html`](file:///home/cp/QNX_Practice/RequirementPSM/ifs_memory_map.html) — Output standalone interactive visualizer

---

## 💻 How to Use

### 1. Generate Visualization
From the workspace directory (`/home/cp/QNX_Practice`), run:

```bash
python3 visualize_ifs.py RequirementPSM/dumpifs.log -o RequirementPSM/ifs_memory_map.html
```

### 2. View the Visual Memory Map
Open the generated `RequirementPSM/ifs_memory_map.html` file in any web browser (Chrome, Firefox, Edge, Safari):

```bash
# Example: Open in default browser on Linux
xdg-open RequirementPSM/ifs_memory_map.html
```

---

## 🛠️ Customization
You can pass any QNX `dumpifs` output log file to the tool:

```bash
python3 visualize_ifs.py /path/to/any_dumpifs.log -o output_memory_map.html
```
