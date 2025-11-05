# Video Script: Common Async Pitfalls

**Module:** 1.1.2.3
**Duration:** 26 minutes
**Target Audience:** UiPath Agent Developers (Intermediate)
**Prerequisites:** Modules 1.1.2.1 and 1.1.2.2

---

## Section 1: Introduction (2 minutes)

**[VISUAL: Title slide with learning objectives]**

**Presenter:**

"Welcome to Module 1.1.2.3: Common Async Pitfalls. This is the final module in our async programming series, and it might be the most important one.

In the previous two modules, you learned how async/await works and how to use concurrent execution patterns. You now know the theory and the tools. But async programming is tricky, and there are many ways to make mistakes that kill performance, cause subtle bugs, or hide errors completely.

In this module, I'm going to show you the most common pitfalls that developers encounter when writing async code, and more importantly, how to avoid them.

We'll cover:
- Blocking the event loop—the #1 performance killer
- Race conditions and deadlocks
- Unhandled exceptions in background tasks
- Resource leaks and debugging techniques

By the end, you'll know what NOT to do, and you'll be able to write robust, production-grade async code.

Let's start with the most common and most damaging mistake: blocking the event loop."

**[VISUAL: Transition to code editor]**

---

## Section 2: Pitfall #1 - Blocking the Event Loop (5 minutes)

**[VISUAL: Code editor showing blocking code]**

**Presenter:**

"The number one mistake in async programming is accidentally blocking the event loop. Let me show you what this looks like:

```python
import time
import asyncio

async def bad_sleep():
    print('Starting sleep...')
    time.sleep(2)  # WRONG!
    print('Done sleeping')
    return 'Result'

async def main():
    await asyncio.gather(
        bad_sleep(),
        bad_sleep(),
        bad_sleep()
    )

asyncio.run(main())
```

**[VISUAL: Run the code, show timing]**

This takes 6 seconds—exactly the time it would take to run sequentially. Why? Because time.sleep() blocks the entire event loop. While one task is sleeping, no other tasks can run. You've completely defeated the purpose of async.

**[VISUAL: Diagram showing blocked event loop]**

When you use a blocking operation like time.sleep(), the event loop stops. It can't switch to other tasks. Everything waits.

Now let's fix it:

```python
async def good_sleep():
    print('Starting sleep...')
    await asyncio.sleep(2)  # CORRECT!
    print('Done sleeping')
    return 'Result'

async def main():
    await asyncio.gather(
        good_sleep(),
        good_sleep(),
        good_sleep()
    )
```

**[VISUAL: Run the code, show timing]**

This takes 2 seconds total. All three tasks run concurrently because asyncio.sleep() yields control to the event loop at the await point.

**[VISUAL: List of blocking operations]**

Here are the most common blocking operations to avoid:

- time.sleep() → Use await asyncio.sleep()
- requests.get() → Use aiohttp
- open() / f.read() → Use aiofiles
- Regular socket operations → Use asyncio sockets
- subprocess.run() → Use asyncio.create_subprocess_exec()

The rule is simple: if you can't await it, and it does I/O, it's probably blocking.

**[VISUAL: Code editor showing file I/O example]**

Even 'fast' operations like file reads block:

```python
# ❌ BAD: Blocks the loop
async def process_file_bad(path):
    with open(path) as f:
        data = f.read()
    return data

# ✅ GOOD: Doesn't block
async def process_file_good(path):
    async with aiofiles.open(path) as f:
        data = await f.read()
    return data
```

Always use async-compatible libraries for ALL I/O operations, even if they seem fast."

---

## Section 3: Pitfall #2 - CPU-Bound Work in Async (3 minutes)

**[VISUAL: Code editor showing CPU-bound example]**

**Presenter:**

"Another common mistake is using async for CPU-bound work. Let me show you:

```python
async def calculate():
    # Pure computation, no I/O
    result = sum(range(10_000_000))
    return result

async def main():
    results = await asyncio.gather(
        calculate(),
        calculate(),
        calculate()
    )
```

**[VISUAL: Run the code, show timing]**

This runs sequentially, not concurrently. Why? Because there are no await points in calculate(). It's pure computation with no I/O. The event loop can't switch tasks because the calculation never yields control.

**[VISUAL: Comparison diagram]**

Remember: async is for I/O-bound work, not CPU-bound work.

I/O-bound: Waiting for network, files, databases
CPU-bound: Computing, calculating, processing

For CPU-bound work, use multiprocessing or run in a thread pool:

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

def cpu_bound_task():
    return sum(range(10_000_000))

async def main():
    loop = asyncio.get_event_loop()
    with ProcessPoolExecutor() as executor:
        results = await asyncio.gather(
            loop.run_in_executor(executor, cpu_bound_task),
            loop.run_in_executor(executor, cpu_bound_task),
            loop.run_in_executor(executor, cpu_bound_task)
        )
```

Now the calculations actually run in parallel on multiple CPU cores.

**[VISUAL: Decision tree]**

Quick decision guide:
- Does it wait for I/O? → Use async
- Does it compute intensively? → Use multiprocessing
- Is it a single quick operation? → Just use sync code"

---

## Section 4: Pitfall #3 - Race Conditions (3 minutes)

**[VISUAL: Code editor showing race condition]**

**Presenter:**

"When multiple tasks access shared state, you can get race conditions. This is a classic problem:

```python
counter = 0

async def increment():
    global counter
    temp = counter
    await asyncio.sleep(0.01)  # Task switch can happen here
    counter = temp + 1

async def main():
    await asyncio.gather(*[increment() for _ in range(100)])
    print(f'Counter: {counter}')

asyncio.run(main())
```

**[VISUAL: Run the code multiple times, show varying results]**

The counter is wrong! Sometimes 50, sometimes 30, never 100. This is a race condition.

**[VISUAL: Timeline diagram showing the race]**

Here's what happens: Task 1 reads counter as 0. Task 2 also reads counter as 0 (it hasn't been updated yet). Task 1 increments to 1 and writes back. Task 2 also increments to 1 and writes back, overwriting Task 1's update. Result: 1 instead of 2.

The solution is asyncio.Lock:

```python
lock = asyncio.Lock()
counter = 0

async def increment():
    global counter
    async with lock:
        temp = counter
        await asyncio.sleep(0.01)
        counter = temp + 1

async def main():
    await asyncio.gather(*[increment() for _ in range(100)])
    print(f'Counter: {counter}')
```

**[VISUAL: Run the code, show correct result]**

Now it's 100 every time. The lock ensures only one task accesses the counter at a time.

**[VISUAL: Highlight rule]**

Rule: Always protect shared mutable state with asyncio.Lock."

---

## Section 5: Pitfall #4 - Deadlocks (3 minutes)

**[VISUAL: Code editor showing deadlock scenario]**

**Presenter:**

"When you use multiple locks, you can create deadlocks. Here's the classic scenario:

```python
lock1 = asyncio.Lock()
lock2 = asyncio.Lock()

async def task_a():
    async with lock1:
        print('Task A has lock1')
        await asyncio.sleep(0.1)
        async with lock2:
            print('Task A has both locks')

async def task_b():
    async with lock2:
        print('Task B has lock2')
        await asyncio.sleep(0.1)
        async with lock1:
            print('Task B has both locks')

asyncio.run(asyncio.gather(task_a(), task_b()))
```

**[VISUAL: Run the code, show it hanging]**

The program hangs forever. This is a deadlock.

**[VISUAL: Diagram showing deadlock cycle]**

Task A holds lock1 and waits for lock2.
Task B holds lock2 and waits for lock1.
Both tasks wait forever.

The solution: always acquire locks in the same order:

```python
async def task_a():
    async with lock1:
        async with lock2:  # Always lock1 then lock2
            pass

async def task_b():
    async with lock1:  # Same order!
        async with lock2:
            pass
```

**[VISUAL: Run the code, show it completing]**

Now it works. Both tasks acquire locks in the same order, so no circular waiting.

**[VISUAL: Best practice slide]**

Best practices to prevent deadlocks:
1. Always acquire locks in the same order
2. Use a single lock if possible
3. Keep lock duration short
4. Use timeouts when acquiring locks"

---

## Section 6: Pitfall #5 - Unhandled Exceptions (3 minutes)

**[VISUAL: Code editor showing fire-and-forget task]**

**Presenter:**

"When you create a task but never await it, exceptions can be silently lost. This is dangerous:

```python
async def risky_task():
    await asyncio.sleep(1)
    raise ValueError('Something went wrong!')

async def main():
    task = asyncio.create_task(risky_task())
    # Do other work
    await asyncio.sleep(2)
    print('Done')

asyncio.run(main())
```

**[VISUAL: Run the code, show output]**

The task fails, but we never see the error. The exception is stored in the task, but never retrieved. This is called a 'fire and forget' task, and it's a common source of hidden bugs.

**[VISUAL: Code editor showing solutions]**

Solution 1: Always await tasks:

```python
async def main():
    task = asyncio.create_task(risky_task())
    try:
        await task
    except ValueError as e:
        print(f'Task failed: {e}')
```

Solution 2: Use a done callback:

```python
def handle_exception(task):
    try:
        task.result()
    except Exception as e:
        print(f'Task failed: {e}')

async def main():
    task = asyncio.create_task(risky_task())
    task.add_done_callback(handle_exception)
    await asyncio.sleep(2)
```

**[VISUAL: Run both examples, show error handling]**

Now exceptions are caught and handled properly.

Rule: Never create a task without either awaiting it or adding error handling."

---

## Section 7: Pitfall #6 - Resource Leaks (2 minutes)

**[VISUAL: Code editor showing resource leak]**

**Presenter:**

"Forgetting to clean up resources is another common mistake:

```python
# ❌ BAD: Session leaked!
async def fetch_data(url):
    session = aiohttp.ClientSession()
    response = await session.get(url)
    return await response.text()
# Session never closed!
```

Every call to fetch_data() creates a session but never closes it. These accumulate and waste resources.

**[VISUAL: Code editor showing fix]**

Always use context managers:

```python
# ✅ GOOD: Session automatically closed
async def fetch_data(url):
    async with aiohttp.ClientSession() as session:
        response = await session.get(url)
        return await response.text()
```

**[VISUAL: List of resources to manage]**

Resources that need cleanup:
- HTTP sessions (aiohttp.ClientSession)
- Database connections
- File handles
- Locks (though usually in a with block)
- Subprocesses

Use 'async with' for all async resources. It guarantees cleanup even if exceptions occur."

---

## Section 8: Pitfall #7 - Multiple asyncio.run() (2 minutes)

**[VISUAL: Code editor showing the mistake]**

**Presenter:**

"A common beginner mistake is calling asyncio.run() multiple times:

```python
async def task1():
    return 'Result 1'

async def task2():
    return 'Result 2'

result1 = asyncio.run(task1())  # OK
result2 = asyncio.run(task2())  # ERROR!
```

**[VISUAL: Run the code, show error]**

RuntimeError: asyncio.run() creates an event loop, runs your code, then closes the loop. You can't reuse a closed loop.

**[VISUAL: Code editor showing fix]**

Solution: Create a single async main() function:

```python
async def main():
    result1 = await task1()
    result2 = await task2()
    return result1, result2

results = asyncio.run(main())  # Single entry point
```

This is the standard pattern: one asyncio.run() call at the program entry point, everything else is async functions that await each other."

---

## Section 9: Pitfall #8 - Not Canceling Tasks (2 minutes)

**[VISUAL: Code editor showing uncanceled tasks]**

**Presenter:**

"When you use timeouts, don't forget to cancel pending tasks:

```python
# ❌ BAD: Tasks keep running!
async def main():
    tasks = [asyncio.create_task(long_task()) for _ in range(10)]

    try:
        await asyncio.wait_for(
            asyncio.gather(*tasks),
            timeout=5.0
        )
    except asyncio.TimeoutError:
        print('Timeout!')
        return

# Tasks still running in background, wasting resources!
```

**[VISUAL: Code editor showing proper cleanup]**

Always cancel tasks you don't need:

```python
# ✅ GOOD: Clean up tasks
async def main():
    tasks = [asyncio.create_task(long_task()) for _ in range(10)]

    try:
        await asyncio.wait_for(
            asyncio.gather(*tasks),
            timeout=5.0
        )
    except asyncio.TimeoutError:
        for task in tasks:
            task.cancel()
        await asyncio.gather(*tasks, return_exceptions=True)
        print('All tasks cancelled')
```

This ensures resources are properly freed and tasks don't run forever."

---

## Section 10: Debugging Async Code (3 minutes)

**[VISUAL: Code editor with debug mode enabled]**

**Presenter:**

"Debugging async code is harder than sync code, but Python provides tools to help. The most important is debug mode:

```python
asyncio.run(main(), debug=True)
```

**[VISUAL: Terminal showing debug warnings]**

Debug mode shows you:

1. 'coroutine was never awaited' - You forgot to await an async function
2. 'Task was destroyed but it is pending!' - Task not finished when program ended
3. 'Executing took X.XX seconds' - Something is taking too long (possibly blocking)

Let me show you a common warning:

```python
async def forgotten_task():
    await asyncio.sleep(1)
    return 42

async def main():
    task = forgotten_task()  # Forgot await!
    print('Done')

asyncio.run(main(), debug=True)
```

**[VISUAL: Run the code, show warning]**

The warning 'coroutine was never awaited' tells you exactly what's wrong.

**[VISUAL: Code editor showing debugging techniques]**

Other debugging techniques:

Check task status:
```python
print(f'Done: {task.done()}')
print(f'Cancelled: {task.cancelled()}')
```

List all running tasks:
```python
tasks = asyncio.all_tasks()
for task in tasks:
    print(task.get_coro())
```

Enable asyncio logging:
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

**[VISUAL: Highlight recommendation]**

Always enable debug mode during development. It catches many mistakes early."

---

## Section 11: Best Practices Checklist (2 minutes)

**[VISUAL: Checklist slide]**

**Presenter:**

"Before we wrap up, let me give you a checklist to use when writing async code:

**Before deploying, verify:**

1. No blocking operations (time.sleep, requests, open)
2. All coroutines are awaited
3. Shared state protected by locks
4. Locks acquired in consistent order
5. Tasks cancelled on timeout/error
6. Resources cleaned up with context managers
7. Exceptions handled in background tasks
8. No CPU-bound work in async functions
9. Single asyncio.run() entry point
10. Debug mode used during development

**[VISUAL: Code review example]**

Let me show you how to apply this checklist to real code:

```python
# Before review
async def process_documents(paths):
    results = []
    for path in paths:
        with open(path) as f:  # ❌ Blocking I/O
            data = f.read()
        results.append(data)
    return results
```

Checklist items 1 and 6 fail. Let's fix:

```python
# After review
async def process_documents(paths):
    async def process_one(path):
        async with aiofiles.open(path) as f:  # ✅ Non-blocking, context manager
            return await f.read()

    return await asyncio.gather(
        *[process_one(p) for p in paths],
        return_exceptions=True  # ✅ Handle exceptions
    )
```

Now it passes all checklist items."

---

## Section 12: Summary and Next Steps (1 minute)

**[VISUAL: Key takeaways slide]**

**Presenter:**

"Let's recap the key pitfalls we covered:

✅ Never block the event loop—use async libraries
✅ Don't use async for CPU-bound work
✅ Protect shared state with locks
✅ Prevent deadlocks by acquiring locks in order
✅ Handle exceptions in background tasks
✅ Clean up resources with context managers
✅ Cancel unused tasks
✅ Call asyncio.run() only once
✅ Enable debug mode during development

**[VISUAL: Next module preview]**

This completes our async programming series! You now understand:
- How async/await works (Module 1.1.2.1)
- Concurrent execution patterns (Module 1.1.2.2)
- Common pitfalls and how to avoid them (this module)

In the next module, we'll move on to Environment Management with .env files, where you'll learn to handle configuration and secrets securely.

**[VISUAL: Lab assignment slide]**

For now, complete the hands-on lab where you'll fix broken async code, identify blocking operations, and add proper error handling. This will solidify everything we've covered in all three async modules.

Async programming is powerful but requires discipline. Follow these best practices, avoid these pitfalls, and you'll build fast, reliable, maintainable UiPath agents.

Thank you for watching, and I'll see you in the next module!"

---

## Presenter Notes

### Key Teaching Points
1. **Start with the worst offender**: Blocking the event loop is #1
2. **Show real examples**: Use actual code that breaks, not toy examples
3. **Demonstrate consequences**: Show timing, show hanging code, show wrong results
4. **Provide fixes**: Always show the wrong way then the right way
5. **Give practical checklist**: Something they can actually use

### Common Questions to Address
- "How do I know if a library blocks?" → Check documentation, or look for async version
- "Is it always bad to block briefly?" → Yes! Even brief blocking compounds across tasks
- "Can I use threading instead?" → Yes for blocking libraries, but async is usually better
- "How do I debug hung async code?" → Use timeout, enable debug mode, check task status

### Demo Tips
- Actually run broken code and show it failing
- Show debug warnings in action
- Use timing measurements to demonstrate blocking
- Show resource monitor for leaks if possible

### Troubleshooting Common Issues
- If code doesn't show warning: Make sure debug=True
- If deadlock doesn't reproduce: Increase sleep times to make it more obvious
- If race condition always works: Increase iteration count until it fails

### Time Management
- Section 1-2: Blocking operations (7 minutes)
- Section 3-5: Synchronization issues (9 minutes)
- Section 6-8: Resource and task management (6 minutes)
- Section 9-12: Debugging and best practices (4 minutes)

Total: 26 minutes

### Code Examples to Prepare
- Have broken code ready to demonstrate failures
- Test all fixes before recording
- Prepare timing comparisons
- Have debug output examples ready

### Key Warnings to Emphasize
1. "Never use synchronous I/O in async code"
2. "Always protect shared state"
3. "Handle exceptions in background tasks"
4. "Enable debug mode during development"
