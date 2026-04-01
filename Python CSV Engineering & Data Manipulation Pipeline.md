# Python CSV Engineering & Data Manipulation Pipeline

## 📖 Overview
In modern systems engineering, DevOps, and data administration, computers demand structure and precision. Configuration files, system logs, and hardware inventories are frequently stored as plain text. The ability to parse flat text into structured, actionable data is a critical engineering skill.

This repository/document serves as a comprehensive, production-ready guide to handling **CSV (Comma Separated Values)** files using Python's standard `csv` library. It transitions from basic sequential processing to robust, schema-agnostic dictionary mapping, adhering to enterprise best practices (PEP 8, context managers, type hinting, and error handling).

---

## 🏗️ Core Concepts & Architecture

Before diving into code, it is crucial to understand the underlying mechanics:
* **Parsing:** The act of analyzing flat text content to correctly structure the data.
* **CSV Format:** A data interchange format where each line represents a *data record* (row), and each record consists of one or more *fields* (columns) separated by commas.
* **Dialects:** CSVs are not universally identical. A dialect defines the parameters of the file structure:
    * `delimiter`: The character separating fields (e.g., `,`, `;`, `\t`).
    * `quotechar`: The character used to quote fields containing special characters (e.g., `"Engineering, R&D"`).
    * `strict`: When set to `True`, raises a `csv.Error` on bad CSV formatting (crucial for data validation pipelines).

---

## 🚀 Phase 1: Sequential Processing (List-Based I/O)

This approach is lightweight and suitable for simple datasets where the column structure is rigid and guaranteed never to change.

### Data Extraction (Reading)
We use `csv.reader()` to iterate over the file. To avoid "magic numbers" (like `row[0]`, `row[1]`), we use **List Unpacking** to assign meaningful variable names immediately.

```python
import csv
from pathlib import Path
import logging

logging.basicConfig(level=logging.INFO, format='%(levelname)s: %(message)s')

def process_employee_inventory(file_path: Path) -> None:
    """
    Parses employee data using standard CSV reader.
    Utilizes list unpacking for cleaner variable assignment.
    """
    try:
        # 'newline=""' is critical across platforms to prevent blank line issues
        with open(file_path, mode='r', encoding='utf-8', newline='') as f:
            reader = csv.reader(f)
            
            # Optional: Skip header row if it exists
            # next(reader, None) 
            
            for row in reader:
                # Unpacking guarantees strict length checking; 
                # throws ValueError if schema mismatches.
                name, phone, role = row
                logging.info(f"Parsed -> Name: {name} | Phone: {phone} | Role: {role}")
                
    except FileNotFoundError:
        logging.error(f"Target file not found: {file_path}")
    except ValueError as e:
        logging.error(f"Schema mismatch (unpacking error): {e}")

# Example usage:
# process_employee_inventory(Path('employees.csv'))
```

### Data Generation (Writing)
- Writing processed data (e.g., scraping network hosts) back to a structured file using `csv.writer()`. We use `writerows()` to execute a bulk write operation, minimizing disk I/O calls.

---

```python
import csv
from typing import List
from pathlib import Path

def export_network_hosts(file_path: Path, hosts_data: List[List[str]]) -> None:
    """
    Exports a matrix (list of lists) of network nodes to a CSV file.
    """
    with open(file_path, mode='w', encoding='utf-8', newline='') as f:
        writer = csv.writer(f)
        # Bulk write is significantly more efficient than looping writerow()
        writer.writerows(hosts_data)
        
# Example usage:
# network_nodes = [["workstation.local", "192.168.25.46"], ["webserver.cloud", "10.2.5.6"]]
# export_network_hosts(Path('hosts.csv'), network_nodes)
```
---

## 🏛️ Phase 2: Enterprise Processing (Dictionary-Based I/O)
- Why use Dictionaries? In production environments, datasets often contain dozens of columns. Relying on list indices (`row[14]`) is unmaintainable. Furthermore, column orders may change in upstream exports.
- `csv.DictReader` and `csv.DictWriter` map data to Dictionary Keys (Header names). This decouples your code from the physical column order, making the pipeline highly resilient to schema evolution.
### Resilient Data Parsing (`DictReader`)

```python
import csv
from pathlib import Path

def analyze_software_versions(file_path: Path) -> None:
    """
    Parses software deployment logs using DictReader.
    Resilient to column reordering.
    """
    with open(file_path, mode='r', encoding='utf-8', newline='') as f:
        reader = csv.DictReader(f)
        
        for row in reader:
            # Accessing by Key instead of Index
            app_name = row.get("name", "Unknown")
            users = row.get("users", "0")
            print(f"[STATUS] {app_name} is currently supporting {users} active users.")
```

### Resilient Data Export (`DictWriter`)
- When writing dictionaries to a CSV, you must define the `fieldnames` (the schema). You must also explicitly call `writeheader()` to ensure the resulting file has column names, which is vital for downstream consumers (like Excel or BI tools).

```python
import csv
from typing import List, Dict
from pathlib import Path

def export_department_roster(file_path: Path, users: List[Dict[str, str]]) -> None:
    """
    Safely exports a list of dictionaries to a structured CSV file.
    """
    # Define the explicit schema
    schema_keys = ["name", "username", "department"]
    
    with open(file_path, mode='w', encoding='utf-8', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=schema_keys)
        
        # Step 1: Write the header row
        writer.writeheader()
        
        # Step 2: Write the payload
        writer.writerows(users)

# Example usage:
# roster = [{"name": "Sol Mansi", "username": "solm", "department": "IT infrastructure"}]
# export_department_roster(Path('by_department.csv'), roster)
```
---

## 🛠️ Phase 3: The Complete Data Pipeline (Refactored Challenge)
- This is a production-grade implementation combining file generation, data manipulation, and data presentation. It showcases separation of concerns, type hinting, and robust dictionary manipulation.

```python
import csv
from pathlib import Path
from typing import List, Dict

class BotanicalDataPipeline:
    """
    A class to handle the end-to-end generation and parsing of botanical datasets.
    Demonstrates professional separation of concerns and robust I/O handling.
    """
    
    def __init__(self, target_file: str):
        self.filepath = Path(target_file)
        self.schema = ["name", "color", "type"]

    def generate_seed_data(self) -> None:
        """Idempotent method to generate the baseline CSV dataset."""
        seed_data = [
            {"name": "carnation", "color": "pink", "type": "annual"},
            {"name": "daffodil", "color": "yellow", "type": "perennial"},
            {"name": "iris", "color": "blue", "type": "perennial"},
            {"name": "poinsettia", "color": "red", "type": "perennial"},
            {"name": "sunflower", "color": "yellow", "type": "annual"},
        ]
        
        with open(self.filepath, mode='w', encoding='utf-8', newline='') as file:
            writer = csv.DictWriter(file, fieldnames=self.schema)
            writer.writeheader()
            writer.writerows(seed_data)
            
    def process_and_report(self) -> str:
        """
        Reads the structured data and transforms it into a human-readable report.
        """
        if not self.filepath.exists():
            return f"Error: {self.filepath} does not exist. Run generate_seed_data() first."
            
        report_lines: List[str] = []
        
        with open(self.filepath, mode='r', encoding='utf-8', newline='') as file:
            reader = csv.DictReader(file)
            
            for row in reader:
                # String formatting using f-strings (Modern Python standard)
                formatted_entry = f"A {row['color']} {row['name']} is a {row['type']} plant."
                report_lines.append(formatted_entry)
                
        return "\n".join(report_lines)

# ==========================================
# Execution / Entry Point
# ==========================================
if __name__ == "__main__":
    pipeline = BotanicalDataPipeline("flowers_db.csv")
    
    # 1. Generate the CSV artifact
    pipeline.generate_seed_data()
    
    # 2. Parse and output the results
    result_report = pipeline.process_and_report()
    
    print("--- Botanical System Report ---")
    print(result_report)
    print("-------------------------------")
```

## 🛡️ Best Practices Checklist for Engineers

### Whenever reviewing or writing CSV manipulation code, ensure the following standards are met:

- [x] Context Managers: Always use `with open(...)` to prevent file descriptor leaks.
- [x] Newline Parameter: Always include `newline='' in open()` for CSV files to prevent cross-platform CR/LF (carriage return) bugs.
- [x] Encoding: Explicitly define `encoding='utf-8'` to prevent fatal errors when reading external data containing special characters.
- [x] Prefer DictReader/DictWriter: Unless the file lacks a header entirely, default to Dictionary-based processing for code resilience.
- [x] Dialect Awareness: If data looks malformed, check the `delimiter` (comma vs. semicolon) and adjust via the `dialect` parameters.
