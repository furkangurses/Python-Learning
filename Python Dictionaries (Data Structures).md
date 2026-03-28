# Python Dictionaries (Data Structures)

## 📌 Overview
Dictionaries (`dict`) are Python's most powerful data collection. Unlike lists, which use positional indexing (0, 1, 2...), dictionaries use a **Key-Value** mapping system. This structure is known in computer science as a Hash Map or Associative Array, providing highly efficient data retrieval ($O(1)$ complexity) regardless of the dataset size.

## 🏗️ 1. Lists vs. Dictionaries

| Feature | `list` (Lists) | `dict` (Dictionaries) |
| :--- | :--- | :--- |
| **Indexing** | Positional (Integers starting from 0) | Keys (Typically Strings, but can be any immutable type) |
| **Order** | Strict sequence | Insertion order (Python 3.7+) |
| **Analogy** | Library Card Catalog | Labeled File Cabinet |
| **Syntax** | `[]` Brackets | `{}` Curly Braces |

---

## 💻 2. Core Operations & Syntax

Dictionaries function as a database of variables stored within a single collection.

### Initialization & Assignment
```python
# Creating an empty dictionary
cabinet = dict() # or {}

# Assigning values to keys
cabinet['summer'] = 12
cabinet['fall'] = 3
cabinet['spring'] = 75

# Updating a value
cabinet['fall'] = cabinet['fall'] + 2 # fall becomes 5
```

### Dictionary Literals
- You can define a dictionary with initial key-value pairs using curly braces:
```python
users = {'chuck': 1, 'fred': 42, 'jan': 100}
```

## 📊 3. The Histogram Pattern (Frequency Counting)
- The most common application of a dictionary is frequency analysis (counting how many times an item appears in a collection).

### Handling Missing Keys (`Traceback` Prevention)
- Referencing a key that does not exist throws a `KeyError`. We solve this using the `.get(key, default)` method.

- The .get() Idiom:
```python
counts = dict()
names = ['csev', 'cwen', 'csev', 'zqian', 'cwen']

for name in names:
    # If the key doesn't exist, return 0. Then add 1.
    counts[name] = counts.get(name, 0) + 1

print(counts) 
# Output: {'csev': 2, 'cwen': 2, 'zqian': 1}
```

### Note: This one-line idiom effectively replaces a 4-line `if-else` block.

## 🔄 4. Dictionary Iteration
### By default, iterating through a dictionary yields its keys. To access different parts of the dictionary, Python provides specific methods:
- `dict.keys()`: Returns a list of keys.
- `dict.values()`: Returns a list of values.
- `dict.items()`: Returns a list of key-value tuples.

### Tuple Assignment (The Pythonic Way)
### Python allows you to iterate through both keys and values simultaneously using two iteration variables. This is a highly elegant and expressive syntax unique to Python.
```python
jjj = {'chuck': 1, 'fred': 42, 'jan': 100}

for key, value in jjj.items():
    print(f"Key: {key}, Value: {value}")
```

## 🚀 5. Capstone: The Word Counting Script
- This script reads a text file, splits the text into individual words, builds a frequency histogram using a dictionary, and then performs a maximum loop to find the most frequently used word.
```python
# 1. Setup and Input
name = input('Enter file name: ')
handle = open(name)
counts = dict()

# 2. Parse and Count (Histogram)
for line in handle:
    words = line.split()
    for word in words:
        counts[word] = counts.get(word, 0) + 1

# 3. Find the Maximum Count
big_word = None
big_count = None

for word, count in counts.items():
    if big_count is None or count > big_count:
        big_word = word
        big_count = count

# 4. Output Result
print(f"Most frequent word: '{big_word}' (Appeared {big_count} times)")
```
