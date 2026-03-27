# File I/O & Log Processing 

This module demonstrates foundational text processing and file handling techniques in Python. In a DevOps context, these skills are critical for parsing server logs (Nginx, Apache, Syslog), extracting dynamic variables from configuration files, and building resilient data pipelines.

## 📌 Core Concepts

### 1. Memory-Efficient File Traversal
When dealing with large files (e.g., multi-gigabyte production logs), reading the entire file into memory using `.read()` can cause memory exhaustion (OOM). The standard practice is to treat the file handle as an iterable, processing it line-by-line.

```python
# Optimal for large log files: reads one line into memory at a time
with open('server_access.log', 'r') as file_handle:
    for line in file_handle:
        process_line(line)
 ```       
 ### 2. Data Sanitization & Extraction

- Raw file data often contains hidden characters like trailing whitespaces or newline escape sequences (`\n`).
- Sanitization: `line.rstrip()` ensures exact string matching by removing trailing whitespace.
- Targeting: `line.startswith()` or the `in` operator allows for quick filtering.
- Extraction: Combining `.find()` with String Slicing enables precise data extraction without external libraries (like Regex), resulting in highly performant code.

### 3. Flow Control: Negative Logic (Fail-Fast)
- Instead of nesting deep `if ` statements, DevOps scripts utilize "negative logic". By identifying what we do not want and using  `continue `, we skip irrelevant data early, keeping the processing logic flat and readable.

### 4. Exception Handling
- Infrastructure automation must fail gracefully. Using `try/except` blocks prevents the script from crashing abruptly with a traceback when a file is missing, permissions are denied, or input is malformed.

---

## 🛠️ Practical Implementation: Log Parser
- The following script (`log_parser.py`) simulates a real-world scenario: reading a log file, safely handling incorrect file paths, skipping irrelevant lines, and extracting specific telemetry (in this case, domain names from an `mbox` format log).


```python
import sys

def parse_logs():
    """
    Scans a log file, filters for specific events, and extracts target data.
    Demonstrates safe I/O, negative logic, and string slicing.
    """
    file_name = input("Enter log file path: ")
    
    # 1. Resilient I/O: Graceful error handling
    try:
        file_handle = open(file_name, 'r')
    except FileNotFoundError:
        print(f"[CRITICAL] File '{file_name}' not found or inaccessible.")
        sys.exit(1) # Exit with an error code for CI/CD pipelines
        
    target_count = 0
    
    # 2. Memory-efficient line-by-line iteration
    for line in file_handle:
        # Sanitize trailing characters (\n, \t, spaces)
        line = line.rstrip()
        
        # 3. Negative Logic: Skip irrelevant log entries immediately
        if not line.startswith('From '):
            continue
            
        # 4. Data Extraction Pipeline (Find & Slice)
        # Target format: "From user@domain.com timestamp"
        at_pos = line.find('@')
        space_pos = line.find(' ', at_pos)
        
        # Ensure both delimiters were found to avoid index errors
        if at_pos != -1 and space_pos != -1:
            # Slice from the character after '@' up to the next space
            domain = line[at_pos + 1 : space_pos]
            print(f"[INFO] Extracted Target Domain: {domain}")
            target_count += 1
            
    print(f"\n[SUCCESS] Total targets extracted: {target_count}")
    file_handle.close()

if __name__ == '__main__':
    parse_logs()
```

## 🚀 DevOps Use Cases

### The mechanisms demonstrated above are directly applicable to:

- Log Aggregation: Pulling specific error codes or IP addresses from `/var/log/syslog`.
- Config Management: Reading `.env`, `.ini`, or `.conf` files and extracting key-value pairs before deployment.
- Security Auditing: Scanning authentication logs for failed SSH login attempts and extracting the offending IP blocks.

---

### 2.2. Stream Processing & Data Normalization
Normalization is the process of transforming data into a common format. In DevOps, this is crucial for log aggregation where different systems might log the same error in different cases (e.g., "Critical", "CRITICAL", "critical"). 

The following implementation demonstrates a **memory-efficient stream** that processes a file line-by-line, strips trailing metadata (newlines), and normalizes the text to uppercase.

```python
# Implementation: Case Normalization Stream
file_name = input("Enter log file path: ")

try:
    with open(file_name, 'r') as fh:
        for line in fh:
            # Normalize and remove trailing control characters (\n)
            print(line.rstrip().upper())
except FileNotFoundError:
    print(f"Error: {file_name} not found.")
```

### Key DevOps Takeaways:
- Memory Efficiency: By iterating with a `for` loop, we avoid loading the entire file into RAM. The script only holds a single line at any given time, allowing it to process 100GB+ log files on a low-spec microservice.
- Idempotency: The `.rstrip().upper()` pipeline ensures that regardless of the input's original formatting, the output is consistent and predictable.

---

# Log Analysis: Metric Extraction & Accumulation

In modern Site Reliability Engineering (SRE) and DevOps workflows, parsing unstructured log data to calculate performance metrics or health scores is a routine task. This module demonstrates an automated approach to scanning server logs, extracting floating-point telemetry, and calculating statistical averages without relying on high-level built-in sum functions—simulating a low-level stream processing environment.

## 📌 Project Overview
The objective is to process a standardized `mbox` mail server log, identify specific "Spam Confidence" headers, and aggregate their values to determine the overall system health or spam filtering efficacy.

### Technical Challenges:
* **Pattern Matching:** Identifying target lines using `startswith()`.
* **String Parsing:** Utilizing `.find()` and **Slicing** to isolate numerical data within a string.
* **The Accumulator Pattern:** Manually tracking sums and counts to manage memory and logic flow without built-in `sum()` functions.
* **Precision Handling:** Converting string-based telemetry into `float` for mathematical accuracy.

---

## 🛠️ Implementation: `spam_confidence_analyzer.py`

This script follows the **Fail-Fast** principle by validating file existence before proceeding to the heavy-lifting phase.

```python
import sys

def run_analysis():
    # 1. User Input & Validation
    file_name = input("Enter log file path: ")
    
    try:
        # 2. Resource Allocation
        file_handle = open(file_name, 'r')
    except FileNotFoundError:
        print(f"[CRITICAL] IO Error: Target file '{file_name}' inaccessible.")
        sys.exit(1)

    # 3. Initialization (Accumulator Pattern)
    line_count = 0
    total_confidence = 0.0

    # 4. Stream Processing Loop
    for line in file_handle:
        # Negative Logic: Skip irrelevant data
        if not line.startswith("X-DSPAM-Confidence:"):
            continue
        
        # Data Extraction via Slicing
        # Standard Format: "X-DSPAM-Confidence: 0.8475"
        try:
            colon_pos = line.find(":")
            # Extract, clean, and cast to floating point
            value_str = line[colon_pos + 1 :].strip()
            value = float(value_str)
            
            # Aggregation
            total_confidence += value
            line_count += 1
        except ValueError:
            # Handle malformed log entries
            print("[WARN] Skipping malformed numeric data.")
            continue

    # 5. Result Calculation & Final Output
    if line_count > 0:
        average = total_confidence / line_count
        print(f"Average spam confidence: {average}")
    else:
        print("[INFO] No relevant telemetry found in file.")

    file_handle.close()

if __name__ == "__main__":
    run_analysis()
```

## 🚀 DevOps Use Cases
- Service Level Indicators (SLIs): Automatically scanning application logs to calculate average response times or error rates.
- Security Auditing: Analyzing firewall logs to find the average number of blocked packets per session.
- Resource Optimization: Parsing `/proc/meminfo` or similar system files to track average memory pressure over a specific period.
