# Video Script: Handling Job Results
## Module 2.2.1.3 - SDK Fundamentals

**Duration:** 20 minutes
**Target Audience:** Developers processing job outputs and errors
**Prerequisites:** Module 2.2.1.2 (job monitoring)
**Learning Outcomes:** Extract, validate, and analyze job results

---

## Scene 1: Introduction (1.5 minutes)

### Visual: Title slide

**[On camera - Instructor]**

Welcome to Module 2.2.1.3: Handling Job Results. We've learned to start jobs and monitor them to completion. Now comes the payoff - getting the results!

### Visual: Pipeline diagram showing start → monitor → results

**[Voice over visual]**

Think of it like ordering food delivery. You place the order (start job), track its progress (monitor status), and finally receive your food (handle results). That last step - getting and verifying what you ordered - is what we're covering today.

### Visual: Learning objectives slide

**[On camera - Instructor]**

You'll learn how to:
- Parse output arguments from successful jobs
- Validate result data
- Analyze errors from failed jobs
- Access robot execution logs
- Calculate performance metrics

Let's dig into those results!

---

## Scene 2: Output Arguments Basics (3 minutes)

### Visual: UiPath Studio showing output arguments

**[On camera - Instructor]**

First, let's understand what output arguments are and where they come from.

### Visual: Studio workflow with output arguments highlighted

**[Voice over visual]**

In UiPath Studio, when you build a process, you can define output arguments - variables that return data to the caller. For example, `out_ProcessedCount` might return how many records were processed, `out_ResultData` might return calculation results.

When the process completes, these outputs are captured and stored in the job object.

### Screen recording - Accessing outputs

**[On camera - Instructor]**

Here's the key thing: OutputArguments are stored as a JSON *string*, not a Python dictionary.

**[Voice over screen recording]**

```python
job = wait_for_job(...)

# OutputArguments is a string!
print(type(job['OutputArguments']))
# <class 'str'>

print(job['OutputArguments'])
# '{"out_Count": 42, "out_Message": "Done"}'
```

We need to parse this JSON string to use it.

### Visual: Parsing demonstration

```python
import json

# Parse JSON string to dictionary
outputs = json.loads(job['OutputArguments'])

# Now we can access values
count = outputs['out_Count']  # 42
message = outputs['out_Message']  # "Done"

print(f"Processed {count} items: {message}")
```

### Visual: Output

```
Processed 42 items: Done
```

---

## Scene 3: Safe Parsing Pattern (2.5 minutes)

### Visual: Error scenarios

**[On camera - Instructor]**

But there's a problem - what if OutputArguments is None? Or an empty string? Or the process has no outputs? We need to handle these cases.

### Screen recording - Safe parsing

**[Voice over screen recording]**

Here's a production-ready parsing function:

```python
import json

def get_job_outputs(job):
    """Safely extract and parse output arguments"""
    
    # First, check job actually succeeded
    if job['State'] != 'Successful':
        return None
    
    # Get OutputArguments (might be None)
    output_json = job.get('OutputArguments')
    
    # Handle None or empty string
    if not output_json:
        return {}  # Return empty dict
    
    # Parse with error handling
    try:
        outputs = json.loads(output_json)
        return outputs
    except json.JSONDecodeError as e:
        print(f"Failed to parse outputs: {e}")
        return None
```

### Visual: Testing edge cases

**[Voice over]**

Let me test this with different scenarios:

```python
# Job with outputs
job1 = {'State': 'Successful', 'OutputArguments': '{"count": 5}'}
print(get_job_outputs(job1))  # {'count': 5}

# Job with no outputs
job2 = {'State': 'Successful', 'OutputArguments': None}
print(get_job_outputs(job2))  # {}

# Faulted job
job3 = {'State': 'Faulted', 'OutputArguments': '...'}
print(get_job_outputs(job3))  # None
```

Perfect - handles all cases gracefully!

---

## Scene 4: Complex Data Types (3 minutes)

### Visual: Different data type examples

**[On camera - Instructor]**

Real processes return complex data - lists, nested objects, even entire tables. Let's see how to handle these.

### Screen recording - Working with lists

**[Voice over screen recording]**

Lists come through as JSON arrays:

```python
outputs = get_job_outputs(job)

# List of items
items = outputs.get('out_ProcessedItems', [])
# ['Item1', 'Item2', 'Item3']

for item in items:
    print(f"Processing {item}")
```

### Visual: DataTable handling

**[Voice over]**

DataTables are especially important. In UiPath, a DataTable becomes a list of dictionaries:

```python
# DataTable with columns: Name, Age, City
table_data = outputs.get('out_Customers', [])

print(table_data)
# [
#   {'Name': 'Alice', 'Age': 30, 'City': 'NYC'},
#   {'Name': 'Bob', 'Age': 25, 'City': 'LA'}
# ]

# Process each row
for row in table_data:
    print(f"{row['Name']} from {row['City']}, age {row['Age']}")
```

### Visual: Converting to pandas

**[Voice over]**

If you're doing data analysis, convert to a pandas DataFrame:

```python
import pandas as pd

table_data = outputs.get('out_Results', [])
df = pd.DataFrame(table_data)

# Now use pandas methods
print(df.describe())
print(f"Average age: {df['Age'].mean()}")
```

This gives you all of pandas' powerful data manipulation capabilities.

---

## Scene 5: Output Validation (2.5 minutes)

### Visual: Why validate?

**[On camera - Instructor]**

Even if a job succeeds, the outputs might not be what you expect. Maybe a process bug returned the wrong type, or a field is missing. Always validate.

### Screen recording - Validation function

**[Voice over screen recording]**

Here's a validation pattern I use:

```python
def validate_outputs(outputs, expected_schema):
    """Validate outputs against expected structure"""
    
    errors = []
    
    for key, expected_type in expected_schema.items():
        # Check key exists
        if key not in outputs:
            errors.append(f"Missing required output: {key}")
            continue
        
        # Check type
        value = outputs[key]
        if not isinstance(value, expected_type):
            errors.append(
                f"Output {key}: expected {expected_type.__name__}, "
                f"got {type(value).__name__}"
            )
    
    return errors
```

### Visual: Using validation

```python
# Define what we expect
expected = {
    'out_ProcessedCount': int,
    'out_Success': bool,
    'out_Message': str,
    'out_Results': list
}

# Validate
outputs = get_job_outputs(job)
errors = validate_outputs(outputs, expected)

if errors:
    print("❌ Validation failed:")
    for error in errors:
        print(f"  - {error}")
else:
    print("✅ All outputs valid")
    # Safe to use outputs
```

---

## Scene 6: Handling Errors (3 minutes)

### Visual: Faulted job diagram

**[On camera - Instructor]**

When jobs fail, the Info field contains detailed error information. Let's learn to extract useful debugging data from it.

### Screen recording - Accessing error info

**[Voice over screen recording]**

```python
job = wait_for_job(...)

if job['State'] == 'Faulted':
    error_info = job.get('Info', 'No error details')
    print(f"Job failed with error:\n{error_info}")
```

### Visual: Error info structure

**[Voice over]**

The Info field typically contains:

```
System.NullReferenceException: Object reference not set to an instance...
   at UiPath.Core.Activities.Click.Execute(CodeActivityContext context)
   at System.Activities.CodeActivity.InternalExecute(...)
   ... (stack trace continues)
```

It's the exception type, message, and full stack trace.

### Screen recording - Parsing errors

**[Voice over]**

Let me write a parser to extract structured information:

```python
def parse_error_info(error_info):
    """Extract key error details"""
    
    lines = error_info.split('\n')
    first_line = lines[0] if lines else ''
    
    # Parse "ExceptionType: message" format
    if ':' in first_line:
        exc_type, exc_message = first_line.split(':', 1)
        exc_type = exc_type.strip()
        exc_message = exc_message.strip()
    else:
        exc_type = 'Unknown'
        exc_message = first_line
    
    return {
        'exception_type': exc_type,
        'message': exc_message,
        'full_trace': error_info
    }
```

### Visual: Using parsed errors

```python
if job['State'] == 'Faulted':
    error_data = parse_error_info(job['Info'])
    
    print(f"Exception: {error_data['exception_type']}")
    print(f"Message: {error_data['message']}")
    
    # Different handling based on error type
    if 'SelectorNotFoundException' in error_data['exception_type']:
        print("⚠️ UI element not found - check selectors")
    elif 'TimeoutException' in error_data['exception_type']:
        print("⚠️ Operation timed out - consider increasing timeout")
```

---

## Scene 7: Robot Logs (2.5 minutes)

### Visual: Log levels diagram

**[On camera - Instructor]**

For deep debugging, access the robot's execution logs. These show exactly what happened during the job.

### Screen recording - Fetching logs

**[Voice over screen recording]**

Robot logs are accessed via a separate endpoint:

```python
# Get logs for a specific job
response = requests.get(
    f"{ORCHESTRATOR_URL}/odata/RobotLogs",
    headers={
        "Authorization": f"Bearer {token}",
        "X-UIPATH-OrganizationUnitId": str(folder_id)
    },
    params={
        "$filter": f"JobKey eq '{job_key}'",
        "$orderby": "TimeStamp asc",
        "$select": "TimeStamp,Level,Message"
    }
)

logs = response.json()['value']
```

### Visual: Log output

**[Voice over]**

Each log entry has:
- TimeStamp: when it was logged
- Level: Trace, Info, Warn, Error, Fatal
- Message: the log message

Let me show you how to analyze these:

```python
# Group by level
errors = [log for log in logs if log['Level'] == 'Error']
warnings = [log for log in logs if log['Level'] == 'Warn']

print(f"Found {len(errors)} errors and {len(warnings)} warnings")

# Show errors
for error_log in errors:
    print(f"[{error_log['TimeStamp']}] {error_log['Message']}")
```

This gives you the complete execution picture!

---

## Scene 8: Performance Metrics (2 minutes)

### Visual: Metrics dashboard concept

**[On camera - Instructor]**

Job objects include timing information. Let's calculate execution metrics.

### Screen recording - Duration calculation

**[Voice over screen recording]**

```python
from datetime import datetime

def calculate_duration(job):
    """Calculate job execution time"""
    
    if not job.get('EndTime'):
        return None  # Job not finished
    
    # Parse ISO timestamps
    start = datetime.fromisoformat(
        job['StartTime'].replace('Z', '+00:00')
    )
    end = datetime.fromisoformat(
        job['EndTime'].replace('Z', '+00:00')
    )
    
    duration = end - start
    
    return {
        'seconds': duration.total_seconds(),
        'formatted': str(duration)
    }
```

### Visual: Using metrics

```python
job = wait_for_job(...)

duration = calculate_duration(job)

print(f"Execution time: {duration['formatted']}")
print(f"Total seconds: {duration['seconds']}")

# Performance check
if duration['seconds'] > 300:  # 5 minutes
    print("⚠️ Job took longer than expected")
```

---

## Scene 9: Complete Example (2 minutes)

### Screen recording - End-to-end

**[On camera - Instructor]**

Let me show you everything together in a real example.

**[Voice over screen recording]**

```python
# Start job
jobs = start_process_job(
    orchestrator_url, token, folder_id, release_key,
    {'in_FilePath': 'data.xlsx'}
)
job_id = jobs[0]['Key']

# Wait for completion
job = wait_for_job(orchestrator_url, token, folder_id, job_id)

# Handle results
if job['State'] == 'Successful':
    # Get outputs
    outputs = get_job_outputs(job)
    
    # Validate
    expected = {'out_ProcessedCount': int, 'out_Success': bool}
    errors = validate_outputs(outputs, expected)
    
    if not errors:
        # Process results
        count = outputs['out_ProcessedCount']
        success = outputs['out_Success']
        
        # Calculate performance
        duration = calculate_duration(job)
        
        print(f"✅ Processed {count} records in {duration['seconds']}s")
        print(f"Success: {success}")
    else:
        print("❌ Output validation failed:", errors)

elif job['State'] == 'Faulted':
    # Parse error
    error = parse_error_info(job['Info'])
    print(f"❌ Job failed: {error['message']}")
    print(f"Exception: {error['exception_type']}")
    
    # Get logs for more details
    logs = get_job_logs(orchestrator_url, token, folder_id, job_id)
    error_logs = [l for l in logs if l['Level'] == 'Error']
    print(f"Found {len(error_logs)} error log entries")
```

### Visual: Complete output

There - comprehensive result handling!

---

## Scene 10: Hands-On Exercise (30 seconds)

### Visual: Exercise instructions

**[On camera - Instructor]**

Your turn! Pause and complete this exercise:

1. Implement get_job_outputs with safe parsing
2. Create output validation for your process
3. Parse error info from a faulted job
4. Retrieve and analyze robot logs
5. Calculate execution duration

Time: 20 minutes. Solution provided.

---

## Scene 11: Conclusion (30 seconds)

### Visual: Key takeaways

**[On camera - Instructor]**

Excellent work! You now have complete control over job results. Remember:
- Always parse OutputArguments safely
- Validate expected structure
- Extract meaningful error information
- Use robot logs for debugging
- Track performance metrics

### Visual: Next module preview

Next up: Batch Process Execution - efficiently running multiple jobs in parallel and aggregating results at scale.

### Visual: End card

Thanks for watching! You're mastering Core Services. See you next time!

---

## Production Notes

**Graphics needed:**
- Output arguments flow diagram
- JSON parsing visualization
- DataTable to list of dicts conversion
- Validation pattern flowchart
- Error info structure
- Log levels hierarchy

**Screen recordings:**
- Basic output access and parsing
- Safe parsing function
- Complex data types (lists, DataTables)
- Validation implementation
- Error parsing from faulted job
- Robot logs retrieval
- Duration calculation
- Complete end-to-end example

**Code examples:**
- get_job_outputs function
- validate_outputs function
- parse_error_info function
- Robot logs query
- calculate_duration function
- Complete integration

**Callouts/Annotations:**
- Highlight JSON string vs dictionary
- Mark validation checks
- Annotate error structure
- Show log levels
- Point out timestamps

**Pacing notes:**
- Clear explanation of JSON parsing
- Methodical validation building
- Practical error handling
- Smooth integration example

**Accessibility:**
- Accurate captions
- Describe all data structures verbally
- High-contrast code
- Verbal description of diagrams
