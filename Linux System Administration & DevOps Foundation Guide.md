# 🚀 Linux System Administration & DevOps Foundation Guide

> **Author:** DevOps Engineering Team
> **Description:** A comprehensive engineering handbook for Linux CLI operations, I/O stream management, data pipelines, and process control. Built on the core Unix philosophy: *"Write programs that do one thing and do it well. Write programs to work together."*

This guide consolidates essential system commands and scripting techniques required for daily cloud infrastructure management, server debugging, and log analysis.

---

## 📂 1. File System Navigation & Operations

Before managing cloud infrastructure, you must master the local file system. Most system configurations and logs are just files. Linux commands typically operate silently on success and only output text upon failure.

### Core Commands
* `pwd`: Print Working Directory.
* `mkdir`: Create a new directory.
* `cd`: Change directory (`.` is current, `..` is parent).
* `touch`: Create an empty file or update timestamps.
* `ls -la`: List all files (including hidden `.` files) with detailed attributes.
* `cp`: Copy files/directories.
* `mv`: Move or rename files/directories.
* `rm` / `rmdir`: Remove files / empty directories.

### 🛠️ DevOps Scenario: Preparing Configuration Files
Imagine you are setting up a new web application configuration.

```bash
# Navigate to your workspace and create a config directory
mkdir -p /opt/app/config
cd /opt/app/config

# Check where we are
pwd
# Output: /opt/app/config

# Copy a template configuration file from a parent directory
cp ../templates/nginx.conf.template ./nginx.conf

# Rename the file to mark it as active
mv nginx.conf active_nginx.conf

# Create an empty local environment file
touch .env.local

# View detailed permissions and inodes
ls -la
# Output will show ownership, group, size, and permissions (e.g., -rw-r--r--)
```

---

## 🔐 2. Permissions, Ownership & System Info
### Security is paramount in DevOps. You must control who can read, write, or execute configuration files and scripts.
- `chmod`: Change file permissions (e.g., making a script executable).
- `chown` / `chgrp`: Change file owner and group.
- `id`: Print current user and group identities (useful for debugging Permission denied errors).
- `free -h`: Check system memory in human-readable format.
```bash
# Check your current user privileges
id
# Output: uid=1000(devops) gid=1000(devops) groups=1000(devops),999(docker)

# Check server RAM usage before deploying a heavy container
free -h

# Make a deployment script executable and move it to a global binary path
chmod +x deploy.sh
sudo mv deploy.sh /usr/local/bin/deploy

# Give web server user ownership of the HTML directory
sudo chown -R www-data:www-data /var/www/html/
```

---

## 🔀 3. I/O Stream Redirection (Data Flow)
### Every process in Linux has three default standard streams:

- STDIN (0): Standard Input (Default: Keyboard)
- STDOUT (1): Standard Output (Default: Terminal display)
- STDERR (2): Standard Error (Default: Terminal display)
- We can redirect these streams to files using `>`  (overwrite), `>>` (append), and `<` (input).

### 🛠️ DevOps Scenario: Capturing Deployment Logs and Errors
- When running deployment scripts or Python applications, you want to log successes to one file and errors to another for monitoring.

```bash
# Overwrite a file with a basic status message
echo "Deployment started at $(date)" > deploy_status.txt

# Append standard output (STDOUT) to a log file
./run_migrations.py >> app_database.log

# Redirect STDERR (file descriptor 2) to a specific error log
./start_server.py 2> server_errors.log

# Feed a file into a script via STDIN
./process_data.py < data_dump.csv

# Advanced: Combine both STDOUT and STDERR to the SAME file
./flaky_script.sh > debug.log 2>&1
```

---

## 🚰 4. Data Pipelines & Text Processing
### Piping (`|`) connects the STDOUT of one command directly to the STDIN of another. This allows us to chain simple commands together to perform complex data analysis without creating temporary files.

- Core Processing Tools:
- `cat`: Concatenate and print files.
- `less`: Pager to view large files.
- `grep`: Search for patterns.
- `cut`: Extract sections from lines (e.g., columns).
- `tr`: Translate or delete characters.
- `sort`: Sort lines of text files.
- `uniq`: Report or omit repeated lines.
- `head` / `tail`: Output the first/last parts of files.

### 🛠️ DevOps Scenario: Analyzing Web Server Logs to Find Top Attacking IPs
- Let's say you want to find out which IP addresses are hitting your server the most by analyzing an `access.log`.

```bash
# Pipeline breakdown:
# 1. Read the log file
# 2. Extract the 1st column (IP address) separated by spaces
# 3. Sort the IPs alphabetically (required for uniq)
# 4. Count unique occurrences
# 5. Sort numerically (-n) in reverse/descending order (-r)
# 6. Show the top 5 results

cat /var/log/nginx/access.log | cut -d' ' -f1 | sort | uniq -c | sort -nr | head -5

# Example Output:
# 1543 192.168.1.50
#  890 10.0.0.12
#  430 172.16.0.5
```

### Extending Pipelines with Python
- You can write Python scripts that natively hook into these Bash pipelines using sys.stdin.
- `format_logs.py` (Make error keywords uppercase):

```bash
#!/usr/bin/env python3
import sys

# Read line by line from STDIN (Bash pipe)
for line in sys.stdin:
    line = line.strip()
    if "error" in line.lower():
        print(f"🚨 {line.upper()}")
    else:
        print(f"✅ {line}")
```

### Using it in the pipeline:

```bash
# Pipe server logs into our custom Python formatter
cat /var/log/syslog | grep "nginx" | ./format_logs.py
```

---

## 🚦 5. Process Management & Signals
### System engineers must manage background jobs, find memory-hogging processes, and gracefully stop or forcibly kill applications. Signals are tokens sent to processes to trigger a specific behavior.

- Essential Signals:
- `Ctrl-C` (SIGINT): Interrupt. Asks the process to finish cleanly and exit.
- `Ctrl-Z` (SIGSTOP): Suspend. Pauses the process without killing it.
- SIGTERM (Default `kill`): Terminate. Polite request to shut down (allows for cleanup).
- SIGKILL (`kill -9`): Forcibly kills a process immediately (use as last resort).

### Core Process Tools:
- `ps ax`: List all running processes.
- `to`p: Real-time interactive dashboard of system resource usage.
- `fg` / `bg`: Bring a stopped job to the foreground / send to background.
- `jobs`: List current suspended or background jobs in the terminal session.

## 🛠️ DevOps Scenario: Debugging and Killing a Rogue Node.js Service
### Imagine a Node.js microservice has frozen and is consuming 100% CPU.

```bash
# 1. Use 'top' to identify high CPU usage (Press 'q' to exit)
top

# 2. Find the exact Process ID (PID) of the node application
ps ax | grep node
# Output: 
# 45192 pts/0  Sl+  0:05 node server.js
# 45201 pts/1  S+   0:00 grep --color=auto node

# 3. Send a polite SIGTERM to let it clean up open DB connections
kill 45192

# 4. If it is completely unresponsive, force kill it with SIGKILL
kill -9 45192
```

### Backgrounding a Task
- If you start a long-running database backup (pg_dump) and realize you need your terminal back:

```bash
# 1. Start the backup
./long_database_backup.sh

# 2. Press Ctrl-Z to send SIGSTOP and pause the process
# Output: [1]+  Stopped   ./long_database_backup.sh

# 3. Resume the process in the background
bg

# 4. Check on it later
jobs
```


---


# 🚀 DevOps Engineering Handbook - Part 2: Advanced Bash Scripting & Automation

> **Author:** DevOps Engineering Team
> **Description:** A professional guide to Bash scripting for cloud infrastructure automation, system diagnostics, and bulk file operations. This document covers variables, conditional logic, loops, and robust error handling.

Bash is not just a command interpreter; it is a powerful scripting language. System engineers rely on Bash to automate repetitive tasks, orchestrate deployments, and construct resilient system checks. 

---

## 🛠️ 1. Automated System Diagnostics

When responding to an incident, running diagnostic commands (`free`, `uptime`, `ps`) manually is inefficient. A professional engineer compiles these into a single executable script to instantly capture a node's health snapshot.

### `node_health_check.sh`
```bash
#!/bin/bash
# -----------------------------------------------------------------------------
# Purpose: Generates a quick health snapshot of the current server node.
# Usage: ./node_health_check.sh
# -----------------------------------------------------------------------------

# Define a variable for clean formatting (Note: No spaces around '=' in Bash)
DIVIDER="---------------------------------------------------------"

echo "Starting Diagnostic at: $(date)"
echo $DIVIDER

echo "[UPTIME & LOAD AVERAGE]"
uptime
echo $DIVIDER

echo "[MEMORY USAGE]"
free -h
echo $DIVIDER

echo "[ACTIVE USERS & SESSIONS]"
who
echo $DIVIDER

echo "Diagnostic Completed at: $(date)"
```


- Engineering Rationale: Grouping commands and executing them sequentially in a script saves crucial seconds during an outage. Using `$(command)` allows us to evaluate a command and inject its output directly into strings.

---

## 🗂️ 2. Variables & Globbing (Pattern Matching)
- Variables store dynamic data. In Bash, you define them strictly without spaces (`ENV="production"`) and reference them using the `$` prefix (`echo $ENV`).
- Globs (`*` and `?`) are wildcard characters used by the shell to match filenames. They are essential for targeting specific infrastructure files without hardcoding names.

```bash
# Match ALL Python scripts in the directory
echo *.py

# Match files starting with 'c' (e.g., config files)
echo c*

# Match exact character counts using '?' (e.g., find 5-letter configuration files)
echo ?????.conf
```

---

## 3. Conditional Logic & Exit Status
### A core tenet of automation is reacting to system states. Bash uses the Exit Status of a command to determine truth (Exit Status 0 = Success, anything else = Failure).

- Scenario: Checking for Critical Errors in Logs
- Instead of just checking if a local IP exists, let's verify if an application is throwing database connection errors.
```bash
#!/bin/bash

LOG_FILE="/var/log/app/production.log"
ERROR_PATTERN="DB_CONNECTION_REFUSED"

# The 'if' statement evaluates the exit status of 'grep'
if grep -q "$ERROR_PATTERN" "$LOG_FILE"; then
    echo "🚨 ALERT: Database connection errors detected in $LOG_FILE!"
    # Here you would typically trigger an alert (e.g., via Slack or PagerDuty)
else
    echo "✅ System stable: No database errors found."
fi
```

### The `test` Command and `[` Alias
- For comparing strings, numbers, or checking if files exist, Bash provides the test utility, most commonly used via its alias `[ ]`.
```bash
# Check if a critical environment variable is empty
if [ -z "$DATABASE_URL" ]; then
    echo "FATAL: DATABASE_URL is not set. Aborting deployment."
    exit 1
fi

# Check if a directory exists
if [ -d "/etc/nginx" ]; then
    echo "Nginx configuration directory found."
fi
```
---

## 4. While Loops: Implementing Retry Logic
### Network calls, API requests, and database connections can fail transiently. A senior engineer writes scripts that expect failure and implement backoff/retry logic using `while` loops.

- `wait_for_service.sh`
- This script attempts to hit a service endpoint up to 5 times before failing.

```bash
#!/bin/bash
# -----------------------------------------------------------------------------
# Purpose: Executes a command with a retry mechanism and progressive delay.
# Usage: ./wait_for_service.sh <command>
# -----------------------------------------------------------------------------

ATTEMPT=1
MAX_ATTEMPTS=5
# $1 captures the first argument passed to the script
TARGET_CMD=$1 

# Loop WHILE the command fails (!) AND attempts are less than or equal to MAX
while ! $TARGET_CMD && [ $ATTEMPT -le $MAX_ATTEMPTS ]; do
    echo "⚠️ Command failed. Retrying in $ATTEMPT seconds... (Attempt $ATTEMPT/$MAX_ATTEMPTS)"
    sleep $ATTEMPT
    ((ATTEMPT+=1)) # Arithmetic expansion to increment
done

if [ $ATTEMPT -gt $MAX_ATTEMPTS ]; do
    echo "❌ ERROR: Command permanently failed after $MAX_ATTEMPTS attempts."
    exit 1
else
    echo "✅ SUCCESS: Command executed successfully."
fi
```

- Usage Example:
```bash
# Wait for a database container to start responding to ping
./wait_for_service.sh "ping -c 1 10.0.0.5"
```

## 5. For Loops & Bulk Operations
### `for` loops iterate over a sequence of items. Combined with globbing, they are highly effective for bulk file modifications, such as normalizing legacy log files or updating configuration extensions.

- Scenario: Normalizing Legacy File Extensions
- Imagine migrating a legacy system where old template files end in uppercase `.HTM`. The modern CI/CD pipeline requires them to be lowercase `.html`.

```bash
#!/bin/bash
# -----------------------------------------------------------------------------
# Purpose: Bulk rename legacy .HTM files to .html
# -----------------------------------------------------------------------------

for file in *.HTM; do
    # 'basename' strips the directory path and the specified suffix
    name=$(basename "$file" .HTM)
    
    # DEV SECRETS: The "Dry Run"
    # Always prepend destructive commands (mv, rm) with 'echo' first to 
    # verify the script's behavior safely.
    
    # echo mv "$file" "${name}.html"
    
    # Once verified, remove 'echo' to execute:
    mv "$file" "${name}.html"
done

echo "✅ File normalization complete."
```

# 📊 Practical Example: Log File Analysis

System administrators view `/var/log/syslog` as a treasure trove of information. However, because these files are extremely dense, you must use a **pipeline** to filter the data effectively.

### The Pipeline Logic:
1.  **`cut -d' ' -f5-`**: Splits the line by spaces and extracts everything from the 5th field onward (removing the date and hostname to focus on the message).
2.  **`sort`**: Groups identical lines together (essential for the next step).
3.  **`uniq -c`**: Collapses identical lines and provides a count of occurrences.
4.  **`sort -nr`**: Sorts the results numerically (`n`) and in reverse (`r`), placing the most frequent events at the top.
5.  **`head`**: Displays only the top results.

---

## 🔄 Scaling: Automating All Logs

To process more than one file, we apply a `for` loop to scan every `.log` file within the `/var/log` directory.

### `top_log_lines.sh`
```bash
#!/bin/bash

# Iterate over all .log files
for logfile in /var/log/*log; do
    echo "--------------------------------------------------"
    echo "Processing: $logfile"
    echo "--------------------------------------------------"
    
    # Find the top 5 most frequent messages for each file
    cut -d' ' -f5- "$logfile" | sort | uniq -c | sort -nr | head -5
done
```

## The "Unreadable Bash" Example
### If you are writing a Bash command like this just to capitalize words in a text, the risk of bugs is high:
- `for i in $(cat story.txt); do B=$(echo -n "${i:0:1}" | tr "[:lower:]" "[:upper:]"); echo -n "${B}${i:1} "; done`

- By contrast, the Python version is much more professional:
```bash
import sys
for line in sys.stdin:
    print(" ".join([word.capitalize() for word in line.split()]))
```
