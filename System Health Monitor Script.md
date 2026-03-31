## 🖥️ System Health Monitor Script

### 💻 The Source Code
- Here is the implementation of our automated health check:
```python
#!/usr/bin/env python3  # Shebang: Locates the Python 3 interpreter

import shutil
import psutil

def check_disk_usage(disk):
    """Calculates the free disk space percentage."""
    du = shutil.disk_usage(disk)
    free = du.free / du.total * 100
    return free > 20  # Returns True if more than 20% is free

def check_cpu_usage():
    """Calculates average CPU usage over a 1-second interval."""
    usage = psutil.cpu_percent(1) # Samples for 1 second to avoid spikes
    return usage < 75  # Returns True if usage is under 75%

# --- Main Logic Block ---
if not check_disk_usage("/") or not check_cpu_usage():
    print("ERROR!")
else:
    print("Everything is OK!")
```

## 🔍 Deep Dive: Code Analysis
- A professional engineer must understand exactly how their tools work. Here is the breakdown of the logic:

### 1️⃣ The Shebang #!/usr/bin/env python3
- Purpose: This line ensures the script is executable as a standalone program in Unix-like environments (Linux/WSL).
- Why /usr/bin/env? It dynamically finds the path to python3, making the script portable across different systems.

### 2️⃣ Disk Space Monitoring (shutil)
- Function: check_disk_usage("/")
- Mechanism: It uses the shutil.disk_usage module to get total, used, and free bytes.
- Threshold: We use a 20% safety margin. If free space drops below 20%, it flags a warning. This prevents system crashes caused by full disks.

### 3️⃣ CPU Utilization (psutil)
- Function: check_cpu_usage()
- Mechanism: It utilizes psutil.cpu_percent(1). The 1 represents a 1-second delay.
- Why the Delay? Measuring CPU for a millisecond is inaccurate due to volatile spikes. A 1-second average provides a reliable metric of the actual system load.
- Threshold: If CPU load exceeds 75%, the function returns False.

### 4️⃣ Logic Flow
- The if not ... or not ... structure ensures that if either resource fails its health check, the system immediately reports an ERROR!.

---

## 🛠️ Installation & Requirements
- Prerequisites
- You must have Python 3 and the psutil library installed.

```python
# Install psutil via pip
pip install psutil
```

### Running the Script
- Grant Permissions:
```python
chmod +x system_health_check.py
```

### Execute:
```python
./system_health_check.py
```

## 🚀 Future Roadmap
- [ ] Email Alerts: Integrate smtplib to send emails when an error is detected.
- [ ] Logging: Implement a log file to track history (Audit Trail).
- [ ] Dashboard: Create a simple web view for real-time monitoring.
