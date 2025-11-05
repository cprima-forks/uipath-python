# Video Script: Concurrent Execution Patterns

**Module:** 1.1.2.2
**Duration:** 32 minutes
**Target Audience:** UiPath Agent Developers (Intermediate)
**Prerequisites:** Module 1.1.2.1 (Understanding Async/Await)

---

## Section 1: Introduction (2 minutes)

**[VISUAL: Title slide with module objectives]**

**Presenter:**

"Welcome to Module 1.1.2.2: Concurrent Execution Patterns. In the previous module, you learned the fundamentals of async/await—how to define coroutines, use the await keyword, and understand the event loop.

Now we're going to take it to the next level. In this module, we'll explore the patterns and tools you need to build real-world concurrent applications.

We'll cover:
- How to run multiple coroutines concurrently using asyncio.gather()
- Task management strategies with create_task()
- Timeout handling and cancellation
- Coordination patterns using Queue, Semaphore, and Lock

By the end of this session, you'll be able to build robust, concurrent UiPath agents that handle multiple operations efficiently and safely.

Let's start with the most fundamental pattern: running multiple coroutines concurrently."

**[VISUAL: Transition to code editor]**

---

## Section 2: asyncio.gather() Basics (4 minutes)

**[VISUAL: Code editor showing gather example]**

**Presenter:**

"The most common way to run multiple coroutines concurrently is asyncio.gather(). Let's see it in action:

```python
import asyncio

async def fetch_user(user_id):
    await asyncio.sleep(1)
    return f'User {user_id}'

async def fetch_orders(user_id):
    await asyncio.sleep(1)
    return f'Orders for {user_id}'

async def fetch_profile(user_id):
    await asyncio.sleep(1)
    return f'Profile for {user_id}'

async def main():
    user, orders, profile = await asyncio.gather(
        fetch_user(123),
        fetch_orders(123),
        fetch_profile(123)
    )
    print(user, orders, profile)

asyncio.run(main())
```

**[VISUAL: Run the code, show timing]**

Notice the timing: this completes in about 1 second, not 3 seconds. All three operations run concurrently.

**[VISUAL: Timeline diagram showing concurrent execution]**

gather() does several important things:

First, it runs all the coroutines concurrently. They're all active at the same time.

Second, it waits for all of them to complete before returning.

Third, and this is crucial: it returns the results in the order you passed the coroutines, NOT in completion order.

**[VISUAL: Highlight the order of results]**

So even if fetch_profile() finishes first, the results will always be in this order: user, orders, profile. This makes it easy to unpack results as we did here.

**[VISUAL: Code editor showing list comprehension with gather]**

You can also use gather() with a list of coroutines:

```python
urls = ['url1', 'url2', 'url3', ...]

tasks = [fetch_url(url) for url in urls]
results = await asyncio.gather(*tasks)
```

Notice the asterisk—that unpacks the list. This pattern is very common when you have a dynamic list of operations to perform."

---

## Section 3: Exception Handling in gather() (4 minutes)

**[VISUAL: Code editor showing exception handling]**

**Presenter:**

"Now let's talk about a critical aspect: exception handling in gather(). By default, if any coroutine raises an exception, gather() immediately raises that exception:

```python
async def risky_task(n):
    if n == 2:
        raise ValueError(f'Task {n} failed!')
    await asyncio.sleep(1)
    return f'Result {n}'

async def main():
    try:
        results = await asyncio.gather(
            risky_task(1),
            risky_task(2),
            risky_task(3)
        )
    except ValueError as e:
        print(f'Error: {e}')
```

**[VISUAL: Run the code, show output]**

When task 2 raises an exception, gather() immediately raises it to the caller. But here's the important part: the other tasks continue running in the background. You just can't get their results anymore.

**[VISUAL: Diagram showing tasks continuing after exception]**

This behavior is often not what you want. You typically want to handle failures gracefully without losing the successful results.

The solution is return_exceptions=True:

```python
results = await asyncio.gather(
    risky_task(1),
    risky_task(2),
    risky_task(3),
    return_exceptions=True
)

for i, result in enumerate(results):
    if isinstance(result, Exception):
        print(f'Task {i} failed: {result}')
    else:
        print(f'Task {i} succeeded: {result}')
```

**[VISUAL: Run the code, show output]**

Now exceptions are returned as results instead of raised. You can inspect each result and decide how to handle failures.

**[VISUAL: Highlight the output showing success and failure]**

This is the pattern I recommend for production code: use return_exceptions=True and handle each result individually. This makes your agent resilient to partial failures."

---

## Section 4: Task Management with create_task() (3 minutes)

**[VISUAL: Code editor showing create_task()]**

**Presenter:**

"gather() is great for simple cases, but sometimes you need more control. That's where asyncio.create_task() comes in.

```python
async def main():
    # Create tasks—they start immediately
    task1 = asyncio.create_task(fetch_data('api1'))
    task2 = asyncio.create_task(fetch_data('api2'))

    print('Tasks are now running in background')

    # We can do other work here
    await asyncio.sleep(0.5)
    print('Still doing other work...')

    # Collect results when we need them
    result1 = await task1
    result2 = await task2
```

**[VISUAL: Run the code, show output]**

The key difference: create_task() returns a Task object and starts execution immediately. gather() doesn't start execution until you await it.

**[VISUAL: Timeline comparison diagram]**

Tasks give you more control. You can:
- Cancel them: task.cancel()
- Check their status: task.done()
- Get their result: task.result()
- Check for exceptions: task.exception()

Let me show you an example:

```python
task = asyncio.create_task(long_operation())

# Check if it's done
if task.done():
    result = task.result()
else:
    print('Still running...')
```

**[VISUAL: Show code execution]**

Use create_task() when you want to start tasks early, do other work, then collect results later. Use gather() when you just want to start everything and wait for all results."

---

## Section 5: Timeouts and Cancellation (4 minutes)

**[VISUAL: Code editor showing asyncio.wait_for()]**

**Presenter:**

"In production, you can't wait forever. Network requests timeout, APIs become unresponsive, operations hang. You need timeout handling.

The primary tool is asyncio.wait_for():

```python
async def slow_api_call():
    await asyncio.sleep(10)
    return 'Data'

async def main():
    try:
        result = await asyncio.wait_for(
            slow_api_call(),
            timeout=5.0
        )
        print(result)
    except asyncio.TimeoutError:
        print('Operation timed out!')
```

**[VISUAL: Run the code, show timeout]**

wait_for() waits up to 5 seconds. If the operation doesn't complete in time, it raises asyncio.TimeoutError. Importantly, it also cancels the underlying task automatically.

**[VISUAL: Python 3.11+ context manager syntax]**

In Python 3.11 and later, there's a cleaner syntax using asyncio.timeout():

```python
try:
    async with asyncio.timeout(5.0):
        result = await slow_api_call()
        print(result)
except asyncio.TimeoutError:
    print('Timed out!')
```

This context manager approach is cleaner and can wrap multiple operations:

```python
async with asyncio.timeout(10.0):
    await operation1()
    await operation2()
    await operation3()
```

All three operations are subject to the 10-second total timeout.

**[VISUAL: Code editor showing task cancellation]**

You can also manually cancel tasks:

```python
async def main():
    task = asyncio.create_task(slow_operation())

    await asyncio.sleep(1)  # Let it run briefly

    task.cancel()  # Request cancellation

    try:
        await task
    except asyncio.CancelledError:
        print('Task was cancelled')
```

**[VISUAL: Diagram showing cancellation flow]**

When you call task.cancel(), the task doesn't stop immediately. Instead, CancelledError is raised at the next await point in the task. The task can catch this exception to perform cleanup:

```python
async def slow_operation():
    try:
        await asyncio.sleep(10)
        return 'Done'
    except asyncio.CancelledError:
        print('Cleaning up before cancellation...')
        # Cleanup code here
        raise  # Re-raise to complete cancellation
```

This cooperative cancellation model ensures resources are cleaned up properly."

---

## Section 6: asyncio.Queue for Coordination (4 minutes)

**[VISUAL: Code editor showing Queue example]**

**Presenter:**

"When you have producers creating work and consumers processing it, you need coordination. That's where asyncio.Queue comes in.

Here's the classic producer-consumer pattern:

```python
async def producer(queue):
    for i in range(5):
        await asyncio.sleep(0.5)
        await queue.put(f'Item {i}')
        print(f'Produced Item {i}')
    await queue.put(None)  # Signal completion

async def consumer(queue):
    while True:
        item = await queue.get()
        if item is None:
            break
        print(f'Processing {item}')
        await asyncio.sleep(1)
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    await asyncio.gather(producer(queue), consumer(queue))
```

**[VISUAL: Run the code, show output]**

Watch the interleaving: the producer adds items while the consumer processes them. The queue coordinates the work.

**[VISUAL: Diagram showing producer-consumer flow]**

Key Queue methods:
- await queue.put(item) - Add item (waits if queue is full)
- item = await queue.get() - Remove item (waits if queue is empty)
- queue.task_done() - Mark item as processed
- await queue.join() - Wait until all items are processed

**[VISUAL: Code editor showing web scraper with queue]**

Here's a realistic example: a web scraper with multiple workers:

```python
async def worker(queue, session, worker_id):
    while True:
        url = await queue.get()
        if url is None:
            break

        try:
            html = await fetch_page(session, url)
            print(f'Worker {worker_id} fetched {url}')
        finally:
            queue.task_done()

async def main():
    queue = asyncio.Queue()

    # Add URLs to queue
    for url in urls:
        await queue.put(url)

    # Create 5 workers
    workers = [
        asyncio.create_task(worker(queue, session, i))
        for i in range(5)
    ]

    await queue.join()  # Wait for all URLs to be processed

    # Stop workers
    for _ in workers:
        await queue.put(None)
    await asyncio.gather(*workers)
```

**[VISUAL: Highlight the worker pool pattern]**

This pattern scales beautifully. You can easily adjust the number of workers based on your needs."

---

## Section 7: Limiting Concurrency with Semaphore (3 minutes)

**[VISUAL: Code editor showing Semaphore]**

**Presenter:**

"Sometimes you can't run unlimited concurrent operations. Maybe an API has rate limits, or you want to control resource usage. That's where asyncio.Semaphore comes in.

A semaphore is like a bouncer at a club: only N tasks can enter at once.

```python
# Allow only 5 concurrent operations
semaphore = asyncio.Semaphore(5)

async def fetch_with_limit(url):
    async with semaphore:
        # Only 5 tasks can be here simultaneously
        return await fetch_url(url)

async def main():
    urls = [f'https://api.example.com/page/{i}' for i in range(100)]

    # Creates 100 tasks, but only 5 run concurrently
    results = await asyncio.gather(
        *[fetch_with_limit(url) for url in urls]
    )
```

**[VISUAL: Diagram showing semaphore limiting concurrency]**

This code creates 100 tasks, but the semaphore ensures only 5 are actually making requests at any given time. The others wait their turn.

**[VISUAL: Comparison diagram]**

This is perfect for API rate limiting. If an API allows 10 requests per second, use Semaphore(10).

Let's see a real-world example:

```python
class RateLimitedClient:
    def __init__(self, rate_limit=10):
        self.semaphore = asyncio.Semaphore(rate_limit)
        self.session = aiohttp.ClientSession()

    async def fetch(self, url):
        async with self.semaphore:
            async with self.session.get(url) as response:
                return await response.text()

    async def close(self):
        await self.session.close()
```

**[VISUAL: Show usage]**

This client automatically rate-limits all requests. Simple and effective."

---

## Section 8: Mutual Exclusion with Lock (3 minutes)

**[VISUAL: Code editor showing race condition]**

**Presenter:**

"When multiple tasks access shared state, you can get race conditions. Let me show you the problem:

```python
counter = 0

async def increment():
    global counter
    current = counter
    await asyncio.sleep(0.01)  # Simulate some work
    counter = current + 1

async def main():
    await asyncio.gather(*[increment() for _ in range(100)])
    print(f'Counter: {counter}')  # Should be 100, but isn't!
```

**[VISUAL: Run the code, show incorrect result]**

The counter is wrong! Why? Because multiple tasks read the same value, all increment it, then all write back. They overwrite each other.

**[VISUAL: Diagram showing race condition]**

The solution is asyncio.Lock:

```python
lock = asyncio.Lock()
counter = 0

async def increment():
    global counter
    async with lock:
        # Only one task can be here at a time
        current = counter
        await asyncio.sleep(0.01)
        counter = current + 1

async def main():
    await asyncio.gather(*[increment() for _ in range(100)])
    print(f'Counter: {counter}')  # Now it's 100!
```

**[VISUAL: Run the code, show correct result]**

The lock ensures only one task accesses the counter at a time. This is mutual exclusion.

**[VISUAL: Comparison table]**

When to use Lock vs Semaphore:
- Lock: Only 1 task at a time (mutual exclusion)
- Semaphore: Up to N tasks (rate limiting)

Use Lock to protect shared state. Use Semaphore to limit concurrency."

---

## Section 9: Other Coordination Primitives (2 minutes)

**[VISUAL: Code editor showing Event]**

**Presenter:**

"asyncio provides other coordination tools. Let me quickly show you asyncio.Event:

```python
event = asyncio.Event()

async def waiter():
    print('Waiting for signal...')
    await event.wait()  # Block until event is set
    print('Signal received!')

async def setter():
    await asyncio.sleep(2)
    print('Sending signal')
    event.set()  # Wake up all waiters

async def main():
    await asyncio.gather(waiter(), setter())
```

**[VISUAL: Run the code]**

Events are useful for signaling between tasks. One task waits, another task signals.

**[VISUAL: Quick reference slide]**

Quick summary of coordination primitives:
- Queue: Producer-consumer coordination
- Semaphore: Limit to N concurrent tasks
- Lock: Mutual exclusion (1 task)
- Event: Signal between tasks

Each has specific use cases. Choose the right tool for your needs."

---

## Section 10: Real-World Example - Document Processor (3 minutes)

**[VISUAL: Code editor with complete example]**

**Presenter:**

"Let's put it all together with a realistic example: a concurrent document processor for a UiPath agent.

```python
import aiofiles
import asyncio

async def process_document(doc_path, semaphore):
    async with semaphore:
        print(f'Processing {doc_path}')

        try:
            async with asyncio.timeout(10.0):
                async with aiofiles.open(doc_path) as f:
                    content = await f.read()

                # Simulate processing
                await asyncio.sleep(1)

                return {
                    'path': doc_path,
                    'length': len(content),
                    'status': 'processed'
                }
        except asyncio.TimeoutError:
            return {'path': doc_path, 'status': 'timeout'}
        except Exception as e:
            return {'path': doc_path, 'status': 'error', 'error': str(e)}

async def process_batch(doc_paths, max_concurrent=5):
    semaphore = asyncio.Semaphore(max_concurrent)

    tasks = [
        process_document(path, semaphore)
        for path in doc_paths
    ]

    results = await asyncio.gather(*tasks, return_exceptions=True)

    # Analyze results
    successful = [r for r in results if not isinstance(r, Exception) and r['status'] == 'processed']
    failed = [r for r in results if isinstance(r, Exception) or r['status'] != 'processed']

    print(f'Processed: {len(successful)}, Failed: {len(failed)}')
    return successful
```

**[VISUAL: Highlight key patterns]**

Notice what we're using:
1. Semaphore to limit concurrent file operations
2. Timeout to prevent hanging on slow files
3. gather() with return_exceptions to handle failures gracefully
4. Structured error handling for different failure modes

This is production-ready code. It's resilient, efficient, and handles errors properly."

---

## Section 11: Best Practices and Common Pitfalls (2 minutes)

**[VISUAL: Checklist slide]**

**Presenter:**

"Let me share the most important best practices:

**Always set timeouts:**
Never wait forever. Always use asyncio.timeout() or wait_for() for network operations.

**Handle exceptions in gather():**
Use return_exceptions=True unless you want the first exception to stop everything.

**Limit concurrency:**
Don't create unlimited tasks. Use Semaphore to control concurrency.

**Clean up tasks:**
Cancel pending tasks when you're done with them.

**[VISUAL: Code examples of pitfalls]**

Common pitfalls to avoid:

Pitfall 1: Not limiting concurrency
```python
# BAD: Creates 10,000 concurrent connections
results = await asyncio.gather(*[fetch(url) for url in urls])

# GOOD: Limit concurrency
async with semaphore:
    results = await asyncio.gather(*[fetch(url) for url in urls])
```

Pitfall 2: Not canceling pending tasks
```python
# BAD: Tasks keep running forever
done, pending = await asyncio.wait(tasks, timeout=5.0)

# GOOD: Cancel what you don't need
for task in pending:
    task.cancel()
```

Avoid these mistakes and your async code will be robust and reliable."

---

## Section 12: Summary and Next Steps (2 minutes)

**[VISUAL: Key takeaways slide]**

**Presenter:**

"Let's recap what we've covered:

✅ asyncio.gather() runs coroutines concurrently and returns results in order
✅ asyncio.create_task() gives you control over task lifecycle
✅ asyncio.wait_for() and asyncio.timeout() handle timeouts
✅ asyncio.Queue coordinates producer-consumer patterns
✅ asyncio.Semaphore limits concurrent operations
✅ asyncio.Lock ensures mutual exclusion
✅ Always handle exceptions and set timeouts

**[VISUAL: Pattern selection guide]**

When building concurrent agents:
- Use gather() for simple concurrent execution
- Use Semaphore for rate limiting
- Use Queue for producer-consumer
- Use Lock for shared state
- Always set timeouts
- Always handle exceptions

**[VISUAL: Next module preview]**

In the next module, Common Async Pitfalls, we'll dive into:
- Identifying blocking operations that kill performance
- Avoiding deadlocks and race conditions
- Debugging techniques for async code
- Performance optimization strategies

**[VISUAL: Lab assignment slide]**

For now, complete the hands-on lab where you'll build a concurrent web scraper with rate limiting, timeout handling, and error recovery. This will solidify the patterns we've covered today.

These patterns are essential for building production-grade UiPath agents. Master them, and you'll be able to build agents that are fast, reliable, and scalable.

Thank you for watching, and I'll see you in the next module!"

---

## Presenter Notes

### Key Teaching Points
1. **Start with gather()**: It's the most common pattern, teach it first
2. **Emphasize exception handling**: Show both default behavior and return_exceptions
3. **Demonstrate timeout importance**: Network operations must have timeouts
4. **Show real examples**: Use realistic code, not toy examples
5. **Explain when to use each tool**: Give clear decision criteria

### Common Questions to Address
- "Why use gather() vs create_task()?" → gather() is simpler for most cases, create_task() gives more control
- "How do I know how many workers to use?" → Start with 5-10, measure, adjust based on resources and API limits
- "What if I need to cancel just one task in gather()?" → Use create_task() for individual control
- "Can I use regular Queue instead of asyncio.Queue?" → No! Regular Queue will block the event loop

### Demo Tips
- Use asyncio.sleep() with short times (0.1-0.5s) for demos
- Show timing differences clearly with time.time()
- Run code live to show real behavior
- Use print statements to show execution order and concurrency

### Troubleshooting Common Issues
- If gather() raises unexpected exceptions: Check return_exceptions parameter
- If tasks seem to not run concurrently: Check for blocking operations (time.sleep, requests)
- If timeout doesn't work: Ensure using asyncio.timeout() not time.sleep()
- If semaphore doesn't limit: Check it's created outside the task, passed in

### Time Management
- Section 1-3: gather() and exceptions (10 minutes)
- Section 4-5: Tasks and timeouts (7 minutes)
- Section 6-8: Coordination primitives (10 minutes)
- Section 9-12: Examples and summary (5 minutes)

Total: 32 minutes with buffer for demonstrations

### Code Examples to Prepare
- Have working examples ready to run live
- Test all code before recording
- Prepare timing measurements to show performance
- Have error examples ready to demonstrate failure handling
