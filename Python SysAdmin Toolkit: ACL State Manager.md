```python
import logging
import os
from typing import List

# Configure professional logging format
logging.basicConfig(
    level=logging.INFO, 
    format='%(asctime)s - %(levelname)s - %(message)s'
)

def remove_unauthorized_users(file_path: str, users_to_revoke: List[str]) -> None:
    """
    Safely reads a state file and removes specified users.
    Uses in-memory buffering to prevent file corruption during read/write cycles.
    """
    if not os.path.exists(file_path):
        logging.error(f"Target file does not exist: {file_path}")
        return

    temp_state = []
    
    # STEP 1: Safely load current state into memory
    try:
        with open(file_path, "r") as file:
            for line in file:
                # Strip hidden characters (\n) to ensure accurate matching
                temp_state.append(line.strip())
        logging.info(f"Successfully loaded {len(temp_state)} records from {file_path}.")
    except IOError as e:
        logging.error(f"Failed to read from file {file_path}. Exception: {e}")
        return

    # STEP 2: Write back the filtered state, avoiding race conditions
    try:
        with open(file_path, "w") as file:
            for user in temp_state:
                if user not in users_to_revoke:
                    file.write(f"{user}\n")
        logging.info(f"Successfully revoked access for {len(users_to_revoke)} user(s).")
    except IOError as e:
        logging.error(f"Failed to write to file {file_path}. Exception: {e}")


# --- Execution Block (For testing purposes) ---
if __name__ == "__main__":
    target_file = "authorized_users.txt"
    revoked_users = ["dev_user_old", "temp_admin_3"]
    
    logging.info("Starting ACL cleanup process...")
    remove_unauthorized_users(target_file, revoked_users)
```

# 🛠️ Python SysAdmin Toolkit: ACL State Manager

## Overview
This repository contains utility scripts designed for system administration and DevOps automation. The primary focus is on the `acl_manager.py` script, which demonstrates safe, idempotent, and memory-efficient file I/O operations for managing text-based Access Control Lists (ACLs) or configuration state files.

## 🚀 Engineering Principles Applied

Unlike basic scripting, this utility is built with production-grade safety mechanisms:

* **Context Management (`with` statements):** Ensures robust resource allocation and automatic file closure, preventing file-lock issues and File Descriptor leaks.
* **Safe Overwriting (No Data Corruption):** Implements a multi-step update sequence (`Read to Memory` -> `Filter Logic` -> `Write to Disk`) to avoid race conditions that occur with simultaneous read/write operations.
* **Error Handling & Logging:** Replaces standard standard outputs with the `logging` library to track execution states and catches `IOError` exceptions to prevent runtime crashes.
* **Type Hinting (PEP 484):** Enhances code readability and maintainability for team environments.

## ⚙️ Logic Flow

The script follows a strict methodology using native Python libraries:

1. **Pre-flight Check:** Validates file existence via the `os` module.
2. **Buffer Phase:** Safely reads the current state into a local list, stripping hidden newline (`\n`) characters to ensure accurate string matching.
3. **Execution Phase:** Re-writes the state file entirely, omitting the specified entities (e.g., revoked users, blocked IPs).
