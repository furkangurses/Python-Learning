# 🛠️ System Engineering & Automation with Python

Welcome to the **System Engineering & Automation** module. This repository contains professional-grade Python patterns for interacting with the underlying operating system. As DevOps engineers, we don't just write code; we build bridges between applications, hardware, and the shell environment.

This guide covers everything from capturing user configuration dynamically, to orchestrating native Linux binaries, handling I/O streams, and managing exit states for CI/CD pipelines.

---

## 📑 Table of Contents
1. [Interactive User Input](#1-interactive-user-input)
2. [I/O Streams: STDIN, STDOUT, & STDERR](#2-io-streams-stdin-stdout--stderr)
3. [Environment Variables & CLI Arguments](#3-environment-variables--cli-arguments)
4. [Subprocess Management & Output Capturing](#4-subprocess-management--output-capturing)

---

## 1. Interactive User Input
While automation is the ultimate goal, CLI tools often require interactive prompts for dangerous or specific operations (like destroying infrastructure or provisioning specific environments). We use the native `input()` function to interact with the operator.

### 💻 `interactive_provision.py`
This script demonstrates how to capture and cast operator input to build a dynamic configuration loop.

```python
#!/usr/bin/env python3

def calculate_resources(instances, cores_per_instance):
    """Calculates total vCPU requirements."""
    return instances * cores_per_instance

print("=== ☁️ Cloud Resource Provisioning Tool ===")

cont = "y"
while cont.lower() == "y":
    try:
        # Taking input and casting to integer for calculations
        instances = int(input("Enter the number of EC2 instances: "))
        cores = int(input("Enter vCPUs per instance: "))
        
        total_vcpu = calculate_resources(instances, cores)
        print(f"[INFO] Allocation approved: {total_vcpu} total vCPUs required.\n")
        
    except ValueError:
        print("[ERROR] Invalid input. Please enter numerical values.")
        
    # Interactive prompt to continue or break the loop
    cont = input("Do you want to provision another cluster? [y/N]: ")

print("[SYSTEM] Exiting provisioning tool. Goodbye!")
```

### 🧠 Logic: The `input()` function always returns a string. In systems programming, we often need integers (for ports, memory, instances), so casting via `int()` is required. We wrap it in a `while` loop to create a persistent CLI session.

---

### 2. I/O Streams: STDIN, STDOUT, & STDERR
- A Python script doesn't exist in a vacuum; it connects to the OS via streams.
- STDIN: How data flows into the program (e.g., piping logs from another command).
- STDOUT: Standard output for expected results.
- STDERR: The diagnostic channel strictly reserved for errors and warnings.

## 💻 `stream_router.py`
- This script demonstrates how to separate standard logging from error reporting.

```python
#!/usr/bin/env python3
import sys

# Reading from STDIN (Allows piping: echo "Server 01" | ./stream_router.py)
# Fallback to input() if not piped
if not sys.stdin.isatty():
    data = sys.stdin.read().strip()
else:
    data = input("Enter target hostname: ")

# Writing standard execution logs to STDOUT
sys.stdout.write(f"[INFO] Initiating connection to {data}...\n")
sys.stdout.write(f"[SUCCESS] Handshake with {data} established.\n")

# Writing critical alerts to STDERR (This allows operators to filter errors in bash: 2> error.log)
sys.stderr.write(f"[CRITICAL] {data} is reporting high CPU utilization (>95%)!\n")
```

### 🧠 Logic: By explicitly routing errors to `STDERR` (rather than just `print()`), system administrators can use bash redirection (e.g., `./script.py > logs.txt 2> alerts.log)` to monitor infrastructure health effectively.

---

### 3. Environment Variables & CLI Arguments
- To make scripts portable and suitable for CI/CD pipelines (like Jenkins or GitHub Actions), we must eliminate hard-coded values. We achieve this using Environment Variables (`os.environ`) and Command-Line Arguments (`sys.argv`).

## 💻 `deploy_config.py`
- This script reads the target environment from the OS and uses a configuration file passed as an argument. It also manages Exit Codes, which tell the shell if the process succeeded (`0`) or failed (`1`).

```python
#!/usr/bin/env python3
import os
import sys

# 1. Fetching Environment Variables using .get() for safe defaults
target_env = os.environ.get("DEPLOY_ENV", "staging")
print(f"[INIT] Target Deployment Environment: {target_env.upper()}")

# 2. Handling Command Line Arguments
# sys.argv[0] is the script name. sys.argv[1] is the first passed parameter.
if len(sys.argv) < 2:
    print("[FATAL] Missing configuration file argument.")
    print("Usage: ./deploy_config.py <config_file.json>")
    sys.exit(1) # Exit Code 1 tells the shell the script failed

config_file = sys.argv[1]

# 3. Validating State and Returning Exit Codes
if not os.path.exists(config_file):
    with open(config_file, "w") as f:
        f.write('{"status": "initialized"}\n')
    print(f"[SUCCESS] Configuration file '{config_file}' generated.")
    sys.exit(0) # Exit Code 0 = Success
else:
    # Writing to STDERR and failing the pipeline
    sys.stderr.write(f"[ERROR] Conflict: '{config_file}' already exists. Aborting overwrite.\n")
    sys.exit(1)
```

### 🧠 Logic: The `os.environ.get()` method is safe; if the variable isn't set, it provides a default instead of crashing. `sys.exit()` is crucial for DevOps: if your script fails and returns `0` by mistake, the pipeline will continue and deploy broken code. Always return `1` on failure!

---


### 4. Subprocess Management & Output Capturing
- Sometimes, native Python libraries aren't enough, and you need to leverage system binaries (like `ping`, `host`, `docker`, or `kubectl`). The `subprocess` module allows us to spawn child processes, control them, and capture their output for Python to process.

## 💻 network_audit.py
- This script uses the system's `host` and `rm` commands to demonstrate capturing output and decoding bytes, which is vital for network and file system audits.

```python
#!/usr/bin/env python3
import subprocess

target_domain = "google.com"

print(f"--- Running DNS Audit on {target_domain} ---")

# 1. Executing a system command and CAPTURING the output
# capture_output=True prevents it from printing directly to the terminal
result = subprocess.run(["host", target_domain], capture_output=True)

# 2. Checking the Return Code (0 = command successful)
if result.returncode == 0:
    # stdout returns a Byte Array (b'string'). We must .decode() it to a UTF-8 string.
    raw_output = result.stdout.decode()
    
    # Processing the output: Splitting the string to extract the actual IP addresses
    lines = raw_output.strip().split('\n')
    ip_address = lines[0].split()[-1]
    print(f"[SUCCESS] Extracted Primary IP: {ip_address}")
else:
    print(f"[ERROR] DNS lookup failed with exit code: {result.returncode}")


print("\n--- Running File System Cleanup Audit ---")

# 3. Handling STDERR from a subprocess
# We intentionally try to delete a non-existent file to capture the error stream
clean_result = subprocess.run(["rm", "/tmp/ghost_file.lock"], capture_output=True)

if clean_result.returncode != 0:
    # Capturing and decoding the STDERR stream
    error_msg = clean_result.stderr.decode().strip()
    print(f"[WARNING] Cleanup encountered an issue: {error_msg}")
```

### 🧠 Logic: `* subprocess.run()` blocks the parent process until the command finishes.
- We pass the command as a list of strings (`["command", "flag", "target"]`) to prevent shell injection vulnerabilities.
- System commands return Bytes. We MUST use `.decode()` to translate those bytes into a standard UTF-8 string before we can apply Python string methods like `.split()`.
- We check `result.returncode` directly within Python to make routing decisions based on the system command's success or failure.

---
