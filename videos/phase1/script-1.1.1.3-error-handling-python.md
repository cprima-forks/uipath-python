# Video Script: Error Handling in Python

**Module**: 1.1.1.3
**Duration**: 22 minutes
**Target Audience**: Beginners learning Python for UiPath Agent development
**Prerequisites**: Module 1.1.1.2 (Python Functions and Modules)

---

## Section 1: Introduction [00:00 - 02:00]

### Presenter Notes
- Start with a relatable story about errors
- Emphasize that errors are inevitable, handling them is what matters
- Set a reassuring tone - errors are normal, not failures
- Connect to agent reliability

### On-Screen Content
**TITLE SLIDE**
- "Error Handling in Python"
- "Building Robust and Reliable Agents"

### Script

**[00:00]** PRESENTER:
"Welcome back! In our previous modules, we learned about variables, data types, and functions - the building blocks of Python programming. Today, we're going to learn something equally important: how to handle errors.

Here's the reality: things go wrong. Files don't exist. Network connections fail. Users enter invalid data. APIs timeout. These aren't questions of 'if' - they're questions of 'when'.

**[00:45]** Without error handling, when something goes wrong, your entire program crashes. Imagine an agent processing thousands of documents, and it crashes on document number 573 because one file has an unexpected format. All that work, lost.

With proper error handling, your agent gracefully handles the problem, logs what happened, and continues processing. That's the difference between a toy program and a production-ready agent.

**[01:15]** By the end of this video, you'll be able to:
- Use try-except-finally blocks to handle errors
- Catch and handle specific exception types
- Create custom exceptions for your domain
- Implement logging for debugging

**[01:45]** This might seem defensive, but trust me - every professional Python developer spends significant time on error handling. It's what makes software reliable. Let's dive in!"

---

## Section 2: The Basics of Try-Except [02:00 - 05:00]

### Presenter Notes
- Start with the problem (code that crashes)
- Show the solution (try-except)
- Explain the flow of execution clearly
- Use simple, clear examples

### On-Screen Content
**CODE EXAMPLE 1: Without Error Handling**
```python
def divide(a, b):
    return a / b

result = divide(10, 0)  # CRASH! ZeroDivisionError
print("This never prints")
```

**CODE EXAMPLE 2: With Error Handling**
```python
def divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        print("Error: Cannot divide by zero")
        return None

result = divide(10, 0)  # Handles error gracefully
print("Program continues")  # This prints!
```

### Script

**[02:00]** PRESENTER:
"Let's start with a classic example - division by zero.

**[DEMONSTRATE CODE EXAMPLE 1]**

**[02:15]** This code crashes. Python raises a ZeroDivisionError, and the program stops. The print statement never executes. In a real application, this could mean losing data, leaving resources open, or leaving your system in an inconsistent state.

Now let's fix it:

**[DEMONSTRATE CODE EXAMPLE 2]**

**[02:45]** We wrap the risky code in a 'try' block. If an error occurs, Python jumps to the matching 'except' block instead of crashing. We handle the error gracefully - print a message, return None - and the program continues normally.

**[03:15]** The flow is:
1. Execute code in the try block
2. If an error occurs, jump to the except block
3. Continue with code after the try-except

**[03:30]** Here's a more realistic example for agent development:

```python
def load_config(file_path):
    try:
        with open(file_path) as f:
            return json.load(f)
    except FileNotFoundError:
        print(f"Config file {file_path} not found, using defaults")
        return {}
```

If the config file doesn't exist, we return empty config instead of crashing. The agent can still run with default settings.

**[04:15]** This is the fundamental pattern: try risky code, catch specific errors, handle them gracefully. Everything else builds on this foundation."

---

## Section 3: Catching Specific Exceptions [05:00 - 08:30]

### Presenter Notes
- Explain why catching specific exceptions matters
- Show the danger of bare except
- Demonstrate multiple except blocks
- Use practical examples

### On-Screen Content
**CODE EXAMPLE 3: Multiple Except Blocks**
```python
def read_config(path):
    try:
        with open(path) as f:
            config = json.load(f)
        return config
    except FileNotFoundError:
        print(f"File {path} not found")
        return {}
    except json.JSONDecodeError:
        print(f"Invalid JSON in {path}")
        return {}
    except PermissionError:
        print(f"No permission to read {path}")
        return {}
```

**CODE EXAMPLE 4: Good vs Bad**
```python
# ❌ BAD: Catches everything, hides bugs
try:
    code()
except:
    pass

# ✅ GOOD: Catches specific issues
try:
    code()
except ValueError:
    handle_error()
```

### Script

**[05:00]** PRESENTER:
"Not all errors are the same. A missing file is different from invalid JSON is different from a permission problem. Each needs different handling.

**[DEMONSTRATE CODE EXAMPLE 3]**

**[05:20]** We can have multiple except blocks, each catching a different exception type. Python tries them in order and executes the first one that matches.

If the file doesn't exist, we use defaults. If the JSON is invalid, we use defaults. If we don't have permission, we use defaults. Each error gets logged with specific context.

**[06:00]** Now, here's a critical best practice:

**[DEMONSTRATE CODE EXAMPLE 4]**

**[06:15]** Never use a bare 'except:' clause. It catches EVERYTHING - including KeyboardInterrupt (when you press Ctrl+C) and SystemExit (when you try to exit the program). It will hide bugs and make your code nearly impossible to debug.

Always catch specific exception types. If you really need to catch anything, use 'except Exception:' which catches all errors but not system-level exceptions.

**[06:45]** How do you know which exceptions to catch? Three ways:

1. Read the documentation for the functions you're calling
2. Run the code and see what exceptions occur
3. Use your IDE - many will suggest likely exceptions

**[07:15]** You can also catch the exception object to get details:

```python
try:
    result = int(user_input)
except ValueError as e:
    print(f"Conversion failed: {e}")
    # Conversion failed: invalid literal for int() with base 10: 'hello'
```

The 'as e' gives you the exception object, which contains error details, messages, and context. Very useful for logging!

**[07:45]** You can even catch multiple exception types with the same handling:

```python
try:
    process_data()
except (ValueError, TypeError) as e:
    print(f"Data error: {e}")
```

The parentheses create a tuple of exception types. If either occurs, this block executes."

---

## Section 4: Finally and Else Clauses [08:30 - 11:30]

### Presenter Notes
- Explain finally's critical role in cleanup
- Show why it's better than just putting code after try-except
- Demonstrate the else clause
- Show practical examples

### On-Screen Content
**CODE EXAMPLE 5: Finally Block**
```python
file = None
try:
    file = open("data.txt")
    data = file.read()
    process(data)
except FileNotFoundError:
    print("File not found")
except PermissionError:
    print("Permission denied")
finally:
    # ALWAYS runs
    if file:
        file.close()
        print("File closed")
```

**CODE EXAMPLE 6: Try-Except-Else-Finally**
```python
try:
    result = calculate_score()
except ValueError:
    print("Calculation error")
    score = 0
else:
    # Runs ONLY if no exception
    print(f"Success: {result}")
    save_result(result)
finally:
    # ALWAYS runs
    log_attempt()
```

### Script

**[08:30]** PRESENTER:
"Python's try-except can have two additional clauses: 'finally' and 'else'. Let's understand what they do.

**[DEMONSTRATE CODE EXAMPLE 5]**

**[08:45]** The 'finally' block ALWAYS executes - whether an exception occurred or not, whether we handled it or not, even if we return from the try or except block. It's guaranteed.

This makes it perfect for cleanup code: closing files, releasing locks, closing network connections - things that absolutely must happen.

**[09:15]** Why not just put the cleanup code after the try-except? Two reasons:

1. If the try block has a return statement, code after try-except won't run
2. If an unhandled exception occurs, code after try-except won't run

Finally guarantees execution. It's your safety net.

**[09:45]** Here's a better way to handle files:

```python
# ✅ BEST: Context manager handles cleanup automatically
try:
    with open("data.txt") as f:
        data = f.read()
        process(data)
except FileNotFoundError:
    print("File not found")
```

The 'with' statement is a context manager. It automatically closes the file when the block ends, even if an exception occurs. This is the preferred pattern in modern Python.

**[10:20]** Now, the else clause:

**[DEMONSTRATE CODE EXAMPLE 6]**

**[10:30]** The 'else' clause runs only if NO exception occurred in the try block. It's useful for code that should run only on success.

Think of it like this:
- try: "Attempt this risky operation"
- except: "If it fails, do this"
- else: "If it succeeds, do this"
- finally: "No matter what, do this"

**[11:00]** Here's when I use each:
- try: Code that might raise an exception
- except: Error handling
- else: Success-only code (keeps try block minimal)
- finally: Cleanup that must happen

You won't always need all four, but knowing when to use each makes your code clearer."

---

## Section 5: Custom Exceptions [11:30 - 14:30]

### Presenter Notes
- Explain when and why to create custom exceptions
- Show the simple syntax
- Demonstrate practical use cases for agents
- Emphasize clarity and maintainability

### On-Screen Content
**CODE EXAMPLE 7: Creating Custom Exceptions**
```python
class AgentConfigError(Exception):
    """Raised when agent configuration is invalid."""
    pass

class ConfidenceThresholdError(Exception):
    """Raised when confidence threshold is out of range."""
    def __init__(self, threshold):
        self.threshold = threshold
        message = f"Threshold {threshold} must be between 0 and 1"
        super().__init__(message)

# Usage
def set_threshold(value):
    if not (0 <= value <= 1):
        raise ConfidenceThresholdError(value)
    return value
```

**CODE EXAMPLE 8: Practical Agent Exceptions**
```python
class DataValidationError(Exception):
    """Data failed validation checks."""
    pass

class APIConnectionError(Exception):
    """Failed to connect to external API."""
    pass

class AgentTimeoutError(Exception):
    """Agent exceeded maximum execution time."""
    pass
```

### Script

**[11:30]** PRESENTER:
"Python has many built-in exceptions - ValueError, TypeError, FileNotFoundError, and so on. But sometimes you need exceptions specific to your domain.

**[DEMONSTRATE CODE EXAMPLE 7]**

**[11:45]** Creating a custom exception is simple: make a class that inherits from Exception. That's it. The simplest version just has 'pass' in the body.

For more sophisticated exceptions, you can add an __init__ method to customize the error message or store additional data.

**[12:15]** Here, ConfidenceThresholdError stores the actual threshold value and creates a helpful error message. This makes debugging much easier.

**[12:30]** When should you create custom exceptions? When you need:

1. Domain-specific error types
2. Callers to handle your errors differently
3. Clear, self-documenting error messages

**[DEMONSTRATE CODE EXAMPLE 8]**

**[12:50]** Look at these exceptions for an agent. They're immediately clear:
- DataValidationError: The input data is invalid
- APIConnectionError: We couldn't connect to an API
- AgentTimeoutError: The agent took too long

Someone calling your agent code can now catch and handle each differently:

```python
try:
    result = run_agent(data)
except DataValidationError:
    return "Please provide valid input"
except APIConnectionError:
    return "Service temporarily unavailable"
except AgentTimeoutError:
    return "Processing took too long"
```

**[13:30]** This is much better than catching generic ValueError for everything. The exception type itself documents what went wrong.

**[13:45]** Here's a complete practical example:

```python
class InvalidAgentInputError(Exception):
    """Raised when agent input fails validation."""
    pass

def validate_agent_input(user_input):
    if not user_input:
        raise InvalidAgentInputError("Input cannot be empty")

    if not isinstance(user_input, str):
        raise InvalidAgentInputError(
            f"Input must be string, got {type(user_input).__name__}"
        )

    if len(user_input) > 10000:
        raise InvalidAgentInputError(
            f"Input too long: {len(user_input)} chars (max 10000)"
        )

    return user_input.strip()
```

Clear, specific, maintainable. This is professional error handling."

---

## Section 6: Raising and Re-raising Exceptions [14:30 - 17:00]

### Presenter Notes
- Show when to raise exceptions
- Explain re-raising for logging
- Demonstrate exception chaining
- Connect to debugging workflows

### On-Screen Content
**CODE EXAMPLE 9: Raising Exceptions**
```python
def process_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age seems unrealistic")
    return age

def process_confidence(score):
    if not isinstance(score, (int, float)):
        raise TypeError("Confidence must be a number")
    if not (0.0 <= score <= 1.0):
        raise ValueError("Confidence must be between 0 and 1")
    return score
```

**CODE EXAMPLE 10: Re-raising**
```python
def process_with_logging(data):
    try:
        result = risky_operation(data)
        return result
    except Exception as e:
        # Log the error
        logger.error(f"Operation failed: {e}")
        # Re-raise to let caller handle it
        raise
```

**CODE EXAMPLE 11: Exception Chaining**
```python
class DataProcessingError(Exception):
    pass

def process_file(path):
    try:
        with open(path) as f:
            data = json.load(f)
        return process_data(data)
    except FileNotFoundError as e:
        raise DataProcessingError(f"Failed to process {path}") from e
```

### Script

**[14:30]** PRESENTER:
"So far we've caught exceptions. Now let's talk about raising them.

**[DEMONSTRATE CODE EXAMPLE 9]**

**[14:45]** You raise exceptions to signal that something is wrong. Use them to enforce rules, validate input, and make errors explicit.

When should you raise an exception? When:
1. You can't complete the operation safely
2. The caller passed invalid arguments
3. You're in an invalid state

Use the appropriate built-in exception type when possible: ValueError for invalid values, TypeError for wrong types, etc.

**[15:30]** Sometimes you want to catch an exception, do something (like logging), then let it propagate. Use 'raise' without arguments:

**[DEMONSTRATE CODE EXAMPLE 10]**

**[15:45]** This logs the error but re-raises the same exception. The caller can still catch and handle it. The traceback is preserved.

This is great for logging layers - you log the error but don't interfere with how the caller handles it.

**[16:10]** Exception chaining is even more powerful:

**[DEMONSTRATE CODE EXAMPLE 11]**

**[16:20]** The 'from e' syntax chains exceptions. When you catch a low-level exception (FileNotFoundError) and raise a higher-level one (DataProcessingError), you preserve the original exception.

The traceback will show both exceptions:

```
DataProcessingError: Failed to process data.json
    The above exception was the direct cause of the following exception:
FileNotFoundError: [Errno 2] No such file or directory: 'data.json'
```

**[16:45]** This is incredibly valuable for debugging. You see the high-level problem ("data processing failed") and the root cause ("file not found"). Always use 'from' when re-raising at a different level."

---

## Section 7: Logging and Best Practices [17:00 - 20:00]

### Presenter Notes
- Show importance of logging
- Demonstrate logging module basics
- Present comprehensive best practices
- Give practical patterns for agents

### On-Screen Content
**CODE EXAMPLE 12: Logging**
```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def process_with_logging(data):
    try:
        result = risky_operation(data)
        logger.info(f"Successfully processed {len(data)} items")
        return result
    except ValueError as e:
        logger.error(f"Value error: {e}")
        raise
    except Exception as e:
        logger.exception(f"Unexpected error: {e}")
        raise
```

**CODE EXAMPLE 13: Agent Error Strategy**
```python
def agent_task(input_data):
    # 1. Validate input early
    try:
        validated = validate_input(input_data)
    except ValidationError as e:
        logger.error(f"Invalid input: {e}")
        return {"status": "error", "message": str(e)}

    # 2. Handle specific known errors
    try:
        result = process_data(validated)
    except NetworkError:
        logger.error("Network error")
        return {"status": "retry", "message": "Network unavailable"}
    except DataError as e:
        logger.error(f"Data error: {e}")
        return {"status": "error", "message": "Invalid data"}
    # 3. Catch unexpected errors
    except Exception as e:
        logger.exception("Unexpected error")
        return {"status": "error", "message": "Internal error"}

    # 4. Return success
    return {"status": "success", "result": result}
```

### Script

**[17:00]** PRESENTER:
"Print statements are great for learning, but in production code, you need proper logging.

**[DEMONSTRATE CODE EXAMPLE 12]**

**[17:15]** Python's logging module gives you structured, level-based logging. You can log at different levels: DEBUG, INFO, WARNING, ERROR, CRITICAL. Then configure which levels to see.

The key method is logger.exception() - it automatically includes the full traceback. Use it in except blocks for unexpected errors.

**[17:45]** With logging, you can:
- Control what gets logged with log levels
- Write logs to files, not just console
- Add timestamps and context automatically
- Search logs for debugging

In production agents, logging is essential. When something goes wrong at 3 AM, logs are how you figure out what happened.

**[18:15]** Let's put everything together with a complete error handling strategy:

**[DEMONSTRATE CODE EXAMPLE 13]**

**[18:30]** This demonstrates professional error handling:

1. Validate input early - fail fast on bad data
2. Handle specific known errors differently
3. Always catch unexpected errors (the safety net)
4. Log everything with context
5. Return structured responses, not just values

**[19:00]** Notice the return structure: {"status": ..., "message": ...}. Consistent response format makes error handling easier for callers.

**[19:15]** Key best practices:

DO:
- Catch specific exceptions
- Use finally for cleanup
- Log errors with context
- Create custom exceptions
- Provide helpful error messages
- Test error cases

DON'T:
- Use bare except
- Silently swallow errors
- Return different types on error (None vs value)
- Ignore the finally block
- Forget to test error paths

**[19:45]** Error handling might seem defensive or pessimistic. It's not. It's what makes your agents production-ready. The difference between code that works in demos and code that works in production is robust error handling."

---

## Section 8: Summary and Next Steps [20:00 - 22:00]

### Presenter Notes
- Recap key concepts
- Emphasize the importance
- Give actionable practice steps
- End encouragingly

### On-Screen Content
**SUMMARY SLIDE**
```
Key Takeaways:
✓ Use try-except to handle errors
✓ Catch specific exceptions
✓ Finally always runs (cleanup)
✓ Else runs only on success
✓ Create custom exceptions
✓ Log errors for debugging
✓ Test error handling
```

### Script

**[20:00]** PRESENTER:
"Let's recap what we've learned about error handling in Python.

We started with try-except blocks - the foundation of error handling. Wrap risky code in try, catch specific exceptions in except blocks.

We learned about finally and else clauses. Finally always runs - use it for cleanup. Else runs only if no exception occurred - use it for success-only code.

**[20:30]** We created custom exceptions for domain-specific errors. This makes your code self-documenting and gives callers precise control over error handling.

We learned to raise and re-raise exceptions. Use raise to signal errors. Use re-raising to log while preserving the exception chain.

And we learned to log errors properly with Python's logging module. Logging is essential for production code.

**[21:00]** Here's the truth: beginners often skip error handling. They think 'I'll add it later.' But error handling isn't decoration - it's foundation. Build it in from the start.

Every function that might fail needs error handling. Every agent that processes user input needs validation. Every production system needs logging.

**[21:30]** For practice:
1. Add error handling to code you've already written
2. Create custom exception classes for a project
3. Implement logging in a script
4. Write tests specifically for error cases

Don't just test the happy path. Test what happens when files don't exist, when input is invalid, when the network fails.

**[21:50]** In our next module, we'll learn about file I/O operations - reading and writing files. And of course, we'll use everything we learned today to handle file errors gracefully.

Thank you for learning with me. Remember: errors aren't failures - they're opportunities to make your code more robust. See you next time!"

**[22:00]** END

---

## Production Notes

### Visual Elements
- Show error messages and tracebacks clearly
- Use color to distinguish try, except, else, finally blocks
- Animate exception flow (from try to except)
- Show before/after comparisons (with and without error handling)
- Display log output alongside code

### Graphics Needed
- Exception hierarchy diagram
- Try-except-else-finally flow chart
- Error handling decision tree
- Best practices checklist
- Common exceptions reference card

### Code Files
- All examples in course repository
- Reference implementation: `error_handling_complete.py`
- Lab starter: `error_handling_lab.py`
- Solutions for practice exercises

### Accessibility
- Closed captions with clear error messages
- Transcript with code samples
- High contrast for error messages
- Clearly narrate all code and errors

### Common Student Questions (FAQ)
1. "When should I use try-except vs if-else?"
2. "Is it okay to catch Exception?"
3. "How do I know which exceptions to catch?"
4. "Why use custom exceptions instead of ValueError?"
5. "Should I log in except block or let caller handle it?"

---

## Timing Breakdown

| Section | Duration | Topics |
|---------|----------|--------|
| 1. Introduction | 2:00 | Why error handling matters |
| 2. Try-Except | 3:00 | Basic syntax and flow |
| 3. Specific Exceptions | 3:30 | Catching, multiple blocks |
| 4. Finally/Else | 3:00 | Cleanup, success handling |
| 5. Custom Exceptions | 3:00 | Creating, using |
| 6. Raising | 2:30 | raise, re-raise, chaining |
| 7. Logging | 3:00 | Best practices, patterns |
| 8. Summary | 2:00 | Recap, next steps |
| **Total** | **22:00** | |

---

## Related Resources
- Python Exceptions: docs.python.org/3/tutorial/errors.html
- Logging HOWTO: docs.python.org/3/howto/logging.html
- PEP 3134 - Exception Chaining
- Real Python - Python Exceptions
