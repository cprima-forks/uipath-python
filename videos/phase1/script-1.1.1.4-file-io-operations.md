# Video Script: File I/O Operations

**Module**: 1.1.1.4
**Duration**: 28 minutes
**Target Audience**: Beginners learning Python for UiPath Agent development
**Prerequisites**: Module 1.1.1.3 (Error Handling in Python)

---

## Section 1: Introduction [00:00 - 02:00]

### Presenter Notes
- Emphasize that file operations are essential for agents
- Connect to real-world scenarios
- Set expectations for comprehensive coverage
- Establish practical focus

### On-Screen Content
**TITLE SLIDE**
- "File I/O Operations"
- "Module 1.1.1.4 - Working with Files in Python"

### Script

**[00:00]** PRESENTER:
"Welcome back! In our previous modules, we learned about variables, functions, and error handling. Today, we're going to learn something that every UiPath Agent uses constantly: file input/output operations - reading and writing files.

Think about what agents do: they read configuration files, process documents like invoices and reports, write logs, import data from CSVs, export results to JSON. All of this requires file I/O.

**[00:45]** By the end of this video, you'll be able to:
- Read and write text files safely using context managers
- Work with CSV and JSON files for data exchange
- Handle file paths in a cross-platform way
- Apply best practices for robust file operations

**[01:15]** This is practical, hands-on knowledge. Every agent you build will use these techniques. File I/O might seem simple at first, but there are important details - like automatically closing files and handling errors - that separate hobbyist code from production code.

**[01:45]** Let's dive in and make you proficient with Python file operations!"

---

## Section 2: Opening and Closing Files [02:00 - 05:30]

### Presenter Notes
- Start with the problem (not closing files)
- Introduce context managers as the solution
- Emphasize "with" as the standard pattern
- Show file modes

### On-Screen Content
**CODE EXAMPLE 1: The Wrong Way**
```python
# ❌ BAD: Manual closing
file = open('data.txt', 'r')
content = file.read()
file.close()  # What if error occurs before this?
```

**CODE EXAMPLE 2: The Right Way**
```python
# ✅ GOOD: Context manager
with open('data.txt', 'r') as file:
    content = file.read()
# File automatically closed here
```

**CODE EXAMPLE 3: File Modes**
```python
'r'   # Read (default)
'w'   # Write (overwrites!)
'a'   # Append
'r+'  # Read and write
'rb'  # Read binary
'wb'  # Write binary
```

### Script

**[02:00]** PRESENTER:
"Let's start with the basics: opening and closing files.

**[DEMONSTRATE CODE EXAMPLE 1]**

**[02:15]** This code opens a file, reads it, and closes it. Simple, right? But there's a problem. What if an error occurs between open and close? Maybe the file is empty and read() fails. The file never gets closed. You've leaked a file handle.

In a long-running agent processing thousands of files, this adds up. Eventually, you hit the operating system's limit on open files, and your agent crashes.

**[03:00]** The solution is context managers - the 'with' statement:

**[DEMONSTRATE CODE EXAMPLE 2]**

**[03:15]** The 'with' statement creates a context. When the block ends - whether normally or because of an exception - Python automatically closes the file. Guaranteed. No leaks.

This is the Pythonic way. Professional Python developers always use 'with' for files. Make it your habit too.

**[03:45]** The second argument to open() is the mode:

**[DEMONSTRATE CODE EXAMPLE 3]**

**[04:00]** Mode 'r' opens for reading - this is the default. The file must exist or you get FileNotFoundError.

Mode 'w' opens for writing. Critical detail: if the file exists, 'w' erases it! All content lost. Use carefully.

Mode 'a' opens for appending - adds to the end without erasing. Perfect for log files.

Mode 'r+' allows both reading and writing. Rarely needed.

The 'b' suffix means binary mode - for images, videos, PDFs. We'll cover this later.

**[04:45]** For agent development, you'll primarily use 'r' for reading config files and data, 'w' for writing results, and 'a' for logging. Remember that 'w' is destructive - it erases the file!"

---

## Section 3: Reading Text Files [05:30 - 08:30]

### Presenter Notes
- Show three reading methods
- Explain file pointer behavior
- Demonstrate memory-efficient line iteration
- Connect to agent use cases

### On-Screen Content
**CODE EXAMPLE 4: Reading Methods**
```python
# Method 1: Read entire file
with open('document.txt', 'r') as f:
    content = f.read()  # Returns one string

# Method 2: Read as list of lines
with open('document.txt', 'r') as f:
    lines = f.readlines()  # ['line1\n', 'line2\n']

# Method 3: Iterate line by line (best for large files)
with open('document.txt', 'r') as f:
    for line in f:
        print(line.strip())  # Process one line at a time
```

**CODE EXAMPLE 5: File Pointer Gotcha**
```python
with open('data.txt', 'r') as f:
    content = f.read()  # File pointer at end
    lines = f.readlines()  # Returns [] (nothing left)
    print(len(lines))  # 0
```

### Script

**[05:30]** PRESENTER:
"There are three main ways to read a file:

**[DEMONSTRATE CODE EXAMPLE 4]**

**[05:45]** First, read() reads the entire file as one string. Simple and convenient for small files. But if the file is 1GB, you just loaded 1GB into memory. Not great.

Second, readlines() reads the entire file but returns a list of lines. Each line is a string. Still loads the whole file into memory, but organized by line.

Third - and this is the Pythonic way - iterate over the file object directly. This reads one line at a time. Memory efficient. If you're processing a 1GB log file, you never load more than one line into memory.

**[06:30]** For agent development, if you're reading configuration files - probably small, a few KB - use read() or readlines(). If you're processing large documents or logs, iterate line by line.

**[06:50]** Here's an important gotcha about the file pointer:

**[DEMONSTRATE CODE EXAMPLE 5]**

**[07:00]** After read(), the file pointer is at the end. There's nothing left to read. Calling readlines() returns an empty list. This surprises beginners.

If you need to read twice, either call read() once and work with that string, or use f.seek(0) to reset the pointer to the beginning.

**[07:30]** Here's a practical example:

```python
def count_errors_in_log(log_file):
    error_count = 0
    with open(log_file, 'r') as f:
        for line in f:
            if 'ERROR' in line:
                error_count += 1
    return error_count
```

Memory efficient even for huge logs. Processes one line at a time."

---

## Section 4: Writing Text Files [08:30 - 11:00]

### Presenter Notes
- Show writing methods
- Warn about mode 'w' overwriting
- Demonstrate appending for logs
- Show practical agent logging example

### On-Screen Content
**CODE EXAMPLE 6: Writing Files**
```python
# Write (overwrites entire file!)
with open('output.txt', 'w') as f:
    f.write('Hello, World!\n')
    f.write('Second line\n')

# Append (adds to end)
with open('log.txt', 'a') as f:
    f.write('New log entry\n')

# Write list of lines
lines = ['Line 1\n', 'Line 2\n']
with open('output.txt', 'w') as f:
    f.writelines(lines)
```

**CODE EXAMPLE 7: Agent Logging**
```python
from datetime import datetime

def log_agent_activity(message, level='INFO'):
    timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    log_entry = f"[{timestamp}] [{level}] {message}\n"

    with open('agent.log', 'a') as f:
        f.write(log_entry)

log_agent_activity('Agent started')
log_agent_activity('Processing document.pdf')
log_agent_activity('File not found', level='ERROR')
```

### Script

**[08:30]** PRESENTER:
"Writing files is straightforward, but there's one critical thing to remember:

**[DEMONSTRATE CODE EXAMPLE 6]**

**[08:45]** Mode 'w' erases the file first. If output.txt had 1000 lines, they're gone. This is by design - 'w' means 'write from scratch'. But it catches beginners off guard.

If you want to add to the end of a file, use mode 'a' for append. Perfect for log files where you're constantly adding new entries.

**[09:15]** The write() method writes a string. It doesn't add a newline automatically, so add '\n' yourself if you want line breaks.

The writelines() method writes a list of strings. Also doesn't add newlines - each string in the list should end with '\n' if you want them on separate lines.

**[09:45]** Here's a realistic example - agent logging:

**[DEMONSTRATE CODE EXAMPLE 7]**

**[10:00]** This function appends timestamped log entries to a file. Each time an agent does something - starts up, processes a file, encounters an error - we log it.

Using mode 'a' means we never lose previous logs. They accumulate over time. In production, you'd also add log rotation to prevent files from growing forever, but this is the basic pattern.

**[10:30]** Notice we add the newline '\n' to each entry. Without it, all log entries would run together on one line.

This is how professional agents track their activity. Logs are invaluable when debugging issues in production."

---

## Section 5: CSV Files [11:00 - 14:30]

### Presenter Notes
- Introduce CSV module
- Show reader and DictReader
- Demonstrate writer and DictWriter
- Practical example processing tabular data

### On-Screen Content
**CODE EXAMPLE 8: Reading CSV**
```python
import csv

# Reading as lists
with open('data.csv', 'r') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)  # ['col1', 'col2', 'col3']

# Reading as dictionaries (better!)
with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['name'], row['age'])
```

**CODE EXAMPLE 9: Writing CSV**
```python
import csv

# Writing dictionaries (cleanest)
data = [
    {'name': 'Alice', 'age': 30, 'city': 'NYC'},
    {'name': 'Bob', 'age': 25, 'city': 'LA'}
]

with open('output.csv', 'w', newline='') as f:
    fieldnames = ['name', 'age', 'city']
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()  # Writes column headers
    writer.writerows(data)
```

### Script

**[11:00]** PRESENTER:
"CSV - Comma-Separated Values - is one of the most common data formats. Spreadsheets, databases, APIs - they all export to CSV. Agents process CSV files constantly.

Python's csv module makes this easy:

**[DEMONSTRATE CODE EXAMPLE 8]**

**[11:20]** csv.reader() parses CSV and yields each row as a list of strings. Simple, but you access columns by index: row[0], row[1]. Easy to make mistakes.

csv.DictReader() is better. It uses the first row as column headers and yields each row as a dictionary. Now you access columns by name: row['name'], row['age']. Much more readable and maintainable.

**[12:00]** For agent development, I recommend DictReader() almost always. Your code is more self-documenting and less error-prone.

**[12:15]** Writing CSV:

**[DEMONSTRATE CODE EXAMPLE 9]**

**[12:25]** DictWriter() is the counterpart to DictReader(). You specify the field names (column headers), write the header row, then write your data.

Notice 'newline=""' in the open() call. This is important on Windows to prevent extra blank lines. It's a CSV module quirk - just include it.

**[13:00]** Here's a practical example:

```python
def process_sales_report(input_csv):
    high_value = []
    with open(input_csv, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            if float(row['amount']) > 1000:
                high_value.append(row)

    with open('high_value_sales.csv', 'w', newline='') as f:
        if high_value:
            writer = csv.DictWriter(f, fieldnames=high_value[0].keys())
            writer.writeheader()
            writer.writerows(high_value)

    return len(high_value)
```

**[13:30]** This agent reads a sales report CSV, filters for high-value sales over $1000, and writes them to a new CSV. This pattern - read CSV, process, write CSV - is extremely common in business automation."

---

## Section 6: JSON Files [14:30 - 17:30]

### Presenter Notes
- Explain JSON's ubiquity
- Clarify dumps vs dump, loads vs load
- Show configuration management pattern
- Connect to agent settings

### On-Screen Content
**CODE EXAMPLE 10: JSON Operations**
```python
import json

# Reading JSON
with open('config.json', 'r') as f:
    config = json.load(f)  # Returns dict/list
print(config['setting'])

# Writing JSON
data = {
    'agent_name': 'DocumentProcessor',
    'version': '1.0',
    'settings': {
        'timeout': 30,
        'retries': 3
    }
}

with open('config.json', 'w') as f:
    json.dump(data, f, indent=2)  # indent for readability
```

**CODE EXAMPLE 11: dumps vs dump**
```python
data = {'name': 'Alice', 'age': 30}

# dumps() - returns STRING
json_string = json.dumps(data)
print(type(json_string))  # <class 'str'>

# dump() - writes to FILE
with open('data.json', 'w') as f:
    json.dump(data, f)

# 's' suffix = string operation
# no 's' = file operation
```

### Script

**[14:30]** PRESENTER:
"JSON - JavaScript Object Notation - is the universal data exchange format. APIs use JSON. Configuration files use JSON. Modern applications use JSON. It's essential.

**[DEMONSTRATE CODE EXAMPLE 10]**

**[14:50]** json.load() reads a JSON file and returns a Python object - usually a dictionary or list. JSON objects become Python dicts, JSON arrays become Python lists.

json.dump() writes a Python object to a file as JSON. The indent parameter makes the JSON human-readable with nice formatting. Without it, everything is on one line.

**[15:20]** For agent configuration, JSON is perfect. It's human-readable, supports nesting, and maps naturally to Python dictionaries.

**[15:35]** Here's a tricky point that confuses beginners:

**[DEMONSTRATE CODE EXAMPLE 11]**

**[15:45]** There are four JSON functions:
- json.dump() - writes to file
- json.dumps() - returns string ('dump string')
- json.load() - reads from file
- json.loads() - parses string ('load string')

The ones with 's' work with strings. The ones without work with files. Easy to remember: 's' for string.

**[16:15]** Here's a practical configuration manager:

```python
import json
from pathlib import Path

class AgentConfig:
    def __init__(self, config_path='config.json'):
        self.config_path = Path(config_path)
        self.config = self.load_config()

    def load_config(self):
        if self.config_path.exists():
            with open(self.config_path, 'r') as f:
                return json.load(f)
        return {'timeout': 30, 'max_retries': 3}

    def save_config(self):
        with open(self.config_path, 'w') as f:
            json.dump(self.config, f, indent=2)

# Usage
config = AgentConfig()
config.config['timeout'] = 60
config.save_config()
```

**[17:00]** This pattern - load config from JSON, modify it in Python, save it back - is how agents manage their settings. JSON makes it easy to provide a config file that users can edit."

---

## Section 7: Cross-Platform Paths [17:30 - 20:30]

### Presenter Notes
- Explain the Windows/Linux path separator problem
- Show os.path.join() as the old way
- Introduce pathlib as the modern way
- Demonstrate pathlib operations

### On-Screen Content
**CODE EXAMPLE 12: Path Handling**
```python
# ❌ BAD: Hardcoded separator
path = 'data\\files\\document.txt'  # Windows only!

# ✅ BETTER: os.path.join()
import os
path = os.path.join('data', 'files', 'document.txt')

# ✅ BEST: pathlib (modern)
from pathlib import Path
path = Path('data') / 'files' / 'document.txt'
```

**CODE EXAMPLE 13: pathlib Operations**
```python
from pathlib import Path

path = Path('data') / 'files' / 'document.txt'

# Check existence
if path.exists():
    print("File exists")

# Get parts
path.name      # 'document.txt'
path.stem      # 'document'
path.suffix    # '.txt'
path.parent    # Path('data/files')

# List directory
for file in Path('data').iterdir():
    print(file)

# Find files by pattern
for txt_file in Path('data').glob('*.txt'):
    print(txt_file)
```

### Script

**[17:30]** PRESENTER:
"Here's a problem: Windows uses backslashes for paths (C:\Users\data), but Linux and Mac use forward slashes (/home/user/data). If you hardcode the separator, your agent breaks on other platforms.

**[DEMONSTRATE CODE EXAMPLE 12]**

**[17:50]** Never hardcode path separators. Three solutions:

Old way: os.path.join(). It chooses the right separator for the platform. Works, but verbose.

Modern way: pathlib. This is the Pythonic way as of Python 3.4+. Object-oriented, intuitive, and powerful.

**[18:15]** With pathlib, you use the division operator to build paths: Path('data') / 'files' / 'document.txt'. Python handles the platform details.

**[18:30]** pathlib is more than just path construction:

**[DEMONSTRATE CODE EXAMPLE 13]**

**[18:45]** Check if paths exist, distinguish files from directories, get file name parts, list directory contents, find files by pattern - all with clean, readable syntax.

**[19:15]** Here's why pathlib is great:

```python
# Old way (os.path)
import os
dirname = os.path.dirname(os.path.abspath(__file__))
parent = os.path.dirname(dirname)
config_path = os.path.join(parent, 'config', 'settings.json')

# New way (pathlib)
from pathlib import Path
config_path = Path(__file__).parent.parent / 'config' / 'settings.json'
```

Much clearer! pathlib is object-oriented. Paths are objects with methods and properties.

**[20:00]** For agent development, use pathlib. It's modern, cross-platform, and makes your code more readable. All new Python code should use pathlib for paths."

---

## Section 8: Error Handling and Best Practices [20:30 - 24:00]

### Presenter Notes
- Review common file errors
- Show EAFP pattern
- Demonstrate binary mode
- Present comprehensive best practices

### On-Screen Content
**CODE EXAMPLE 14: Error Handling**
```python
def safe_read_file(file_path):
    try:
        with open(file_path, 'r') as f:
            return f.read()
    except FileNotFoundError:
        print(f"File not found: {file_path}")
        return None
    except PermissionError:
        print(f"Permission denied: {file_path}")
        return None
    except Exception as e:
        print(f"Error: {e}")
        return None
```

**CODE EXAMPLE 15: EAFP vs LBYL**
```python
# LBYL: Look Before You Leap
if os.path.exists(file_path):
    with open(file_path) as f:
        data = f.read()
# ⚠️ Race condition!

# EAFP: Easier to Ask Forgiveness (Pythonic)
try:
    with open(file_path) as f:
        data = f.read()
except FileNotFoundError:
    data = None
# ✅ No race condition
```

### Script

**[20:30]** PRESENTER:
"File operations fail. Files don't exist, permissions are wrong, disks are full. Professional code handles these errors gracefully.

**[DEMONSTRATE CODE EXAMPLE 14]**

**[20:45]** Common file exceptions:
- FileNotFoundError: File doesn't exist
- PermissionError: No permission to read/write
- IsADirectoryError: Path is a directory, not a file
- UnicodeDecodeError: Encoding problems

Catch these specifically and handle appropriately. Maybe missing files get defaults, permission errors get logged, etc.

**[21:20]** Python philosophy: EAFP - Easier to Ask Forgiveness than Permission.

**[DEMONSTRATE CODE EXAMPLE 15]**

**[21:30]** Don't check if a file exists before opening it. Just try to open it and handle the error. Why?

First, it's a race condition. The file could be deleted between your check and your open.

Second, there are many failure modes: file doesn't exist, no permission, it's actually a directory. Try-except handles all of them.

**[22:00]** EAFP is more Pythonic and more robust.

**[22:10]** Binary mode for non-text files:

```python
# Reading an image
with open('photo.jpg', 'rb') as f:
    data = f.read()  # bytes object

# Writing binary data
with open('copy.jpg', 'wb') as f:
    f.write(data)
```

Text mode ('r', 'w') does encoding/decoding - converts between bytes and strings. Binary mode ('rb', 'wb') gives you raw bytes.

Use binary for: images, videos, PDFs, executables, any non-text file.

**[23:00]** Best practices summary:

Always use 'with' statements
- Use pathlib for paths
- Specify encoding explicitly (UTF-8)
- Handle errors gracefully
- Use binary mode for non-text files
- EAFP over LBYL
- Be careful with mode 'w'

Follow these and your agent's file handling will be robust and professional."

---

## Section 9: Complete Example and Summary [24:00 - 28:00]

### Presenter Notes
- Show comprehensive real-world example
- Tie everything together
- Recap key concepts
- Motivate for practice

### On-Screen Content
**CODE EXAMPLE 16: Complete Document Processor**
```python
from pathlib import Path
import json
import csv

def process_documents(input_dir, output_dir):
    """Process all JSON documents and create CSV summary."""
    input_path = Path(input_dir)
    output_path = Path(output_dir)

    # Ensure output directory exists
    output_path.mkdir(parents=True, exist_ok=True)

    results = []

    # Process each JSON file
    for json_file in input_path.glob('*.json'):
        try:
            with open(json_file, 'r') as f:
                data = json.load(f)

            # Extract key information
            result = {
                'filename': json_file.name,
                'doc_id': data.get('id', 'unknown'),
                'status': 'processed',
                'size': json_file.stat().st_size
            }
            results.append(result)

        except json.JSONDecodeError:
            print(f"Invalid JSON: {json_file}")
        except Exception as e:
            print(f"Error processing {json_file}: {e}")

    # Write summary CSV
    if results:
        with open(output_path / 'summary.csv', 'w', newline='') as f:
            writer = csv.DictWriter(f, fieldnames=results[0].keys())
            writer.writeheader()
            writer.writerows(results)

    return len(results)
```

### Script

**[24:00]** PRESENTER:
"Let's tie everything together with a complete example - a document processing agent:

**[DEMONSTRATE CODE EXAMPLE 16]**

**[24:15]** This agent:
1. Takes an input directory of JSON files
2. Processes each file
3. Creates a CSV summary report

It demonstrates everything we learned:
- pathlib for cross-platform paths
- with statements for all file operations
- JSON loading with error handling
- CSV writing with DictWriter
- Directory creation
- Glob patterns for finding files

**[25:00]** This is realistic agent code. Business automation often involves: read files from a folder, process them, generate a report. This is that pattern.

**[25:15]** Notice the error handling. We catch JSON errors separately so one bad file doesn't stop the whole job. We try-except around the file processing. This is robust code that handles real-world messiness.

**[25:35]** Let's recap what we covered:

Opening files with 'with' statements - automatic cleanup, no leaks.

Reading files - read(), readlines(), or iterate line by line for large files.

Writing files - mode 'w' overwrites, mode 'a' appends.

CSV files - DictReader and DictWriter for working with tabular data.

JSON files - load()/dump() for files, loads()/dumps() for strings.

pathlib - modern, cross-platform path handling.

Error handling - EAFP, catch specific exceptions.

**[26:30]** File I/O is fundamental. Every agent uses it. Configuration, data processing, logging, reporting - all require file operations.

**[26:45]** Master these patterns:
- Context managers for all files
- pathlib for all paths
- JSON for configuration
- CSV for data exchange
- Proper error handling
- Binary mode for non-text

**[27:10]** Now it's your turn to practice. The hands-on lab has you build a file processing utility that reads, processes, and writes various file types. Use everything we covered. Take your time, refer back to this video, and experiment.

**[27:35]** In our next module, we'll learn about virtual environments and package management - how to create isolated Python environments and manage dependencies. Essential skills for agent development.

Thank you for learning with me today. You now have professional-level file I/O skills. See you next time!"

**[28:00]** END

---

## Production Notes

### Visual Elements
- Split screen: code editor and file system
- Show actual files being created/modified
- Display file contents alongside code
- Animate file pointer movement
- Use icons for different file types (CSV, JSON, TXT)

### Graphics Needed
- File operation flowchart (open → process → close)
- File modes comparison table
- pathlib vs os.path side-by-side
- CSV structure diagram
- JSON structure diagram
- Error handling decision tree

### Code Files
- All examples in course repository
- Sample data files (CSV, JSON, TXT)
- Reference implementation: `file_operations_complete.py`
- Lab starter: `file_operations_lab.py`
- Solutions for practice exercises

### Accessibility
- Closed captions with file paths clearly spelled
- Transcript with all code samples
- High contrast for file listings
- Clear narration of all operations

### Common Student Questions (FAQ)
1. "When should I use binary mode?"
2. "Why does 'w' mode erase my file?"
3. "What's the difference between dump() and dumps()?"
4. "How do I read really large files?"
5. "Why use pathlib instead of strings?"

---

## Timing Breakdown

| Section | Duration | Topics |
|---------|----------|--------|
| 1. Introduction | 2:00 | Overview, objectives |
| 2. Opening/Closing | 3:30 | with statements, modes |
| 3. Reading | 3:00 | read(), readlines(), iteration |
| 4. Writing | 2:30 | write(), append, logging |
| 5. CSV | 3:30 | reader, writer, DictReader |
| 6. JSON | 3:00 | load, dump, configuration |
| 7. Paths | 3:00 | pathlib, cross-platform |
| 8. Errors | 3:30 | handling, EAFP, binary |
| 9. Example/Summary | 4:00 | Complete example, recap |
| **Total** | **28:00** | |

---

## Related Resources
- Python File I/O: docs.python.org/3/tutorial/inputoutput.html
- pathlib Module: docs.python.org/3/library/pathlib.html
- csv Module: docs.python.org/3/library/csv.html
- json Module: docs.python.org/3/library/json.html
- Real Python - Reading and Writing Files
