# Process Resource Monitor

This script monitors the resource usage of a program while it runs.  
It tracks **CPU usage, memory consumption, thread count, and I/O statistics** in real time, then reports peak and average values once the process finishes.

---

## ⚙️ Features
- Monitors:
  - CPU usage (per core normalized)
  - RAM usage (RSS)
  - Thread count
  - I/O read and write bytes
- Reports:
  - Execution time
  - Peak and average values for each metric
- Human-readable byte conversion (KB, MB, GB, etc.)

---

## 📦 Requirements
- Python 3.7+
- [psutil](https://pypi.org/project/psutil/) library

Install dependencies:
```bash
pip install psutil
