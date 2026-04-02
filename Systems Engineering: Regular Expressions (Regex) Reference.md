# 🛡️ Systems Engineering: Regular Expressions (Regex) Reference

This documentation provides a technical framework for implementing Regular Expressions using the Python `re` module. It is designed for engineers handling high-throughput log parsing, security auditing, and infrastructure automation.

**Engineering Principle:** Avoid brittle string methods (e.g., `.index()` or hardcoded indexing) which fail when data formats shift. Regex creates a **pattern-based state machine** that ensures system stability and data integrity.

---

## 🛠️ 1. Architecture & Production Standards

For production-grade reliability, all Regex implementations must follow these two standards:

1.  **Raw String Notation (`r"..."`):** Always prefix patterns with `r`. This prevents the Python interpreter from misinterpreting backslashes (e.g., ensuring `\b` is read as a Regex word boundary rather than a string backspace).
2.  **Defensive Programming (`None` Validation):** `re.search()` returns `None` if no match is found. You **must** verify the result before calling methods like `.group()` to avoid an `AttributeError`.

---

## ⚙️ 2. Deterministic Anchors & Metacharacters

Anchors define boundaries to prevent "false positive" matches within dense data streams.

| Token | Technical Definition | Systems Scenario |
| :---: | :--- | :--- |
| `^` | **Start Anchor** | Identifying logs starting with specific `ISO-8601` timestamps. |
| `$` | **End Anchor** | Verifying that a stream ends with a valid `EXIT_CODE` or `SIGTERM`. |
| `.` | **Wildcard** | Matching any single character in a dynamic segment of a serial number. |
| `[ ]` | **Character Class** | Validating specific hex ranges (e.g., `[a-f0-9]` for MAC addresses). |
| `[^ ]` | **Negated Class** | Filtering out all non-numeric characters from a port configuration. |
| `|` | **Logic OR** | Switching between protocols (e.g., `TCP\|UDP\|ICMP`). |



### 2.1. Special Sequences (Shorthands)

* `\w` : Alphanumeric + Underscore (Ideal for `HOSTNAME` or `USER_ID`).
* `\d` : Digits only (Used for `PORT`, `PID`, or `UID`).
* `\s` : Whitespace (Used to parse space-delimited log columns).
* `\b` : Word Boundary (Prevents partial matches in IDs or Hex codes).

---

## 📈 3. Quantifiers: Controlling Precision

Quantifiers define the frequency of a match. In systems engineering, precision prevents "Greedy" patterns from consuming the entire log line.

| Qualifier | Frequency | Behavior |
| :---: | :--- | :--- |
| `*` | 0 or more | **Greedy:** Matches the longest possible string. Use with caution. |
| `+` | 1 or more | **Mandatory:** Ensures a required field (e.g., `IP_ADDR`) is present. |
| `?` | 0 or 1 | **Optionality:** Used for optional headers (e.g., `https?`). |
| `{n,m}` | Range | **Fixed Bound:** `\d{1,5}` for valid port ranges (1-65535). |



---

## 🚀 4. Production Implementation Scenarios

### Scenario A: High-Precision PID Extraction
Extracting a Process ID (PID) from kernel logs using capturing groups.

```python
import re

def extract_pid(log_entry: str) -> str:
    """Extracts PID from [digit] format. Returns empty string if failed."""
    # Pattern: Escaped brackets \[ \] with a digit capturing group (\d+)
    pattern = r"\[(\d+)\]"
    result = re.search(pattern, log_entry)
    
    if result is None:
        return ""
        
    return result[1] # Returns the first captured group

# Example
log = "Apr 02 21:00:05 srv-prod-01 kernel: [54321] Out of memory"
print(f"Extracted PID: {extract_pid(log)}") # Output: 54321
```

### Scenario B: System Identifier Validation
- Validating if a string meets the criteria for a system variable (Must start with a letter/underscore, followed by alphanumeric/underscores).

```python
import re

def validate_identifier(name: str) -> bool:
    """Checks if a string is a valid system-level identifier."""
    # ^ : Start, [a-zA-Z_] : Start char, [a-zA-Z0-9_]* : Tail, $ : End
    pattern = r"^[a-zA-Z_][a-zA-Z0-9_]*$"
    return bool(re.search(pattern, name))

# Tests
print(validate_identifier("DB_PORT_5432")) # True
print(validate_identifier("2_invalid_var")) # False (Starts with digit)
```

### Scenario C: PII Masking (Log Sanitization)
- Using re.sub to redact sensitive data (e.g., emails) before sending logs to an external monitoring stack (ELK/Splunk).
```python
import re

def sanitize_logs(raw_log: str) -> str:
    """Redacts email addresses to maintain security compliance (GDPR/SOC2)."""
    email_pattern = r"[\w.+-]+@[\w.-]+\.[a-zA-Z]{2,}"
    return re.sub(email_pattern, "[REDACTED_IDENTITY]", raw_log)

# Example
raw_entry = "CRITICAL: Access denied for sys_admin@infrastructure.io from 10.0.0.5"
print(sanitize_logs(raw_entry))
```

## 🛠️ 3. Advanced Implementation (Python re Module)
### High-Precision IP & Port Extraction
- Targeting network patterns instead of arbitrary strings ensures accurate infrastructure monitoring.
```python
import re

def extract_network_identities(log_entry: str):
    """
    Identifies and extracts IPv4 addresses and associated port numbers.
    Pattern Logic: 4 octets (1-3 digits) followed by an optional port.
    """
    # Regex: \b for boundary, \d{1,3} for octets, literal dots \.
    ip_pattern = r"\b(?:\d{1,3}\.){3}\d{1,3}\b"
    
    matches = re.findall(ip_pattern, log_entry)
    return matches

# Example Production Log
log = "ERROR 2026-04-02 Source: 192.168.1.15 Destination: 10.0.0.50 Port: 8080"
print(f"Detected IPs: {extract_network_identities(log)}")
```

### Capturing Groups & Log Scrubbing (re.sub)
- In compliance-heavy environments (GDPR/SOC2), scrubbing PII (Personally Identifiable Information) like emails from logs is mandatory.
```python
import re

def sanitize_logs(raw_log: str) -> str:
    """Redacts email addresses from system logs using regex substitution."""
    # Pattern: Captures email structure and replaces it with a generic label
    email_pattern = r"[\w.+-]+@[\w.-]+\.[a-zA-Z]{2,}"
    
    return re.sub(email_pattern, "[REDACTED_EMAIL]", raw_log)

log_line = "User user_admin@company.com accessed root at 21:00"
print(sanitize_logs(log_line)) 
# Output: User [REDACTED_EMAIL] accessed root at 21:00
```

### Fault-Tolerant PID Extraction
- Extracting a Process ID (PID) requires capturing groups () and a safety check against NoneType results.
```python
import re

def get_pid_from_log(line: str) -> str:
    """Extracts PID enclosed in brackets. Returns empty string if no match."""
    # Pattern: Escaped brackets \[ \] with a digit capturing group (\d+)
    pattern = r"\[(\d+)\]"
    result = re.search(pattern, line)
    
    # Defensive Check: Prevent AttributeError if search returns None
    if result is None:
        return ""
        
    return result[1] # Return the first captured group

# Tests
print(f"PID Found: {get_pid_from_log('kernel: [12345] protocol error')}") # '12345'
print(f"PID Found: {get_pid_from_log('system boot sequence')}")          # ''
```

### Advanced Manipulation: split & Backreferences
- Complex Delimiters (re.split)
- Splitting data streams based on multiple network separators (e.g., semicolons, pipes, or tabs).
```python
import re

network_data = "192.168.1.1;AUTH_SUCCESS|DEVICE_ID:99;STATUS:OK"
# Split by either ; or |
tokens = re.split(r"[;|]", network_data)
print(tokens) 
# Output: ['192.168.1.1', 'AUTH_SUCCESS', 'DEVICE_ID:99', 'STATUS:OK']
```

### Backreferences for Data Reordering
- Using \1, \2 to rearrange captured groups in a single pass. Ideal for converting log formats (e.g., ID:Value to Value (ID)).
```python
import re

raw_config = "ID:conf_01, VAL:active"
# Rearranging groups using re.sub and backreferences
formatted = re.sub(r"ID:(\w+), VAL:(\w+)", r"Status: \2 (Key: \1)", raw_config)
print(formatted) 
# Output: Status: active (Key: conf_01)
```

---

# 🛡️ Systems Engineering: Regular Expressions (Regex) Reference (Part 2)

This section covers advanced implementations of the Python `re` module, focusing on precise data extraction, fault-tolerant parsing, stream manipulation, and complex pattern matching for enterprise-grade environments.

---

## 📐 1. Precise Quantitative Repetition (Numeric Bounds)

While `*`, `+`, and `?` provide broad matching, systems engineering often requires strict boundary checks (e.g., validating MAC addresses, hashes, or exact-length identifiers). We use numeric qualifiers `{}` for absolute precision.

| Syntax | Description | Use Case |
| :--- | :--- | :--- |
| `{n}` | Exactly *n* times. | `r"\b\w{5}\b"` (Matches exactly 5-character status codes). |
| `{n,m}` | Between *n* and *m* times. | `r"\d{3,5}"` (Matches valid Port numbers up to 5 digits). |
| `{n,}` | At least *n* times. | `r"\w{8,}"` (Enforcing minimum password or token length). |
| `{,m}` | Up to *m* times. | `r"ERR_{,10}"` (Matching error codes with a maximum prefix length). |

### Example: Strict Boundary Matching
```python
import re

# Finding exact 5-character operational states
print(re.findall(r"\b\w{5}\b", "State: ALIVE, Check: FATAL, Node: READY"))
# Output: ['ALIVE', 'FATAL', 'READY']

# Finding tokens of at least 16 characters
print(re.findall(r"\w{16,}", "TOKEN: abcdef1234567890xyz REJECTED"))
# Output: ['abcdef1234567890xyz']
```

## 🛡️ 2. Fault-Tolerant Data Extraction (Defensive Regex)
### When parsing high-volume logs, the structure is never guaranteed. Accessing a captured group without verifying the match object will result in an AttributeError and crash the pipeline.

- The Anti-Pattern (Unsafe Extraction)
```python
import re
# This will crash if the log line is malformed and does not contain a PID in brackets.
result = re.search(r"\[(\d+)\]", "kernel: memory fault at segment A")
# print(result[1]) -> TypeError/AttributeError: 'NoneType' object is not subscriptable
```

### The Engineering Standard (Safe Extraction)
- Always encapsulate Regex searches in functions that handle None returns gracefully.
```python
import re

def extract_pid(log_line: str) -> str:
    """Safely extracts a Process ID (PID) from standard syslog formats."""
    # Pattern: Escaped brackets \[ \] encapsulating a digit capture group (\d+)
    regex = r"\[(\d+)\]"
    result = re.search(regex, log_line)
    
    # Defensive check: if no match, return a safe fallback (empty string)
    if result is None:
        return ""
    
    return result[1] # Return the first captured group

# Production Tests
print(extract_pid("July 31 07:51:48 srv-01 bad_process[12345]: ERROR")) # Output: 12345
print(extract_pid("Warning: Unrecognized process spawned"))              # Output: (Empty String)
```

## 🔄 3. Advanced Stream Manipulation (`re.split` & `re.sub`)
### Multi-Delimiter Parsing (`re.split`)
- Standard `.split()` only accepts one delimiter. `re.split` allows you to break strings apart using complex patterns (e.g., splitting by semicolons, pipes, or dynamic whitespace).
```python
import re

raw_telemetry = "NODE_01|TEMP:45C;STATUS:OK|LOAD:80%"
# Split by either pipe (|) or semicolon (;)
data_points = re.split(r"[|;]", raw_telemetry)
print(data_points)
# Output: ['NODE_01', 'TEMP:45C', 'STATUS:OK', 'LOAD:80%']
```

### Data Sanitization & PII Masking (`re.sub`)
- Mandatory for GDPR/SOC2 compliance. `re.sub` identifies patterns and replaces them with redacted labels before logs hit the storage layer.
```python
import re

# Swap "Key, Value" to "Value [Key]" format
raw_config = "Database_Host, 192.168.1.50"
# \1 captures the key, \2 captures the value
formatted = re.sub(r"^([\w .-]*), ([\w .-]*)$", r"Target: \2 [Prop: \1]", raw_config)

print(formatted)
# Output: Target: 192.168.1.50 [Prop: Database_Host]
```
### Advanced Evaluation Logic
- For highly complex string analysis, Regex provides logical operators and assertions.
- Alternation (`|`) & Character Ranges (`[ ]`)
- Alternation: Functions as a logical `OR`.
- `r"region.*(us-east|eu-west|ap-south)`" matches specific cloud regions.
- Ranges: Match specific character sets without hardcoding every option.
- `r"[A-F0-9]`" matches valid hexadecimal characters (useful for MAC/IPv6).
- `r"[0-9\-,.]`" matches digits, dashes, commas, or dots.

## Lookaheads `(?=...)`
- Lookaheads allow you to match a pattern only if it is followed by another specific pattern, without including that second pattern in the final extraction. Highly useful in CI/CD log parsing.
```python
import re

ci_cd_output = "Build1-Passed, Build2-Passed, Build3-Failed, Build4-Passed"

# Match "Build\d" ONLY IF it is followed by "-Passed"
# The lookahead (?=-Passed) asserts the condition but isn't captured in the result.
successful_builds = re.findall(r"(Build\d)(?=-Passed)", ci_cd_output)

print(successful_builds)
# Output: ['Build1', 'Build2', 'Build4']
```
