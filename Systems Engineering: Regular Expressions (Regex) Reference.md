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
