# Video Script: Python Variables and Data Types

**Module**: 1.1.1.1
**Duration**: 20 minutes
**Target Audience**: Beginners with no Python experience
**Prerequisites**: None

---

## Section 1: Introduction and Welcome [00:00 - 02:00]

### Presenter Notes
- Welcome learners enthusiastically
- Set expectations for the foundational nature of this module
- Connect to real-world UiPath Agent development
- Establish a supportive, encouraging tone

### On-Screen Content
**TITLE SLIDE**
- "Python Variables and Data Types"
- "Module 1.1.1.1 - Your Foundation for Agent Development"

### Script

**[00:00]** PRESENTER:
"Welcome to the UiPath Python SDK course! I'm excited to begin this journey with you. Whether you're completely new to programming or coming from another language, this course will teach you everything you need to build powerful UiPath Agents using Python.

We're starting at the very beginning - with variables and data types. Now, I know that might sound basic, but these concepts are absolutely fundamental. Every agent you build, every automation you create, will use variables and data types. Understanding them deeply will make everything else easier.

**[00:45]** By the end of this video, you'll be able to:
- Create and name variables following Python best practices
- Work confidently with Python's four primitive data types
- Convert between different types safely
- Apply professional naming conventions that make your code readable

**[01:15]** Here's why this matters for UiPath Agents: When you build an agent, you're constantly working with data - user inputs, API responses, configuration settings, results from calculations. All of that data lives in variables, and understanding how Python handles different types of data is crucial for writing reliable agents.

**[01:45]** So let's jump right in. Don't worry if some concepts seem new - we'll take them step by step, and by the end, you'll be writing real Python code with confidence!"

---

## Section 2: What are Variables? [02:00 - 04:30]

### Presenter Notes
- Use relatable metaphors (labeled boxes)
- Show the simplicity of Python's approach
- Emphasize that Python infers types automatically
- Build confidence by showing how easy it is

### On-Screen Content
**CODE EXAMPLE 1: First Variables**
```python
agent_name = "DocumentProcessor"
confidence_score = 0.95
retry_count = 3
is_active = True
```

**VISUAL**: Show labeled boxes with values inside

### Script

**[02:00]** PRESENTER:
"So what exactly is a variable? Think of it as a labeled container - a box with a name tag on it that holds some piece of information.

In Python, creating a variable is beautifully simple. Let's look at an example:

**[DEMONSTRATE CODE EXAMPLE 1]**

**[02:30]** We just created four variables. Let me break down what's happening:
- `agent_name` is a variable holding text: 'DocumentProcessor'
- `confidence_score` holds a decimal number: 0.95
- `retry_count` holds a whole number: 3
- `is_active` holds a true/false value

Notice we didn't have to tell Python what type of data each variable would hold. Python figured it out automatically from the value we assigned. This is called 'dynamic typing' and it's one of the things that makes Python so easy to learn.

**[03:15]** The equals sign is the assignment operator. It takes the value on the right and stores it in the variable on the left. Think of it as putting a value into the labeled box.

We can also change what's in a variable:

```python
retry_count = 3
retry_count = 4  # Now it's 4
retry_count = 5  # Now it's 5
```

The variable name stays the same, but the value inside changes.

**[03:45]** Here's something cool - we can assign multiple variables at once:

```python
x, y, z = 10, 20, 30
```

Now x is 10, y is 20, and z is 30. You'll see this pattern often in Python code, especially when a function returns multiple values.

**[04:15]** Variables are essential because they let us:
- Store data to use later
- Give meaningful names to values
- Update information as our program runs
- Build complex logic step by step

Think of variables as the memory of your program - they remember information so you don't have to!"

---

## Section 3: Variable Naming Rules and Conventions [04:30 - 07:30]

### Presenter Notes
- Clearly distinguish between rules (must follow) and conventions (should follow)
- Show plenty of good and bad examples
- Explain the reasoning behind PEP 8 conventions
- Emphasize readability

### On-Screen Content
**CODE EXAMPLE 2: Naming Rules**
```python
# ✅ VALID names
user_name = "Alice"
_private_key = "secret"
count2 = 10
MAX_SIZE = 100

# ❌ INVALID names
2nd_user = "Bob"      # Can't start with number
user-name = "Carol"   # No hyphens allowed
user name = "Dave"    # No spaces allowed
class = "Python"      # Can't use reserved words
```

**CODE EXAMPLE 3: PEP 8 Conventions**
```python
# ✅ Variables and functions: snake_case
user_age = 25
calculate_total()

# ✅ Constants: SCREAMING_SNAKE_CASE
MAX_RETRIES = 3
API_KEY = "abc123"

# ✅ Classes: PascalCase
class DocumentAgent:
    pass
```

### Script

**[04:30]** PRESENTER:
"Now let's talk about naming variables. Python has some strict rules you must follow, and some conventions you should follow for professional code.

**[04:45]** First, the rules - these aren't optional:

**[DEMONSTRATE CODE EXAMPLE 2]**

Variable names must:
1. Start with a letter or underscore - never a number
2. Contain only letters, numbers, and underscores - no hyphens, spaces, or special characters
3. Not be a Python keyword like 'class', 'def', 'if', 'for', etc.

Also, Python is case-sensitive. That means 'name', 'Name', and 'NAME' are three different variables. Be careful with capitalization!

**[05:45]** Now, the conventions - these are from PEP 8, Python's style guide. Following these isn't required, but it makes your code look professional and helps other Python developers understand it.

**[DEMONSTRATE CODE EXAMPLE 3]**

For regular variables and functions, use snake_case - that's all lowercase words separated by underscores. Like 'user_age' or 'total_count'.

For constants - values that shouldn't change - use SCREAMING_SNAKE_CASE. All capitals with underscores. Like 'MAX_RETRIES' or 'API_KEY'.

For class names, use PascalCase - capitalize the first letter of each word with no underscores. Like 'DocumentAgent' or 'DataProcessor'. We'll learn about classes in a future module.

**[06:45]** Here's a tip: if you're coming from JavaScript or Java, you might be tempted to use camelCase. Resist that urge! In Python, we use snake_case. When in Rome, do as the Romans do. When in Python, write Pythonic code!

**[07:00]** Finally, choose descriptive names. Look at these examples:

```python
# ❌ BAD: Too short, unclear
x = 30
usr = "Alice"

# ✅ GOOD: Clear and descriptive
user_age = 30
user_name = "Alice"
```

Your code is read far more often than it's written. Spend the extra second to type a clear name. Your future self will thank you!"

---

## Section 4: Python's Four Primitive Data Types [07:30 - 11:00]

### Presenter Notes
- Introduce all four types with clear examples
- Show practical uses for each type
- Demonstrate type() function
- Connect to agent development scenarios

### On-Screen Content
**CODE EXAMPLE 4: The Four Types**
```python
# Integer (int) - Whole numbers
age = 30
count = -5
big_number = 1_000_000

# Float (float) - Decimal numbers
temperature = 98.6
pi = 3.14159
score = 0.95

# String (str) - Text
name = "Alice"
message = 'Hello!'

# Boolean (bool) - True/False
is_active = True
has_error = False
```

### Script

**[07:30]** PRESENTER:
"Python has four primitive data types - these are the basic building blocks of all data in Python. Let's look at each one.

**[DEMONSTRATE CODE EXAMPLE 4]**

**[07:45]** First, integers - type 'int'. These are whole numbers, positive or negative, with no decimal point. 30, -5, 1000. In Python 3, integers can be as big as your computer's memory allows - there's no maximum size!

Notice that big_number uses underscores: 1_000_000. Python ignores these underscores; they're just there to make large numbers easier to read. One million is clearer than 1000000, right?

**[08:30]** Second, floats - type 'float'. These are numbers with decimal points. 98.6, 3.14159, 0.95. You use floats whenever you need fractional values. In agent development, confidence scores are typically floats between 0 and 1.

One thing to know about floats: they have limited precision. Due to how computers store decimal numbers, you might see tiny rounding errors:

```python
print(0.1 + 0.2)  # 0.30000000000000004
```

For most purposes, this doesn't matter. But if you're doing financial calculations where exactness matters, use Python's decimal module instead.

**[09:15]** Third, strings - type 'str'. These are sequences of characters - basically, text. You create strings by putting text in quotes.

You can use single quotes or double quotes - Python doesn't care:

```python
name1 = "Alice"
name2 = 'Bob'
```

Both work exactly the same. Pick one style and be consistent. I personally use double quotes, but many Python programmers use single quotes. Either is fine.

**[09:45]** For multi-line strings, use triple quotes:

```python
description = '''This is a
multi-line string that
spans several lines'''
```

Strings are incredibly important in agent development. User inputs, API responses, log messages, error descriptions - all strings.

**[10:15]** Fourth, booleans - type 'bool'. These have only two values: True or False. Notice the capital T and F - that's important in Python.

```python
is_active = True
has_error = False
```

Booleans are used for conditions, flags, and decision-making in your code. Is the agent running? Did the operation succeed? Should we retry? All boolean questions.

**[10:45]** You can check the type of any value using the type() function:

```python
print(type(42))          # <class 'int'>
print(type(3.14))        # <class 'float'>
print(type("hello"))     # <class 'str'>
print(type(True))        # <class 'bool'>
```

This is super useful when debugging - if you're not sure what type a variable is, just check!"

---

## Section 5: Type Conversion [11:00 - 14:00]

### Presenter Notes
- Show why type conversion is necessary
- Demonstrate conversion functions
- Warn about conversion errors
- Show practical examples with user input

### On-Screen Content
**CODE EXAMPLE 5: Type Conversion**
```python
# String to number
age_str = "25"
age_int = int(age_str)      # 25
age_float = float(age_str)  # 25.0

# Number to string
count = 42
count_str = str(count)      # "42"

# Float to int (truncates)
price = 19.99
price_int = int(price)      # 19 (loses .99!)

# Boolean conversions
int(True)    # 1
int(False)   # 0
bool(1)      # True
bool(0)      # False
```

**CODE EXAMPLE 6: Safe Conversion**
```python
user_input = "123"

try:
    value = int(user_input)
    print(f"You entered: {value}")
except ValueError:
    print("That's not a valid number!")
```

### Script

**[11:00]** PRESENTER:
"Often, you'll need to convert data from one type to another. This is called type conversion or type casting. Let's see how it works.

**[DEMONSTRATE CODE EXAMPLE 5]**

**[11:15]** The most common conversion is from strings to numbers. Why? Because when you get input from a user or read from a file, it usually comes as a string. Even if the user types '25', it comes as the string '25', not the number 25.

To convert a string to an integer, use int(). To convert to a float, use float(). And to convert anything to a string, use str().

**[11:45]** Watch out when converting floats to integers - it truncates the decimal part, it doesn't round:

```python
int(19.99)  # 19, not 20!
```

If you want rounding, use the round() function first:

```python
int(round(19.99))  # 20
```

**[12:15]** Here's something interesting: booleans in Python are actually a subtype of integers. True is 1, and False is 0:

```python
True + True   # 2
False * 10    # 0
```

This can be useful for counting:

```python
correct_answers = [True, False, True, True]
score = sum(correct_answers)  # 3
```

**[12:45]** Now, here's a critical point: not all conversions work. If you try to convert something that doesn't make sense, Python raises an error:

```python
int("hello")  # ValueError: invalid literal for int()
```

**[DEMONSTRATE CODE EXAMPLE 6]**

**[13:15]** In real code, especially for agents that process user input, you need to handle these errors gracefully. We'll learn more about error handling in Module 1.1.1.3, but the try-except pattern shown here is the basic approach.

**[13:30]** Here's a practical example for agent development:

```python
# Getting confidence threshold from user
threshold_input = "0.85"

# Convert to float for comparison
confidence_threshold = float(threshold_input)

if confidence_threshold < 0.0 or confidence_threshold > 1.0:
    print("Confidence must be between 0 and 1")
else:
    print(f"Threshold set to {confidence_threshold:.0%}")
```

Type conversion happens constantly in real programs. Master it now and you'll save yourself headaches later!"

---

## Section 6: Working with Strings [14:00 - 16:30]

### Presenter Notes
- Show string operations and methods
- Emphasize f-strings for formatting
- Demonstrate practical examples
- Connect to agent output formatting

### On-Screen Content
**CODE EXAMPLE 7: String Operations**
```python
# Concatenation
greeting = "Hello" + " " + "World"  # "Hello World"

# Repetition
laugh = "Ha" * 3  # "HaHaHa"

# Indexing
name = "Alice"
first = name[0]     # "A"
last = name[-1]     # "e"

# Length
length = len(name)  # 5

# Methods
upper = name.upper()     # "ALICE"
lower = name.lower()     # "alice"
```

**CODE EXAMPLE 8: String Formatting**
```python
# f-strings (recommended!)
name = "Alice"
age = 30
message = f"My name is {name} and I'm {age} years old"

# Formatting numbers
score = 0.856
print(f"Confidence: {score:.2%}")  # "Confidence: 85.60%"
print(f"Score: {score:.3f}")       # "Score: 0.856"

# Expressions in f-strings
print(f"Next year I'll be {age + 1}")
```

### Script

**[14:00]** PRESENTER:
"Let's dive deeper into strings since you'll use them constantly in agent development.

**[DEMONSTRATE CODE EXAMPLE 7]**

**[14:15]** Strings have many useful operations. You can concatenate them with the plus sign, repeat them with the asterisk, and access individual characters using square brackets.

Python uses zero-based indexing, so the first character is at position 0. Negative indices count from the end: -1 is the last character, -2 is second to last, and so on.

**[14:45]** Strings also have many helpful methods:

```python
text = "  Hello World  "
text.strip()        # "Hello World" (removes whitespace)
text.replace("Hello", "Hi")  # "  Hi World  "
text.split()        # ["Hello", "World"]
```

These methods don't change the original string - they return a new string. Strings in Python are immutable, meaning they can't be changed after creation.

**[15:15]** Now, let's talk about string formatting - this is super important for creating readable output.

**[DEMONSTRATE CODE EXAMPLE 8]**

**[15:30]** The modern way to format strings in Python is f-strings. Put an 'f' before the opening quote, and you can include variables and expressions inside curly braces.

F-strings are fantastic because they're:
- Fast
- Readable
- Powerful

You can format numbers, do calculations, call functions - all inside the curly braces.

**[16:00]** The format specifiers after the colon are really useful:
- `.2f` means 2 decimal places
- `.2%` means percentage with 2 decimal places
- `:,` adds thousand separators

Here's a practical agent example:

```python
agent_name = "DocumentProcessor"
documents_processed = 1_523
success_rate = 0.976

print(f"{agent_name} Status:")
print(f"Processed: {documents_processed:,} documents")
print(f"Success Rate: {success_rate:.1%}")
```

Output:
```
DocumentProcessor Status:
Processed: 1,523 documents
Success Rate: 97.6%
```

Professional, readable output makes your agents look polished!"

---

## Section 7: Practical Examples and Best Practices [16:30 - 19:00]

### Presenter Notes
- Show real-world agent scenarios
- Emphasize best practices
- Build confidence
- Prepare for hands-on work

### On-Screen Content
**CODE EXAMPLE 9: Agent Configuration**
```python
# Configuration for a document processing agent
AGENT_NAME = "DocumentProcessor"
AGENT_VERSION = "1.0.2"
MAX_RETRIES = 3
TIMEOUT_SECONDS = 30.0
CONFIDENCE_THRESHOLD = 0.85
IS_PRODUCTION = True

# Using the configuration
if IS_PRODUCTION:
    environment = "PRODUCTION"
    log_level = "INFO"
else:
    environment = "DEVELOPMENT"
    log_level = "DEBUG"

print(f"=== {AGENT_NAME} v{AGENT_VERSION} ===")
print(f"Environment: {environment}")
print(f"Confidence Threshold: {CONFIDENCE_THRESHOLD:.0%}")
print(f"Max Retries: {MAX_RETRIES}")
print(f"Timeout: {TIMEOUT_SECONDS}s")
```

**CODE EXAMPLE 10: Data Validation**
```python
def validate_user_input(value, min_val, max_val):
    """Validate that input is a number in range."""
    try:
        num = float(value)
    except ValueError:
        return False, "Must be a number"

    if num < min_val or num > max_val:
        return False, f"Must be between {min_val} and {max_val}"

    return True, num

# Usage
user_input = "0.75"
valid, result = validate_user_input(user_input, 0.0, 1.0)

if valid:
    confidence = result
    print(f"Valid confidence score: {confidence:.0%}")
else:
    print(f"Error: {result}")
```

### Script

**[16:30]** PRESENTER:
"Let's put everything together with some real-world examples you might use when building UiPath Agents.

**[DEMONSTRATE CODE EXAMPLE 9]**

**[16:45]** This is a typical agent configuration. Notice several best practices:
1. Constants are in SCREAMING_SNAKE_CASE
2. Variable names are descriptive
3. We use the right data types - integers for counts, floats for timeouts, booleans for flags
4. F-strings create clean, formatted output

**[17:15]** This isn't just theoretical - this is exactly the kind of code you'll write when configuring real agents. The better your variable names and organization, the easier your agent is to maintain.

**[17:30]** Here's another practical example - validating user input:

**[DEMONSTRATE CODE EXAMPLE 10]**

**[17:45]** This function demonstrates several concepts we've learned:
- Type conversion with error handling
- Clear variable names
- Descriptive error messages
- Returning multiple values (valid/invalid and the result/error)

In agent development, you're constantly validating inputs, converting types, and handling errors. These patterns will become second nature.

**[18:15]** Let me leave you with some best practices:

1. **Choose meaningful names** - `user_age` not `ua`
2. **Follow PEP 8** - Use snake_case for variables
3. **Check types when uncertain** - Use type() and isinstance()
4. **Convert types explicitly** - Don't assume types
5. **Handle conversion errors** - Use try-except for user input
6. **Use f-strings** - They're the modern, clean way to format
7. **Comment when helpful** - But good names often eliminate the need

**[18:45]** Remember: code is read far more than it's written. Write code that's easy to understand. Your teammates (and future you) will appreciate it!"

---

## Section 8: Summary and Next Steps [19:00 - 20:00]

### Presenter Notes
- Recap key concepts
- Motivate for hands-on practice
- Preview next module
- End on an encouraging note

### On-Screen Content
**SUMMARY SLIDE**
```
Key Takeaways:
✓ Variables store and label data
✓ Four primitive types: int, float, str, bool
✓ Follow naming conventions (snake_case)
✓ Convert types with int(), float(), str(), bool()
✓ Check types with type() and isinstance()
✓ Use f-strings for formatting
✓ Validate and handle errors
```

### Script

**[19:00]** PRESENTER:
"Congratulations! You've learned the foundations of Python programming. Let's recap what we covered:

Variables are named containers for data. Python has four primitive types - integers for whole numbers, floats for decimals, strings for text, and booleans for true/false values.

We learned to name variables following PEP 8 - using snake_case for clarity. We explored type conversion and why it's necessary when working with different data sources. And we saw how to use f-strings to create professional, readable output.

**[19:30]** These might seem like simple concepts, but they're the foundation everything else builds on. Every UiPath Agent you create will use variables and data types extensively.

Now it's time for you to practice. The hands-on lab will have you create variables, convert types, and format output for a real agent scenario. Take your time, experiment, and don't be afraid to make mistakes - that's how we learn!

**[19:50]** In our next module, we'll learn about functions and modules - how to organize code into reusable pieces. You'll see how the concepts from today fit into larger programs.

Thank you for learning with me today. You've taken your first step toward becoming a UiPath Agent developer. See you in the next module!"

**[20:00]** END

---

## Production Notes

### Visual Elements
- Show code editor with syntax highlighting
- Display output console alongside code
- Use split screen for before/after comparisons
- Animate type conversions visually (string → number)
- Show variable "boxes" holding values

### Graphics Needed
- Variable as labeled box diagram
- Data type comparison chart
- Type conversion flowchart
- Naming conventions reference card
- PEP 8 quick guide

### Code Files
- All examples available in course repository
- Completed reference file: `variables_complete.py`
- Starter template for lab: `variables_lab.py`
- Solutions for practice exercises

### Accessibility
- Closed captions with technical terms
- Transcript with code samples
- High contrast code examples
- Clear narration of all code

### Common Student Questions (FAQ)
1. "Why can't I use camelCase like JavaScript?"
2. "What's the difference between int and float?"
3. "When should I use single vs double quotes?"
4. "How do I know what type a variable is?"
5. "Why does 0.1 + 0.2 not equal 0.3 exactly?"

---

## Timing Breakdown

| Section | Duration | Topics |
|---------|----------|--------|
| 1. Introduction | 2:00 | Welcome, objectives |
| 2. Variables | 2:30 | What are variables, assignment |
| 3. Naming | 3:00 | Rules, conventions, PEP 8 |
| 4. Data Types | 3:30 | int, float, str, bool |
| 5. Conversion | 3:00 | Type casting, errors |
| 6. Strings | 2:30 | Operations, formatting |
| 7. Examples | 2:30 | Real-world patterns |
| 8. Summary | 1:00 | Recap, next steps |
| **Total** | **20:00** | |

---

## Related Resources
- Python Documentation: docs.python.org
- PEP 8 Style Guide: python.org/dev/peps/pep-0008
- Real Python Variables: realpython.com/python-variables
- Course Repository: github.com/uipath/python-course
