# 📦 Python Tuples: Immutable Data Structures & Optimization

This documentation covers the architectural characteristics of the `tuple` data structure in Python, its performance advantages over lists, and its critical role in advanced dictionary sorting algorithms.

## 🧠 1. Architectural Overview: Tuples vs. Lists

Tuples are functionally similar to lists but are defined by one core architectural constraint: **Immutability.**

- **Syntax:** Uses parentheses `()` instead of square brackets `[]`.
- **Memory Efficiency:** Since tuples are immutable, Python does not allocate extra "buffer" memory for dynamic resizing. This makes them more memory-dense and faster to process than lists.
- **Data Integrity:** Tuples are used to protect data from accidental modification throughout the program lifecycle.
- **Constraints:** Methods that modify the structure (e.g., `.append()`, `.extend()`, `.remove()`, `.pop()`, `.reverse()`) are unavailable. Only `.count()` and `.index()` are supported.



```python
# List (Mutable)
x = [9, 8, 7]
x[2] = 6  # Success

# Tuple (Immutable)
z = (9, 8, 7)
z[2] = 0  # TypeError: 'tuple' object does not support item assignment
```

## ⚙️ 2. Core Mechanics
### Tuple Assignment Pattern
- Tuples allow for simultaneous assignment by placing a tuple of variables on the left side of an assignment statement.
```python
(x, y) = (4, 'fred')
# Result: x = 4, y = 'fred'
```

### Comparison Logic (Significant Element Rule)
- Python compares tuples by evaluating elements from left to right. Once a difference is found that satisfies the condition, the evaluation stops immediately (ignoring all subsequent elements).
- `(0, 1, 2) < (5, 1, 2)` ➔ True (Evaluation stops at 0 < 5).
- `(1, 2, 100) < (1, 3, 0)`➔ True (1 == 1, evaluation moves to 2 < 3, result is True).

## 📊 3. Sorting Dictionaries via Tuples
- Dictionaries are unordered. To sort them, we leverage the fact that `dict.items()` returns a list of (`Key, Value`) tuples.

### Scenario A: Sorting by Key
- This is the default behavior when passing `dict.items()` into the `sorted()` function.

```python
d = {'c': 22, 'a': 10, 'b': 1}
for k, v in sorted(d.items()):
    print(k, v)
# Output: a 10, b 1, c 22
```

### Scenario B: Sorting by Value (The Flip Pattern)
- To sort by frequency (values), we must construct a temporary list of tuples where the Value is the first element.
```python
d = {'a': 10, 'b': 1, 'c': 22}
tmp = list()

# Step 1: Extract and flip to (Value, Key)
for k, v in d.items():
    tmp.append((v, k))

# Step 2: Sort descending based on the first element (Value)
tmp = sorted(tmp, reverse=True)

print(tmp)
# Output: [(22, 'c'), (10, 'a'), (1, 'b')]
```

## 🚀 4. Production Application: Top 10 Most Common Words
- Combining Histograms (Dictionaries) and Sorting (Tuples) to extract the 10 most frequent words from a text file:
```python
fhand = open('data.txt')
counts = dict()

# 1. Generate Histogram
for line in fhand:
    words = line.split()
    for word in words:
        counts[word] = counts.get(word, 0) + 1

# 2. Convert to (Value, Key) List
lst = list()
for key, val in counts.items():
    lst.append((val, key))

# 3. Sort Descending
lst = sorted(lst, reverse=True)

# 4. Extract Top 10 via List Slicing
for val, key in lst[:10]:
    print(key, val)
```

## 💎 Pro-Tip: List Comprehension
- Experienced engineers often replace the explicit loop with a List Comprehension for more succinct code:
```python
# Create (v, k) tuples for all items and sort them in one line
lst = sorted([(v, k) for k, v in counts.items()], reverse=True)
```

---

# Mbox Email Time Distribution Analyzer

## 📌 Overview
This lightweight Python utility parses standard `mbox` email log files to analyze and extract the hourly distribution of messages. It scans through the email routing data, isolates the commit time for each message, and generates a chronologically sorted histogram displaying the volume of emails processed per hour.

## ⚙️ Core Concepts & Data Structures
This script demonstrates the efficient use of foundational Python data structures to handle data parsing and aggregation:
* **Strings:** Sequential splitting for data extraction.
* **Dictionaries (`dict`):** Key-value mapping for $O(1)$ lookup time and frequency counting (Histogram pattern).
* **Tuples (`tuple`):** Immutable pairs used for secure state transfer and sorting.

## 🚀 Usage

### Prerequisites
* Python 3.x installed on your local machine.
* A valid `mbox` formatted text file (e.g., `mbox-short.txt`) in the same directory as the script.

### Execution
Run the script via your terminal:
```bash
python email_analyzer.py
```

- When prompted, enter the name of the file you wish to analyze. If you press Enter without typing a name, the script defaults to mbox-short.txt for rapid testing.

## 🧠 Architectural Logic & Flow
- The script is built on a straightforward, four-step pipeline:

### 1. Fault-Tolerant File I/O
- The program begins by attempting to open the user-defined file. It implements a try/except block to handle FileNotFoundError exceptions gracefully, preventing ugly tracebacks and providing a clean exit strategy if the file does not exist.

### 2. Data Filtering & Tokenization
- The script reads the file line by line (memory efficient for large files).
- Target Identification: It uses `.startswith('From ')` to strictly filter lines indicating a sender. It intentionally ignores From: lines to avoid processing email bodies or metadata.
- Tokenization: The target line is split into a list of words using `.split()`.
- Sample Line: `From stephen.marquard@uct.ac.za Sat Jan  5 09:14:16 2008`
- The time string (`09:14:16`) is consistently located at index [5].

### 3. Double Splitting & Aggregation
- Once the time string is isolated, a secondary `.split(':')` is applied to separate hours, minutes, and seconds.
- The hour (index `[0]`) is extracted.
- Histogram Logic: The script uses the dictionary `.get(key, default)` method to increment the count for that specific hour. If the hour doesn't exist in the dictionary yet, it initializes it to `0` and adds `1`.

### 4. Chronological Sorting
- Dictionaries are inherently unordered. To present the data chronologically:
- `.items()` extracts the data into a list of `(hour, count)` tuples.
- `sorted()` naturally orders these tuples based on their first element (the hour), ensuring a clean, ascending output from `00` to `23`.

## 📊 Example Output 
Enter file: mbox-short.txt
```bash
04 3
06 1
07 1
09 2
10 3
11 6
14 1
15 2
16 4
17 2
18 1
19 1
```

## ⏱️ Performance Considerations
- Time Complexity: $O(N \log N)$ in the worst case due to the final sorting operation, where $N$ is the number of unique hours (maximum 24). The file reading and counting process operates in $O(L)$ time, where $L$ is the number of lines in the file.
- Space Complexity: $O(U)$ where $U$ is the number of unique hours (max 24), making it highly memory efficient as the entire file is never loaded into RAM at once.
