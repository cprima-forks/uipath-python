# Video Script: Job Status Monitoring
## Module 2.2.1.2 - SDK Fundamentals

**Duration:** 24 minutes
**Target Audience:** Developers monitoring async job execution
**Prerequisites:** Module 2.2.1.1 (can start jobs)
**Learning Outcomes:** Monitor and wait for job completion

---

## Scene 1: Introduction (1.5 minutes)

### Visual: Title slide

**[On camera - Instructor]**

Welcome to Module 2.2.1.2: Job Status Monitoring. In the last module, we learned how to START jobs. Now we need to know when they FINISH and get the results.

### Visual: Timeline showing async execution

**[Voice over visual]**

Remember, StartJobs returns immediately. The job is created but hasn't run yet. It might take seconds, minutes, or even hours to complete. How do we know when it's done?

### Visual: Learning objectives slide

**[On camera - Instructor]**

In this video, you'll master:
- Polling job status at appropriate intervals
- Understanding job state transitions
- Implementing wait-for-completion logic
- Handling timeouts and errors
- Monitoring multiple jobs efficiently
- Retrieving results

Let's get started!

---

## Scene 2: Job Lifecycle States (3 minutes)

### Visual: State transition diagram

**[On camera - Instructor]**

First, let's understand the journey a job takes from creation to completion.

### Visual: Animated state flow

**[Voice over animation]**

A job starts in **Pending** state - it's been created and is waiting for an available robot to pick it up.

Once a robot is assigned and starts execution, the job moves to **Running** state - your process is actively executing.

From Running, two things can happen:

If everything goes well, the job reaches **Successful** state - process completed without errors.

If an error occurs, the job enters **Faulted** state - something went wrong during execution.

There's also a **Stopped** state if the job is manually canceled, and a transitional **Stopping** state while cancellation happens.

### Visual: State table

**[On camera - Instructor]**

Successful, Faulted, and Stopped are *terminal states* - the job won't change state again. We can stop polling.

Pending, Running, and Stopping are *non-terminal* - the job is still progressing. We need to keep checking.

---

## Scene 3: Getting Job Status (2.5 minutes)

### Visual: API endpoint

**[On camera - Instructor]**

To check a job's status, we use the Jobs endpoint with the job ID.

### Screen recording - Basic status check

**[Voice over screen recording]**

The endpoint uses OData key syntax with parentheses:

```python
job_id = "7d8f9a12-3b4c-5d6e-7f8g-9h0i1j2k3l4m"

response = requests.get(
    f"{ORCHESTRATOR_URL}/odata/Jobs({job_id})",
    headers={
        "Authorization": f"Bearer {token}",
        "X-UIPATH-OrganizationUnitId": str(folder_id)
    }
)

job = response.json()
print(f"State: {job['State']}")
```

### Visual: Response showing job object

**[Voice over]**

The response gives us the complete job object with lots of useful information:
- State: current execution state
- StartTime: when the robot began execution
- EndTime: when it finished (null if still running)
- Info: error details if it faulted
- OutputArguments: results if successful

Let me run this against a real job...

### Visual: Output showing job details

There we go - this job is in Running state, started 30 seconds ago, and hasn't finished yet.

---

## Scene 4: Implementing Polling (4 minutes)

### Visual: Polling concept diagram

**[On camera - Instructor]**

Polling means repeatedly checking status until we reach a terminal state. Let's implement this pattern.

### Screen recording - Basic polling loop

**[Voice over screen recording]**

Here's the simplest version:

```python
import time

def wait_for_job_basic(orchestrator_url, token, folder_id, job_id):
    while True:
        # Get current status
        response = requests.get(
            f"{orchestrator_url}/odata/Jobs({job_id})",
            headers={
                "Authorization": f"Bearer {token}",
                "X-UIPATH-OrganizationUnitId": str(folder_id)
            }
        )

        job = response.json()
        state = job['State']

        print(f"Current state: {state}")

        # Check if done
        if state in ['Successful', 'Faulted', 'Stopped']:
            return job

        # Wait before checking again
        time.sleep(2)
```

### Visual: Highlight the sleep

**[Voice over]**

That sleep(2) is crucial. We wait 2 seconds between checks to avoid hammering the API.

Let me demonstrate this with a real job...

### Visual: Running the function

```python
job = wait_for_job_basic(ORCHESTRATOR_URL, token, folder_id, job_id)
```

### Visual: Output showing polling progress

**[Voice over output]**

```
Current state: Pending
Current state: Pending
Current state: Running
Current state: Running
Current state: Running
Current state: Successful
```

Perfect! We polled every 2 seconds and detected completion immediately.

---

## Scene 5: Adding Timeout (3 minutes)

### Visual: Problem scenario

**[On camera - Instructor]**

But there's a problem with our basic version. What if the job never completes? Maybe no robots are available, or there's a configuration issue. We'd poll forever!

### Screen recording - Adding timeout

**[Voice over screen recording]**

Let's add a timeout:

```python
def wait_for_job(
    orchestrator_url,
    token,
    folder_id,
    job_id,
    poll_interval=2,
    timeout=300  # 5 minutes
):
    start_time = time.time()

    while True:
        # Check if we've exceeded timeout
        elapsed = time.time() - start_time
        if elapsed > timeout:
            raise TimeoutError(
                f"Job {job_id} did not complete within {timeout}s"
            )

        # Get status
        response = requests.get(
            f"{orchestrator_url}/odata/Jobs({job_id})",
            headers={
                "Authorization": f"Bearer {token}",
                "X-UIPATH-OrganizationUnitId": str(folder_id)
            }
        )

        job = response.json()
        state = job['State']

        print(f"[{elapsed:.0f}s] State: {state}")

        # Terminal state check
        if state in ['Successful', 'Faulted', 'Stopped']:
            return job

        time.sleep(poll_interval)
```

### Visual: Demonstrating timeout

**[Voice over]**

Let me test this with a job that won't complete:

```python
try:
    job = wait_for_job(..., timeout=10)  # 10 second timeout
except TimeoutError as e:
    print(f"Timeout! {e}")
```

### Visual: Output showing timeout

After 10 seconds... TimeoutError! Our function properly detected that the job didn't finish in time.

---

## Scene 6: Handling Different Final States (3 minutes)

### Visual: State handling flowchart

**[On camera - Instructor]**

Not all completions are successful. Let's handle each terminal state appropriately.

### Screen recording - Enhanced handling

**[Voice over screen recording]**

```python
def wait_for_job_enhanced(
    orchestrator_url,
    token,
    folder_id,
    job_id,
    timeout=300
):
    start_time = time.time()

    while True:
        if time.time() - start_time > timeout:
            raise TimeoutError("Job timeout")

        job = get_job_status(orchestrator_url, token, folder_id, job_id)
        state = job['State']

        # Handle each terminal state differently
        if state == 'Successful':
            print("✓ Job completed successfully!")
            return job

        elif state == 'Faulted':
            error_info = job.get('Info', 'Unknown error')
            print(f"✗ Job failed: {error_info}")
            raise RuntimeError(f"Job execution failed: {error_info}")

        elif state == 'Stopped':
            print("⚠ Job was stopped manually")
            return job

        time.sleep(2)
```

### Visual: Demonstrating each outcome

**[Voice over]**

Let me show you each scenario:

**Successful job:**
```
✓ Job completed successfully!
```

**Faulted job:**
```
✗ Job failed: Object reference not set to an instance of an object.
RuntimeError: Job execution failed
```

**Stopped job:**
```
⚠ Job was stopped manually
```

Each gets appropriate handling!

---

## Scene 7: Monitoring Multiple Jobs (3 minutes)

### Visual: Multiple jobs diagram

**[On camera - Instructor]**

What if you start 5 jobs and need to wait for all of them? You could wait for each sequentially, but that's inefficient. Let me show you a better way.

### Screen recording - Concurrent monitoring

**[Voice over screen recording]**

```python
def wait_for_all_jobs(
    orchestrator_url,
    token,
    folder_id,
    job_ids,
    timeout=300
):
    """Wait for multiple jobs concurrently"""

    start_time = time.time()
    pending_jobs = set(job_ids)  # Jobs we're still waiting for
    results = {}

    while pending_jobs:
        if time.time() - start_time > timeout:
            raise TimeoutError(f"{len(pending_jobs)} jobs still pending")

        # Check ALL pending jobs on each iteration
        for job_id in list(pending_jobs):
            job = get_job_status(orchestrator_url, token, folder_id, job_id)

            # If terminal state, remove from pending
            if job['State'] in ['Successful', 'Faulted', 'Stopped']:
                pending_jobs.remove(job_id)
                results[job_id] = job
                print(f"✓ Job {job_id[:8]}... finished: {job['State']}")

        # Wait before next iteration
        if pending_jobs:
            time.sleep(2)

    return results
```

### Visual: Running with multiple jobs

**[Voice over]**

Let's test with 3 jobs:

```python
# Start 3 jobs
jobs = start_process_job(..., robot_count=3)
job_ids = [j['Key'] for j in jobs]

# Wait for all
results = wait_for_all_jobs(ORCHESTRATOR_URL, token, folder_id, job_ids)

# Analyze
successful = sum(1 for j in results.values() if j['State'] == 'Successful')
print(f"✓ {successful}/{len(job_ids)} jobs successful")
```

### Visual: Output showing concurrent completion

```
✓ Job 7d8f9a12... finished: Successful
✓ Job 3b4c5d6e... finished: Successful
✓ Job 9h0i1j2k... finished: Successful
✓ 3/3 jobs successful
```

All three monitored in parallel!

---

## Scene 8: Retrieving Output Arguments (2 minutes)

### Visual: Output arguments concept

**[On camera - Instructor]**

When a job succeeds, you often want the results - the output arguments from the process.

### Screen recording - Parsing outputs

**[Voice over screen recording]**

```python
job = wait_for_job(...)

if job['State'] == 'Successful':
    # OutputArguments is a JSON string
    output_json = job.get('OutputArguments')

    if output_json:
        outputs = json.loads(output_json)

        # Access specific outputs
        count = outputs.get('out_ProcessedCount')
        message = outputs.get('out_Message')
        data = outputs.get('out_ResultData')

        print(f"Processed {count} items")
        print(f"Message: {message}")
        print(f"Data: {data}")
```

### Visual: Actual output display

**[Voice over]**

Here's the output from a real job:

```
Processed 15 items
Message: Processing completed successfully
Data: {'total': 15, 'failed': 0}
```

Perfect! We've extracted structured results from the job execution.

---

## Scene 9: Best Practices (2 minutes)

### Visual: Best practices checklist

**[On camera - Instructor]**

Let me share key best practices I've learned from production systems.

### Visual: Practices appearing

**[Voice over]**

**1. Poll at reasonable intervals** - 2-5 seconds is the sweet spot. Faster wastes API calls, slower delays detection.

**2. Always set timeouts** - Never poll indefinitely. Jobs can get stuck.

**3. Log state changes** - Not every poll, just when state changes. Helps with debugging.

**4. Handle all terminal states** - Successful, Faulted, AND Stopped all need appropriate handling.

**5. Implement exponential backoff for long jobs** - Start with 2 seconds, gradually increase to 30 seconds for jobs that take hours.

**6. Cache token** - Don't get a new OAuth token for every status check!

---

## Scene 10: Complete Example (2 minutes)

### Screen recording - End-to-end

**[On camera - Instructor]**

Let me put it all together with a complete, production-ready example.

**[Voice over screen recording]**

```python
from config import OrchestratorConfig
from auth import TokenManager
from jobs import start_process_job, wait_for_job

# Setup
config = OrchestratorConfig()
token_mgr = TokenManager(config.client_id, config.client_secret)

# Authenticate
token = token_mgr.get_token()

# Start job
print("Starting job...")
jobs = start_process_job(
    config.orchestrator_url,
    token,
    config.folder_id,
    config.release_key,
    input_args={"FilePath": "/data/input.xlsx"}
)

job_id = jobs[0]['Key']
print(f"Job {job_id[:8]}... started")

# Monitor to completion
try:
    print("Monitoring job...")
    job = wait_for_job(
        config.orchestrator_url,
        token,
        config.folder_id,
        job_id,
        timeout=600
    )

    # Handle success
    if job['State'] == 'Successful':
        print("✓ Job completed!")

        # Get results
        if job.get('OutputArguments'):
            outputs = json.loads(job['OutputArguments'])
            print(f"Results: {outputs}")

except TimeoutError:
    print("✗ Job timed out after 10 minutes")
except RuntimeError as e:
    print(f"✗ Job failed: {e}")
```

### Visual: Output showing complete execution

There we have it - a complete, robust job execution and monitoring system!

---

## Scene 11: Hands-On Exercise (30 seconds)

### Visual: Exercise instructions

**[On camera - Instructor]**

Now it's your turn! Pause and complete this exercise:

1. Implement wait_for_job with timeout
2. Test with a real job
3. Handle all three terminal states
4. Add logging for state changes
5. Implement wait_for_all_jobs for multiple jobs
6. Parse and display output arguments

Time: 25 minutes. Solution in course materials.

---

## Scene 12: Conclusion (1 minute)

### Visual: Key takeaways

**[On camera - Instructor]**

Excellent work! You can now monitor job execution from start to finish. Let's recap:

**[Voice over]**

- Poll at 2-5 second intervals
- Always implement timeouts
- Handle Successful, Faulted, and Stopped states
- Monitor multiple jobs concurrently for efficiency
- Retrieve output arguments from successful jobs
- Log appropriately for debugging

### Visual: Next module preview

In Module 2.2.1.3, we'll dive deeper into handling job results - processing outputs, downloading artifacts, and advanced error analysis.

### Visual: End card

Thank you for watching! You're making excellent progress through Core Services. See you next time!

---

## Production Notes

**Graphics needed:**
- Job state transition diagram
- Polling concept timeline
- Timeout visualization
- Multiple job monitoring diagram
- Output arguments structure

**Screen recordings:**
- Basic status check
- wait_for_job implementation and execution
- Adding timeout feature
- Handling each terminal state
- Monitoring multiple jobs concurrently
- Parsing output arguments
- Complete end-to-end example

**Code examples:**
- Basic polling loop
- Wait with timeout
- Enhanced state handling
- Multiple job monitoring
- Output argument parsing
- Complete integration

**Callouts/Annotations:**
- Highlight sleep interval
- Mark terminal state checks
- Annotate timeout calculation
- Show state transitions
- Point out OutputArguments parsing

**Pacing notes:**
- Clear explanation of states
- Methodical code building
- Real demonstrations of each scenario
- Smooth flow through patterns
- Practical complete example

**Accessibility:**
- Accurate captions
- Describe all state transitions verbally
- High-contrast code display
- Verbal description of diagrams
