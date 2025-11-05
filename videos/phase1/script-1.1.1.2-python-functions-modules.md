# Video Script: Python Functions and Modules

**Module**: 1.1.1.2
**Duration**: 22 minutes
**Target Audience**: Beginners learning Python for UiPath Agent development
**Prerequisites**: Module 1.1.1.1 (Python Variables and Data Types)

---

## Section 1: Introduction and Overview [00:00 - 02:00]

### Presenter Notes
- Welcome learners warmly
- Set clear expectations for what they'll learn
- Connect to real-world UiPath Agent development scenarios
- Show enthusiasm for the practical power of functions

### On-Screen Content
**TITLE SLIDE**
- "Python Functions and Modules"
- "Building Reusable Code"

### Script

**[00:00]** PRESENTER:
"Welcome back! In our previous module, we learned about Python variables and data types - the building blocks of Python programming. Today, we're going to learn how to organize and reuse our code through functions and modules. This is where Python really starts to become powerful and efficient.

By the end of this video, you'll be able to:
- Define and use functions with parameters and return values
- Work with flexible function arguments using *args and **kwargs
- Organize code into reusable modules
- Understand variable scope and namespaces
- Use lambda functions for quick, simple operations

These concepts are fundamental to building UiPath Agents. Every agent you create will use functions to organize its logic, and modules to structure your codebase. Let's dive in!"

---

## Section 2: Function Basics [02:00 - 05:30]

### Presenter Notes
- Start with the simplest possible function
- Build complexity gradually
- Emphasize the DRY principle (Don't Repeat Yourself)
- Show real debugging workflow

### On-Screen Content
**CODE EXAMPLE 1: Simple Function**
```python
def greet():
    print("Hello, UiPath Developer!")

greet()
```

**CODE EXAMPLE 2: Function with Parameters**
```python
def greet_user(name):
    print(f"Hello, {name}!")

greet_user("Alice")
greet_user("Bob")
```

**CODE EXAMPLE 3: Function with Return Value**
```python
def add_numbers(a, b):
    return a + b

result = add_numbers(5, 3)
print(f"The sum is: {result}")
```

### Script

**[02:00]** PRESENTER:
"Let's start with the basics. A function is a reusable block of code that performs a specific task. Think of it as a tool in your toolbox that you can use whenever you need it.

Here's the simplest function possible. We use the 'def' keyword to define a function, followed by the function name and parentheses. The code inside the function is indented.

**[DEMONSTRATE CODE EXAMPLE 1]**

We call the function by using its name followed by parentheses. This executes the code inside the function.

**[02:45]** But functions become really useful when they can work with different data. We do this using parameters. Parameters are variables that we pass into the function.

**[DEMONSTRATE CODE EXAMPLE 2]**

Here, 'name' is a parameter. When we call greet_user with 'Alice', the function uses 'Alice' as the value for name. When we call it with 'Bob', it uses 'Bob'. Same function, different behavior based on the input.

**[03:30]** Most functions need to give us something back - a result. We use the 'return' statement for this.

**[DEMONSTRATE CODE EXAMPLE 3]**

This function takes two numbers, adds them, and returns the result. We can store that result in a variable or use it directly. The return statement is how functions communicate their results to the rest of your code.

**[04:15]** Here's a key principle: if you find yourself writing the same code more than once, it probably belongs in a function. This is called the DRY principle - Don't Repeat Yourself. Functions help keep your code organized, maintainable, and easier to test.

**[04:45]** Let's see a practical example for UiPath Agents."

**CODE EXAMPLE 4: Practical Agent Example**
```python
def validate_email(email):
    """Check if an email address is valid."""
    if "@" in email and "." in email:
        return True
    return False

# Using in an agent workflow
user_email = "developer@uipath.com"
if validate_email(user_email):
    print("Email is valid, proceeding with agent task")
else:
    print("Invalid email, cannot proceed")
```

**[05:00]** PRESENTER:
"In this example, we have a validation function that an agent might use to check user input before processing. Notice the docstring - that's the text in triple quotes that describes what the function does. Always document your functions!"

---

## Section 3: Default Parameters and Keyword Arguments [05:30 - 08:00]

### Presenter Notes
- Show flexibility of Python function calls
- Emphasize readability with keyword arguments
- Explain common use cases in agent configuration

### On-Screen Content
**CODE EXAMPLE 5: Default Parameters**
```python
def create_agent_config(name, timeout=30, retries=3):
    """Create configuration for an agent."""
    config = {
        "name": name,
        "timeout": timeout,
        "retries": retries
    }
    return config

# Using defaults
config1 = create_agent_config("DataProcessor")

# Overriding defaults
config2 = create_agent_config("FileAnalyzer", timeout=60, retries=5)
```

**CODE EXAMPLE 6: Keyword Arguments**
```python
def send_notification(message, recipient, priority="normal", send_email=True):
    print(f"Sending {priority} priority message to {recipient}")
    print(f"Message: {message}")
    print(f"Email notification: {send_email}")

# Using positional and keyword arguments
send_notification("Task completed", "admin@uipath.com")

# More explicit with keywords
send_notification(
    message="Error occurred",
    recipient="dev-team@uipath.com",
    priority="high",
    send_email=True
)
```

### Script

**[05:30]** PRESENTER:
"Python functions are incredibly flexible. You can provide default values for parameters, making them optional.

**[DEMONSTRATE CODE EXAMPLE 5]**

In this function, 'name' is required, but 'timeout' and 'retries' have default values. If we don't provide them, the defaults are used. This is extremely useful for configuration functions in agents - you can provide sensible defaults but allow customization when needed.

**[06:30]** When calling functions, you can use keyword arguments to make your code more readable and to specify arguments in any order.

**[DEMONSTRATE CODE EXAMPLE 6]**

See how much clearer the second call is? We can immediately see what each value represents. This is especially helpful when a function has many parameters. In UiPath Agent development, this makes your code self-documenting and easier for your team to understand.

**[07:30]** Pro tip: When you have more than 3 parameters, consider using keyword arguments exclusively to improve readability. Your future self will thank you!"

---

## Section 4: *args and **kwargs [08:00 - 10:30]

### Presenter Notes
- Start with the problem these solve
- Show progression from fixed to flexible parameters
- Use relatable examples
- Emphasize when to use each

### On-Screen Content
**CODE EXAMPLE 7: Using *args**
```python
def calculate_total(*numbers):
    """Calculate the sum of any number of arguments."""
    total = 0
    for num in numbers:
        total += num
    return total

# Works with any number of arguments
print(calculate_total(1, 2, 3))           # 6
print(calculate_total(10, 20, 30, 40))    # 100
print(calculate_total(5))                  # 5
```

**CODE EXAMPLE 8: Using **kwargs**
```python
def create_agent(**properties):
    """Create an agent with flexible properties."""
    print("Creating agent with properties:")
    for key, value in properties.items():
        print(f"  {key}: {value}")
    return properties

agent = create_agent(
    name="DocumentProcessor",
    model="gpt-4",
    temperature=0.7,
    max_tokens=1000
)
```

**CODE EXAMPLE 9: Combining All Parameter Types**
```python
def process_data(required_param, *args, default_param="default", **kwargs):
    """Example combining all parameter types."""
    print(f"Required: {required_param}")
    print(f"Args: {args}")
    print(f"Default: {default_param}")
    print(f"Kwargs: {kwargs}")

process_data(
    "value1",
    "extra1", "extra2",
    default_param="custom",
    custom_key="custom_value"
)
```

### Script

**[08:00]** PRESENTER:
"Sometimes you don't know how many arguments a function will receive. That's where *args and **kwargs come in. Don't let the asterisks intimidate you - they're incredibly useful!

**[08:15]** Let's start with *args - that's 'arguments'. The asterisk lets you pass any number of positional arguments.

**[DEMONSTRATE CODE EXAMPLE 7]**

The name 'numbers' becomes a tuple containing all the arguments we passed. We can have 1 argument, 3 arguments, or 100 - the function handles them all. This is perfect for operations where you don't know ahead of time how many items you'll process.

**[09:00]** Now **kwargs - that's 'keyword arguments'. The double asterisk lets you pass any number of keyword arguments.

**[DEMONSTRATE CODE EXAMPLE 8]**

Inside the function, 'properties' is a dictionary containing all the keyword arguments. This is incredibly powerful for configuration and settings. You'll see this pattern a lot in UiPath Agent libraries where you need flexible configuration options.

**[09:45]** You can even combine regular parameters, *args, and **kwargs in the same function, but they must appear in this order: regular parameters, then *args, then keyword-only parameters, then **kwargs.

**[DEMONSTRATE CODE EXAMPLE 9]**

**[10:15]** Remember: *args collects extra positional arguments into a tuple, **kwargs collects extra keyword arguments into a dictionary. You don't have to call them 'args' and 'kwargs' - the asterisks are what matter - but it's a Python convention everyone follows."

---

## Section 5: Variable Scope and Namespaces [10:30 - 13:00]

### Presenter Notes
- Clarify common source of bugs
- Use visual metaphors (rooms in a house)
- Show both correct and incorrect examples
- Emphasize best practices

### On-Screen Content
**CODE EXAMPLE 10: Scope Basics**
```python
# Global scope
global_var = "I'm global"

def my_function():
    # Local scope
    local_var = "I'm local"
    print(global_var)  # Can access global
    print(local_var)   # Can access local

my_function()
# print(local_var)  # ERROR! Can't access local_var outside function
```

**CODE EXAMPLE 11: Scope Problems and Solutions**
```python
# Problem: Trying to modify global variable
counter = 0

def increment_wrong():
    counter = counter + 1  # ERROR! Can't modify before assignment

# Solution 1: Use global keyword (not recommended)
def increment_with_global():
    global counter
    counter = counter + 1

# Solution 2: Return the value (recommended)
def increment_better(value):
    return value + 1

counter = increment_better(counter)
```

**CODE EXAMPLE 12: Best Practice with Agent State**
```python
class AgentState:
    """Better approach: Use a class to manage state."""
    def __init__(self):
        self.counter = 0
        self.results = []

    def increment(self):
        self.counter += 1

    def add_result(self, result):
        self.results.append(result)

# Clean and clear
state = AgentState()
state.increment()
state.add_result("Task completed")
```

### Script

**[10:30]** PRESENTER:
"Now let's talk about scope - where your variables are visible and accessible. This is a common source of bugs for beginners, so pay close attention!

**[10:45]** Think of scope like rooms in a house. Variables defined in a function are like items in a bedroom - only accessible from that room. Global variables are like items in the living room - accessible from anywhere.

**[DEMONSTRATE CODE EXAMPLE 10]**

Inside the function, we can see both the global variable and the local variable. But outside the function, we can't see the local variable. It only exists while the function is running.

**[11:30]** Here's where people often get confused: modifying global variables from inside functions.

**[DEMONSTRATE CODE EXAMPLE 11]**

The first approach fails because Python sees you're trying to use 'counter' before you've assigned it locally. You can use the 'global' keyword, but this is generally considered bad practice - it makes code harder to test and debug.

The better approach is to pass the value as a parameter and return the new value. This makes your function 'pure' - it doesn't have side effects and is easier to test.

**[12:15]** For agent development, here's the best practice:

**[DEMONSTRATE CODE EXAMPLE 12]**

Use classes to encapsulate state. This gives you the benefits of encapsulation while keeping your code clean and testable. We'll learn more about classes in a future module, but this is the pattern you'll see in professional UiPath Agent code."

---

## Section 6: Lambda Functions [13:00 - 15:00]

### Presenter Notes
- Present as shorthand for simple functions
- Show when to use and when not to use
- Demonstrate common use cases with list operations
- Connect to agent data processing scenarios

### On-Screen Content
**CODE EXAMPLE 13: Lambda Basics**
```python
# Regular function
def square(x):
    return x ** 2

# Equivalent lambda
square_lambda = lambda x: x ** 2

print(square(5))          # 25
print(square_lambda(5))   # 25
```

**CODE EXAMPLE 14: Lambda with Built-in Functions**
```python
# Sorting agent results by confidence score
results = [
    {"text": "Result A", "confidence": 0.85},
    {"text": "Result B", "confidence": 0.92},
    {"text": "Result C", "confidence": 0.78}
]

# Sort by confidence (highest first)
sorted_results = sorted(results, key=lambda x: x["confidence"], reverse=True)

for result in sorted_results:
    print(f"{result['text']}: {result['confidence']}")
```

**CODE EXAMPLE 15: Lambda with map and filter**
```python
# Processing agent outputs
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Double all numbers
doubled = list(map(lambda x: x * 2, numbers))
print(f"Doubled: {doubled}")

# Get only even numbers
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(f"Evens: {evens}")

# Practical: Filter documents by size
documents = [
    {"name": "doc1.pdf", "size_mb": 2.5},
    {"name": "doc2.pdf", "size_mb": 15.2},
    {"name": "doc3.pdf", "size_mb": 0.8}
]

# Get documents under 10MB
small_docs = list(filter(lambda d: d["size_mb"] < 10, documents))
```

### Script

**[13:00]** PRESENTER:
"Lambda functions are Python's way of creating small, anonymous functions on the fly. Think of them as function shortcuts for simple operations.

**[13:15]** Here's a comparison:

**[DEMONSTRATE CODE EXAMPLE 13]**

Both do exactly the same thing, but the lambda is more concise. Lambda functions are defined using the 'lambda' keyword, followed by parameters, a colon, and the expression to return. You don't write 'return' - the expression result is automatically returned.

**[13:45]** Lambda functions shine when you need a simple function to pass to another function. Here's a common pattern in agent development:

**[DEMONSTRATE CODE EXAMPLE 14]**

We're using a lambda to tell the sorted function what part of each dictionary to use for sorting. This is much cleaner than defining a separate function just for this one use.

**[14:15]** Lambda functions are also commonly used with map and filter:

**[DEMONSTRATE CODE EXAMPLE 15]**

Map applies a function to every item in a list. Filter keeps only items where the function returns True. These are powerful tools for data processing in agents.

**[14:45]** Important guidelines: Use lambda for simple, one-line operations. If your logic is complex or needs multiple statements, write a regular function. Remember: code readability is more important than brevity!"

---

## Section 7: Modules and Imports [15:00 - 19:00]

### Presenter Notes
- Show progression from single file to organized project
- Demonstrate both creating and importing modules
- Cover important standard library modules for agents
- Show proper project structure

### On-Screen Content
**CODE EXAMPLE 16: Creating Your First Module**

**File: math_utils.py**
```python
"""Utility functions for mathematical operations."""

def add(a, b):
    """Add two numbers."""
    return a + b

def multiply(a, b):
    """Multiply two numbers."""
    return a * b

def calculate_percentage(part, whole):
    """Calculate percentage."""
    return (part / whole) * 100

# Module-level variable
PI = 3.14159
```

**File: main.py**
```python
# Import entire module
import math_utils

result = math_utils.add(5, 3)
print(f"5 + 3 = {result}")
print(f"PI = {math_utils.PI}")

# Import specific functions
from math_utils import multiply, calculate_percentage

product = multiply(4, 7)
percentage = calculate_percentage(45, 200)
print(f"4 × 7 = {product}")
print(f"45/200 = {percentage}%")

# Import with alias
from math_utils import add as addition
print(f"Using alias: {addition(10, 20)}")
```

**CODE EXAMPLE 17: Standard Library for Agents**
```python
# Operating system operations
import os
file_path = os.path.join("data", "documents", "file.txt")
print(f"Current directory: {os.getcwd()}")

# Working with JSON (common in agents)
import json
agent_config = {
    "name": "DocumentProcessor",
    "version": "1.0",
    "settings": {"max_tokens": 1000}
}
json_string = json.dumps(agent_config, indent=2)
config_back = json.loads(json_string)

# Date and time
from datetime import datetime
timestamp = datetime.now()
print(f"Agent started at: {timestamp}")

# Random numbers (for sampling)
import random
sample = random.sample(range(100), 5)
print(f"Random sample: {sample}")
```

**CODE EXAMPLE 18: Organizing an Agent Project**
```
my_agent_project/
├── main.py
├── config.py
├── agents/
│   ├── __init__.py
│   ├── document_agent.py
│   └── data_agent.py
├── utils/
│   ├── __init__.py
│   ├── validation.py
│   └── formatting.py
└── tests/
    ├── test_agents.py
    └── test_utils.py
```

**File: agents/__init__.py**
```python
"""Agent modules for the project."""
from .document_agent import DocumentAgent
from .data_agent import DataAgent

__all__ = ["DocumentAgent", "DataAgent"]
```

**File: main.py**
```python
from agents import DocumentAgent, DataAgent
from utils.validation import validate_input

# Clean imports from your organized modules
doc_agent = DocumentAgent()
data_agent = DataAgent()
```

### Script

**[15:00]** PRESENTER:
"As your code grows, keeping everything in one file becomes unmanageable. Modules let you organize code into separate files. Any Python file is a module!

**[15:15]** Let's create a simple module:

**[DEMONSTRATE CODE EXAMPLE 16]**

We create a file called 'math_utils.py' with some functions. Now in another file, we can import and use those functions. Notice the different ways to import: you can import the whole module, import specific items, or use aliases.

The docstring at the top of the module is important - it describes what the module does. This shows up when someone uses Python's help system.

**[16:15]** Python comes with a rich standard library - modules that are built in and ready to use. Here are some essential ones for agent development:

**[DEMONSTRATE CODE EXAMPLE 17]**

The 'os' module helps with file paths and operating system operations. The 'json' module is crucial for working with JSON data - you'll use this constantly with agents. The 'datetime' module handles dates and times - important for logging and timestamps. The 'random' module is useful for sampling and randomization.

You don't need to install these - they come with Python. Learning the standard library will make you much more productive.

**[17:15]** Now let's look at how to structure a real agent project:

**[DEMONSTRATE CODE EXAMPLE 18]**

This is a professional project structure. The '__init__.py' files make directories into Python packages. They can be empty or contain initialization code. The '__all__' list controls what gets imported when someone uses 'from package import *'.

Notice how we organize by functionality: agents in one folder, utilities in another, tests separate. This makes the codebase easy to navigate as it grows.

**[18:30]** When you start building UiPath Agents, you'll be importing from uipath libraries:

```python
from uipath.agents import Agent
from uipath.actions import Action
```

Understanding modules and imports is essential for working with any Python library, including the UiPath SDK. This is how modern Python development works - you're not writing everything yourself, you're composing functionality from modules!"

---

## Section 8: Best Practices and Summary [19:00 - 22:00]

### Presenter Notes
- Summarize key concepts
- Emphasize practical application
- Provide clear next steps
- End with encouragement and motivation

### On-Screen Content
**CODE EXAMPLE 19: Function Best Practices**
```python
# ✅ GOOD: Clear name, documented, single responsibility
def calculate_confidence_score(responses, threshold=0.7):
    """
    Calculate average confidence score from agent responses.

    Args:
        responses: List of response dictionaries with 'confidence' keys
        threshold: Minimum confidence to include (default: 0.7)

    Returns:
        float: Average confidence score

    Example:
        >>> responses = [{"confidence": 0.8}, {"confidence": 0.9}]
        >>> calculate_confidence_score(responses)
        0.85
    """
    valid_responses = [r for r in responses if r["confidence"] >= threshold]

    if not valid_responses:
        return 0.0

    total = sum(r["confidence"] for r in valid_responses)
    return total / len(valid_responses)


# ❌ BAD: Unclear name, no docs, does too much
def process(data, x):
    result = 0
    for item in data:
        if item["confidence"] >= x:
            result += item["confidence"]
    if len(data) > 0:
        result = result / len(data)
    return result
```

**SUMMARY SLIDE: Key Takeaways**
```
Functions & Modules Checklist:

✓ Functions encapsulate reusable logic
✓ Use parameters and return values for communication
✓ Default parameters provide flexibility
✓ *args and **kwargs handle variable arguments
✓ Understand scope to avoid bugs
✓ Lambda functions for simple operations
✓ Modules organize code into files
✓ Standard library provides powerful tools
✓ Import intelligently for clean code
✓ Document everything with docstrings
```

**NEXT STEPS SLIDE**
```
Practice Exercises:
1. Create a module with validation functions
2. Build a data processing pipeline using functions
3. Organize a small project into modules

Next Module: Error Handling in Python
- Try/except/finally blocks
- Custom exceptions
- Logging and debugging

Resources:
- Python Documentation: docs.python.org
- UiPath SDK Documentation
- Code examples in course repository
```

### Script

**[19:00]** PRESENTER:
"Let's wrap up with some best practices. Here's a side-by-side comparison of good and bad function design:

**[DEMONSTRATE CODE EXAMPLE 19]**

The good example has:
- A descriptive name that tells you what it does
- A comprehensive docstring with args, returns, and example
- Clear variable names
- Single responsibility - it does one thing well
- Edge case handling

The bad example is unclear and hard to maintain. When you're writing functions for agents, remember that other developers (or future you) will need to understand and modify this code.

**[19:45]** Let's review what we've covered today:

**[SHOW SUMMARY SLIDE]**

Functions are the fundamental building blocks of organized code. They let you break complex problems into manageable pieces. Modules let you organize those functions into a coherent structure.

When building UiPath Agents, you'll use functions to define agent behaviors, process data, and handle responses. You'll use modules to organize your agent code into maintainable projects.

**[20:30]** Here's what I want you to practice:

**[SHOW NEXT STEPS SLIDE]**

First, create a utility module with validation functions - check email formats, validate file paths, verify data types. This will reinforce what you learned about modules and functions.

Second, build a small data processing pipeline using functions. Take some input, transform it through several functions, and produce output. This will help you understand function composition.

Third, organize a small project into modules. Even if it's simple, practice creating the directory structure and using imports correctly.

**[21:15]** In our next module, we'll learn about error handling - how to gracefully handle things when they go wrong. This is crucial for building robust agents that can handle unexpected situations.

**[21:30]** Before we end, I want to emphasize something important: learning to write good functions is a skill that develops over time. Your first functions might not be perfect, and that's okay! Every expert developer started as a beginner. The key is to practice, get feedback, and keep improving.

**[21:50]** When you're ready, try the hands-on lab where you'll build a complete utility module for agent development. Take your time, refer back to this video if needed, and don't hesitate to experiment.

Thank you for learning with us today. You're building real skills that you'll use throughout your career as a UiPath Agent developer. See you in the next module!"

**[22:00]** END

---

## Production Notes

### Visual Elements to Include
- Split screen showing code editor and output console
- Highlight syntax as code is explained
- Animate the flow of data through functions
- Show directory structure visually for module organization
- Use consistent color coding for different code elements

### Graphics Needed
- Scope diagram showing global vs local variables
- Module import visualization showing file relationships
- Function flow diagram showing input → process → output
- Project structure tree diagram

### Code Files Referenced
- All code examples should be available in the course repository
- Create a completed reference version
- Create a starter template for the lab exercise

### Accessibility
- Include closed captions with technical terms spelled out
- Provide transcript with code samples in accessible format
- Use high contrast color scheme for code examples
- Clearly narrate all on-screen code

### Common Questions to Address (FAQ)
1. "When should I use a function vs. writing code inline?"
2. "What's the difference between parameters and arguments?"
3. "Can I have a function inside another function?"
4. "How do I know if my function is doing too much?"
5. "Should I always use type hints?"

---

## Timing Breakdown

| Section | Duration | Topics |
|---------|----------|--------|
| 1. Introduction | 2:00 | Overview, objectives |
| 2. Function Basics | 3:30 | Syntax, parameters, returns |
| 3. Parameters | 2:30 | Defaults, keywords |
| 4. Args/Kwargs | 2:30 | Variable arguments |
| 5. Scope | 2:30 | Namespaces, visibility |
| 6. Lambda | 2:00 | Anonymous functions |
| 7. Modules | 4:00 | Imports, organization |
| 8. Best Practices | 3:00 | Summary, next steps |
| **Total** | **22:00** | |

---

## Related Resources
- Python Functions Tutorial: python.org/tutorial
- PEP 8 Style Guide: python.org/dev/peps/pep-0008
- UiPath SDK Documentation
- Course Repository: github.com/uipath/python-course
