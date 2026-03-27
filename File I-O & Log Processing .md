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
