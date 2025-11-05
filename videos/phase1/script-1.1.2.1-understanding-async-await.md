# Video Script: Understanding Async/Await

**Module:** 1.1.2.1
**Duration:** 35 minutes
**Target Audience:** UiPath Agent Developers (Beginner to Intermediate)
**Prerequisites:** Python basics, functions

---

## Section 1: Introduction (3 minutes)

**[VISUAL: Title slide with module number and learning objectives]**

**Presenter:**

"Welcome to Module 1.1.2.1: Understanding Async/Await. This is a crucial module for anyone building UiPath agents that need to handle multiple operations efficiently.

In this session, we're going to explore asynchronous programming in Python—a powerful technique that allows your agents to handle multiple tasks concurrently without blocking.

By the end of this module, you'll be able to define and use coroutines, understand how the event loop works, and know when to use async versus sync patterns in your UiPath agents.

Let's start with a real-world scenario: Imagine you're building an agent that needs to process 100 documents. Each document requires an API call that takes 2 seconds. With synchronous code, that's 200 seconds—over 3 minutes! With async code, you could process all 100 documents in just 2 seconds. That's the power of async programming.

Let's dive in."

**[VISUAL: Transition to problem statement slide]**

---

## Section 2: The Problem with Synchronous Code (4 minutes)

**[VISUAL: Code editor showing synchronous example]**

**Presenter:**

"First, let's understand the problem we're solving. Here's typical synchronous code that fetches data from three APIs:

```python
import time

def fetch_data(name):
    print(f'Fetching {name}...')
    time.sleep(2)  # Simulates network delay
    print(f'Got {name}')
    return f'Data from {name}'

start = time.time()
data1 = fetch_data('API-1')
data2 = fetch_data('API-2')
data3 = fetch_data('API-3')
print(f'Total time: {time.time() - start:.2f}s')
```

**[VISUAL: Run the code, show output]**

Notice what happens: We fetch API-1, wait 2 seconds. Then fetch API-2, wait 2 seconds. Then API-3, wait another 2 seconds. Total: 6 seconds.

**[VISUAL: Timeline diagram showing sequential execution]**

The problem is that while we're waiting for each API call, our program is completely idle. It's not doing anything useful—just waiting. This is called 'blocking' because each operation blocks the next one from starting.

Think of it like a restaurant where the waiter takes one order, goes to the kitchen, waits there doing nothing until the food is ready, delivers it, and only then takes the next order. Very inefficient!

Now let's see the async solution."

**[VISUAL: Code editor showing async example]**

**Presenter:**

"Here's the same logic using async/await:

```python
import asyncio

async def fetch_data(name):
    print(f'Fetching {name}...')
    await asyncio.sleep(2)
    print(f'Got {name}')
    return f'Data from {name}'

async def main():
    start = time.time()
    results = await asyncio.gather(
        fetch_data('API-1'),
        fetch_data('API-2'),
        fetch_data('API-3')
    )
    print(f'Total time: {time.time() - start:.2f}s')

asyncio.run(main())
```

**[VISUAL: Run the code, show output]**

Look at the total time: 2 seconds! We're running all three API calls concurrently. While waiting for one API, we're also waiting for the others. This is the power of async programming.

**[VISUAL: Timeline diagram showing concurrent execution]**

Now the waiter takes all three orders, the kitchen cooks them all at once, and the waiter can do other tasks while waiting. Much more efficient!"

---

## Section 3: Concurrency vs Parallelism (3 minutes)

**[VISUAL: Diagram comparing concurrency and parallelism]**

**Presenter:**

"Before we go further, let's clarify an important distinction: concurrency versus parallelism. Many people confuse these terms.

**Concurrency** means multiple tasks make progress by switching between them. Think of it like juggling—you're only touching one ball at a time, but all balls stay in the air by rapidly switching your attention.

**[VISUAL: Animation of task switching]**

Python's async/await provides concurrency. It's single-threaded, but the event loop rapidly switches between tasks at await points.

**Parallelism**, on the other hand, means multiple tasks run simultaneously on multiple CPU cores. This requires threads or processes.

**[VISUAL: Animation of parallel execution]**

Here's the key: Async is NOT parallelism. It doesn't use multiple CPU cores. But for I/O-bound operations—like network requests, file operations, database queries—you don't need multiple cores. You just need to avoid wasting time waiting.

Async excels at I/O-bound work. For CPU-bound work, like heavy calculations, you'd use multiprocessing instead. We'll cover that in a later module."

---

## Section 4: Coroutines and Async Functions (4 minutes)

**[VISUAL: Code editor showing function definitions]**

**Presenter:**

"Now let's learn the core building block of async programming: the coroutine.

A coroutine is a special function that can pause and resume its execution. You define a coroutine using 'async def' instead of just 'def':

```python
# Regular function
def regular_function():
    return 42

# Coroutine (async function)
async def coroutine_function():
    return 42
```

**[VISUAL: Highlight the 'async' keyword]**

The 'async' keyword tells Python: this function can pause and resume. It transforms the function into a coroutine function.

Now here's something crucial: when you call a coroutine function, it doesn't execute immediately. Instead, it returns a coroutine object:

```python
coro = coroutine_function()
print(type(coro))  # <class 'coroutine'>
```

**[VISUAL: Run the code, show output]**

This is different from regular functions! A regular function executes immediately when called. A coroutine function returns a coroutine object that must be awaited or run by the event loop.

This is one of the most common beginner mistakes: calling an async function and expecting it to run:

```python
async def get_value():
    return 42

result = get_value()  # This does NOT return 42!
print(result)  # <coroutine object>
```

**[VISUAL: Show the warning message]**

If you do this, Python will warn you: 'coroutine was never awaited'. This means you created a coroutine but never executed it.

To execute a coroutine, you must either await it inside another async function:

```python
async def main():
    result = await get_value()  # Now it runs
    print(result)  # 42
```

Or run it from synchronous code using asyncio.run():

```python
result = asyncio.run(get_value())
print(result)  # 42
```

We'll explore both of these patterns in detail."

---

## Section 5: The Await Keyword (4 minutes)

**[VISUAL: Diagram showing await behavior]**

**Presenter:**

"The await keyword is where the magic happens. It does three things:

First, it suspends the current coroutine.
Second, it yields control back to the event loop.
Third, it resumes the coroutine when the awaited operation completes.

Let's see this in action:

```python
async def example():
    print('Before await')
    result = await some_async_operation()
    print('After await')
    return result
```

**[VISUAL: Step-by-step execution diagram]**

Here's what happens: The function executes normally until it hits 'await'. At that point, execution pauses, and control goes back to the event loop. The event loop can now run other tasks. When 'some_async_operation' completes, the event loop resumes our function at the line after await.

This is fundamentally different from regular function calls. With a regular function call, you wait there until it completes. With await, you yield control, allowing other work to happen.

**[VISUAL: Code editor]**

Here's a concrete example:

```python
async def task1():
    print('Task 1: Start')
    await asyncio.sleep(0)  # Yield control
    print('Task 1: End')

async def task2():
    print('Task 2: Start')
    await asyncio.sleep(0)  # Yield control
    print('Task 2: End')

async def main():
    await asyncio.gather(task1(), task2())

asyncio.run(main())
```

**[VISUAL: Run the code, show output]**

The output is:
```
Task 1: Start
Task 2: Start
Task 1: End
Task 2: End
```

Notice how both tasks start before either ends. They interleave at the await points. This is the event loop switching between them.

**[VISUAL: Animation of event loop switching]**

One important rule: you can only use await inside async functions. If you try to use await in a regular function, Python will give you a syntax error. The async keyword enables the await keyword."

---

## Section 6: The Event Loop (4 minutes)

**[VISUAL: Diagram of event loop architecture]**

**Presenter:**

"Let's talk about the event loop—the engine that makes all of this work.

The event loop is a continuous loop that:
- Schedules coroutines to run
- Switches between them at await points
- Handles I/O operations
- Manages callbacks and timers

**[VISUAL: Flowchart of event loop operation]**

Think of the event loop as a task scheduler. It has a queue of tasks. It picks a task, runs it until it hits an await, then switches to the next task. When an awaited operation completes, the event loop resumes that task.

Here's the key insight: Python's async is single-threaded. There's only one event loop running on one thread. But it achieves concurrency by rapidly switching between tasks.

**[VISUAL: Code editor showing asyncio.run()]**

To run async code from synchronous code, you use asyncio.run():

```python
async def my_agent():
    print('Agent starting...')
    await asyncio.sleep(1)
    print('Agent completed!')
    return 'Success'

result = asyncio.run(my_agent())
print(result)
```

asyncio.run() does three things:
1. Creates a new event loop
2. Runs your coroutine to completion
3. Closes the event loop

**[VISUAL: Warning symbol]**

Important: You can only call asyncio.run() once per program. Once the event loop closes, you can't reuse it. This is a common pitfall:

```python
# Wrong!
asyncio.run(task1())
asyncio.run(task2())  # Error!
```

**[VISUAL: Correct pattern]**

Instead, create a single main() function:

```python
# Correct!
async def main():
    await task1()
    await task2()

asyncio.run(main())
```

This is your entry point from synchronous code into the async world."

---

## Section 7: Running Multiple Coroutines (4 minutes)

**[VISUAL: Code editor showing asyncio.gather()]**

**Presenter:**

"Now let's learn how to run multiple coroutines concurrently. Python provides several tools for this. The most common is asyncio.gather():

```python
async def fetch_user(user_id):
    await asyncio.sleep(1)
    return f'User {user_id}'

async def fetch_orders(user_id):
    await asyncio.sleep(1)
    return f'Orders for {user_id}'

async def main():
    user, orders = await asyncio.gather(
        fetch_user(123),
        fetch_orders(123)
    )
    print(user)
    print(orders)

asyncio.run(main())
```

**[VISUAL: Run the code, show timing]**

asyncio.gather() runs all the coroutines concurrently and waits for all of them to complete. It returns the results in order. Notice this takes 1 second total, not 2, because both operations run concurrently.

**[VISUAL: Code editor showing asyncio.create_task()]**

Another approach is asyncio.create_task():

```python
async def main():
    # Create tasks (they start immediately)
    task1 = asyncio.create_task(fetch_user(123))
    task2 = asyncio.create_task(fetch_orders(123))

    # Do other work here...
    print('Tasks running in background...')

    # Wait for results when needed
    user = await task1
    orders = await task2
```

The difference: create_task() starts the task immediately and gives you a task object. You can await it later. This is useful when you want to start tasks, do other work, then collect results.

**[VISUAL: Comparison diagram]**

gather() is simpler when you just want to run multiple things and wait for all results.

create_task() gives you more control—you can cancel tasks, check their status, or collect results at different times.

For most UiPath agent scenarios, gather() is what you'll use."

---

## Section 8: Async Libraries (3 minutes)

**[VISUAL: Table of sync vs async libraries]**

**Presenter:**

"Here's a critical concept: You cannot use regular synchronous libraries in async code. Well, technically you can, but they'll block your event loop and defeat the entire purpose of async.

Let me show you the wrong way:

```python
import time

async def bad_example():
    time.sleep(2)  # Wrong! Blocks the event loop
    return 'Done'
```

**[VISUAL: Diagram showing blocked event loop]**

time.sleep() is synchronous. When you call it, the entire event loop stops. No other tasks can run. You've blocked everything.

Instead, use async equivalents:

```python
async def good_example():
    await asyncio.sleep(2)  # Correct! Yields control
    return 'Done'
```

**[VISUAL: List of async library alternatives]**

This applies to all I/O operations:

- Don't use requests → Use aiohttp
- Don't use open() → Use aiofiles
- Don't use psycopg2 → Use asyncpg
- Don't use redis → Use aioredis

**[VISUAL: Code example with aiohttp]**

Here's a real example using aiohttp:

```python
import aiohttp

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = [
        'https://api.example.com/users',
        'https://api.example.com/products',
        'https://api.example.com/orders'
    ]

    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)

    print(f'Fetched {len(results)} URLs concurrently')
```

All three HTTP requests happen concurrently. This is massively faster than sequential requests.

The pattern is: use async with for async context managers, and await for async operations."

---

## Section 9: Async Context Managers and Iterators (3 minutes)

**[VISUAL: Code comparison of sync vs async context managers]**

**Presenter:**

"You're familiar with context managers in Python—the 'with' statement:

```python
with open('file.txt') as f:
    data = f.read()
```

For async operations, use 'async with':

```python
async with aiofiles.open('file.txt') as f:
    data = await f.read()
```

**[VISUAL: Highlight the async keyword and await]**

Notice two differences: 'async with' and 'await' for the read operation. This ensures the file operations don't block the event loop.

Common async context managers include:
- File operations with aiofiles
- HTTP sessions with aiohttp
- Database connections with asyncpg

**[VISUAL: Code editor showing async iterators]**

Similarly, Python has async iterators using 'async for':

```python
async def fetch_pages():
    for page in range(1, 6):
        await asyncio.sleep(0.5)
        yield f'Page {page}'

async def main():
    async for page in fetch_pages():
        print(page)
```

**[VISUAL: Run the code]**

async for is useful for streaming data, paginated APIs, or any scenario where you're iterating over async operations.

The pattern is consistent:
- async def for async functions
- await for calling them
- async with for async context managers
- async for for async iterators

Once you understand these patterns, async code becomes straightforward."

---

## Section 10: Real-World Example - Document Processing (3 minutes)

**[VISUAL: Code editor with full agent example]**

**Presenter:**

"Let's build a realistic example: a document processing agent that handles multiple documents concurrently.

```python
import aiofiles
import asyncio

async def process_document(doc_path):
    '''Process a single document asynchronously.'''
    # Read file asynchronously
    async with aiofiles.open(doc_path) as f:
        content = await f.read()

    # Simulate processing (API call, analysis, etc.)
    await asyncio.sleep(1)

    return {
        'path': doc_path,
        'length': len(content),
        'status': 'processed'
    }

async def process_batch(doc_paths):
    '''Process multiple documents concurrently.'''
    print(f'Processing {len(doc_paths)} documents...')

    # Create tasks for all documents
    tasks = [process_document(path) for path in doc_paths]

    # Run all tasks concurrently
    results = await asyncio.gather(*tasks)

    # Calculate statistics
    total_chars = sum(r['length'] for r in results)

    print(f'Processed {len(results)} documents')
    print(f'Total characters: {total_chars:,}')

    return results

# Usage
documents = ['doc1.txt', 'doc2.txt', 'doc3.txt']
results = asyncio.run(process_batch(documents))
```

**[VISUAL: Run the code, show output]**

This agent processes all documents concurrently. If each document takes 1 second, all three complete in 1 second total, not 3.

**[VISUAL: Diagram showing concurrent document processing]**

This is the pattern you'll use for UiPath agents:
1. Define an async function for each operation
2. Create tasks or use gather() for concurrent execution
3. Use async libraries for all I/O
4. Collect and return results

This scales beautifully—whether you're processing 10 documents or 100, the pattern is the same."

---

## Section 11: Common Pitfalls (2 minutes)

**[VISUAL: List of common mistakes]**

**Presenter:**

"Let me highlight the most common pitfalls to avoid:

**Pitfall 1: Forgetting await**

```python
async def main():
    result = get_data()  # Wrong! Creates coroutine but doesn't run it
    result = await get_data()  # Correct!
```

You'll see the warning: 'coroutine was never awaited'.

**Pitfall 2: Using synchronous blocking operations**

```python
async def bad():
    time.sleep(1)  # Blocks event loop!

async def good():
    await asyncio.sleep(1)  # Yields control
```

Always use async equivalents.

**Pitfall 3: Multiple asyncio.run() calls**

```python
asyncio.run(task1())
asyncio.run(task2())  # Error!
```

Create one main() function instead.

**Pitfall 4: Not handling exceptions in concurrent tasks**

```python
# If one task fails, gather() raises immediately
results = await asyncio.gather(task1(), task2(), task3())

# Better: collect exceptions
results = await asyncio.gather(
    task1(), task2(), task3(),
    return_exceptions=True
)
```

With return_exceptions=True, exceptions are returned as results instead of raised.

**[VISUAL: Checkmark]**

Avoid these pitfalls and your async code will be robust and efficient."

---

## Section 12: When to Use Async (2 minutes)

**[VISUAL: Decision tree diagram]**

**Presenter:**

"A common question: when should I use async?

**Use async for:**
- Network requests (APIs, webhooks)
- File I/O operations
- Database queries
- Multiple concurrent operations
- I/O-bound workloads

**Don't use async for:**
- CPU-intensive calculations
- Simple sequential scripts
- Quick utility functions
- CPU-bound workloads

**[VISUAL: Performance comparison chart]**

Here's the key insight: Async helps when you're waiting for I/O. It doesn't help with computation.

For CPU-bound work, the time is spent calculating, not waiting. Async adds overhead without benefit. Use multiprocessing instead for CPU-bound parallelism.

For UiPath agents, you'll typically use async because agents often:
- Call multiple APIs
- Process multiple documents
- Wait for external systems
- Handle concurrent user requests

These are perfect use cases for async programming."

---

## Section 13: Summary and Next Steps (2 minutes)

**[VISUAL: Key takeaways slide]**

**Presenter:**

"Let's recap what we've learned:

✅ Async/await enables concurrent I/O operations without blocking
✅ Coroutines are defined with 'async def' and called with 'await'
✅ The event loop schedules and switches between tasks
✅ asyncio.run() is your entry point from synchronous code
✅ asyncio.gather() runs multiple coroutines concurrently
✅ Always use async libraries to avoid blocking the event loop
✅ Async excels at I/O-bound work, not CPU-bound work

**[VISUAL: Next module preview]**

In the next module, Concurrent Execution Patterns, we'll dive deeper into:
- Advanced task management
- Timeouts and cancellation
- Semaphores and rate limiting
- Error handling strategies for concurrent code

**[VISUAL: Practice lab slide]**

For now, I encourage you to complete the hands-on lab where you'll convert synchronous code to async, build a concurrent document processor, and experiment with async libraries.

Practice is essential. Start with simple examples and gradually build complexity. The patterns you've learned today will serve you throughout your UiPath agent development journey.

Thank you for watching, and I'll see you in the next module!"

---

## Presenter Notes

### Key Teaching Points
1. **Start with the problem**: Show slow synchronous code before introducing async
2. **Emphasize concurrency ≠ parallelism**: This confuses many beginners
3. **Demonstrate with running code**: Don't just show code, run it and show timing
4. **Highlight common mistakes**: Forgetting await, blocking the loop
5. **Use real-world examples**: Document processing, API calls for UiPath agents

### Common Questions to Address
- "Why not just use threading?" → Threads have overhead, GIL limitations, and complexity. Async is simpler for I/O-bound work.
- "Can I mix sync and async code?" → Yes, but be careful. Never call blocking sync code from async.
- "How many concurrent tasks can I run?" → Thousands! Much more than threads. Limited by memory and system resources.

### Demo Tips
- Use asyncio.sleep() with very short times (0.5s) for demos to keep pace
- Show the actual timing differences with time.time()
- Run the code live when possible to show real behavior
- Use print statements to show execution order

### Troubleshooting Common Issues
- If students get "coroutine never awaited": Show them they forgot await
- If event loop errors occur: Show them they're calling asyncio.run() twice
- If code is slow despite using async: Check for blocking operations like time.sleep()

### Time Management
- Section 1-3: Foundation (10 minutes)
- Section 4-6: Core concepts (12 minutes)
- Section 7-9: Practical usage (10 minutes)
- Section 10-13: Application (3 minutes)

Total: 35 minutes with buffer for demonstrations
