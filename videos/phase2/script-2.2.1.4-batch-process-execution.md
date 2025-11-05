# Video Script: Batch Process Execution
## Module 2.2.1.4 - SDK Fundamentals

**Duration:** 22 minutes
**Target Audience:** Developers implementing high-volume automation
**Prerequisites:** Module 2.2.1.3 (result handling)
**Learning Outcomes:** Execute and manage parallel batch processing

---

## Scene 1: Introduction (1.5 minutes)

### Visual: Title slide with parallel processing animation

**[On camera - Instructor]**

Welcome to Module 2.2.1.4: Batch Process Execution. This is where we scale up from running single jobs to orchestrating dozens or even hundreds of parallel job instances.

### Visual: Timeline comparison - serial vs parallel

**[Voice over visual]**

Imagine you need to process 10,000 transactions. Running them one at a time could take hours or even days. But what if you could split that work across 10 or 20 robots running simultaneously? You could finish in minutes instead of hours.

### Visual: Learning objectives slide

**[On camera - Instructor]**

In this video, you'll learn:
- When and why to use batch execution
- How to partition data across parallel jobs
- Efficient patterns for monitoring multiple jobs
- Result aggregation from batch runs
- Handling partial failures gracefully
- Performance optimization techniques

Let's learn to process at scale!

---

## Scene 2: The Scale Problem (2 minutes)

### Visual: Animation of single job processing items slowly

**[On camera - Instructor]**

Let me show you why batch execution matters with a real example.

### Visual: Calculation on screen

**[Voice over visual]**

You have 10,000 invoices to process. Each invoice takes about 2 seconds to handle. Let's do the math:

```
10,000 invoices × 2 seconds = 20,000 seconds
20,000 seconds ÷ 60 = 333 minutes
333 minutes ÷ 60 = 5.5 hours
```

Over five and a half hours! And that's if nothing goes wrong.

### Visual: Split into parallel streams

**[On camera - Instructor]**

Now, what if we split those 10,000 invoices into 10 batches of 1,000 each, and process them in parallel?

### Visual: Parallel calculation

**[Voice over]**

```
1,000 invoices × 2 seconds = 2,000 seconds per batch
All batches run in parallel = 2,000 seconds total
2,000 seconds ÷ 60 = 33 minutes
```

Thirty-three minutes! We just reduced processing time by 90%.

### Visual: Time savings visualization

That's the power of batch execution - massive time savings by leveraging parallelism.

---

## Scene 3: Data Partitioning (3 minutes)

### Visual: Data being split into chunks

**[On camera - Instructor]**

The first step in batch execution is partitioning your data - dividing it into non-overlapping chunks that can be processed independently.

### Screen recording - Partition function

**[Voice over screen recording]**

Let me show you a robust partitioning function:

```python
def partition_data(data, num_partitions):
    """Split data into equal-sized chunks"""

    chunk_size = len(data) // num_partitions

    partitions = []
    for i in range(num_partitions):
        start = i * chunk_size

        # Last partition gets any remainder
        if i < num_partitions - 1:
            end = start + chunk_size
        else:
            end = len(data)  # Include remainder in last batch

        partitions.append(data[start:end])

    return partitions
```

### Visual: Highlight remainder handling

**[Voice over]**

Notice how we handle the remainder. If you have 1,000 items and 10 partitions, that's 100 per partition. But with 1,005 items, the last partition gets 105. This ensures every item is processed.

### Visual: Example usage

```python
# 1000 invoice IDs
invoice_ids = list(range(1, 1001))

# Split into 10 batches
batches = partition_data(invoice_ids, num_partitions=10)

print(f"Created {len(batches)} batches")
for i, batch in enumerate(batches):
    print(f"Batch {i}: {len(batch)} items")
```

### Visual: Output

```
Created 10 batches
Batch 0: 100 items
Batch 1: 100 items
...
Batch 9: 100 items
```

Perfect - evenly distributed!

---

## Scene 4: Starting Batch Jobs (3.5 minutes)

### Visual: Multiple jobs being launched

**[On camera - Instructor]**

Once we have partitioned data, we need to start multiple jobs - each with different input arguments.

### Screen recording - Start batch function

**[Voice over screen recording]**

Here's a function that starts multiple jobs:

```python
def start_batch_jobs(orchestrator_url, token, folder_id,
                      release_key, input_batches):
    """Start multiple jobs with different inputs"""

    job_ids = []

    for i, batch_data in enumerate(input_batches):
        # Prepare input arguments for this batch
        input_args = {
            'batch_id': i,
            'data': batch_data
        }

        # Start job
        jobs = start_process_job(
            orchestrator_url, token, folder_id,
            release_key, input_args
        )

        job_ids.append(jobs[0]['Key'])
        print(f"✓ Started batch {i}, Job ID: {jobs[0]['Key']}")

    return job_ids
```

### Visual: Jobs starting in Orchestrator UI

**[On camera - Instructor]**

Each job gets its own batch of data in the input arguments. The robot knows which invoices to process from the 'data' argument.

### Visual: Error handling version

**[Voice over]**

In production, add error handling so one failed start doesn't stop the whole batch:

```python
def start_batch_jobs_safe(orchestrator_url, token, folder_id,
                          release_key, input_batches):
    """Start batch jobs with error handling"""

    job_ids = []
    failed_batches = []

    for i, batch_data in enumerate(input_batches):
        try:
            input_args = {'batch_id': i, 'data': batch_data}
            jobs = start_process_job(
                orchestrator_url, token, folder_id,
                release_key, input_args
            )
            job_ids.append(jobs[0]['Key'])
            print(f"✓ Started batch {i}")
        except Exception as e:
            print(f"✗ Failed to start batch {i}: {e}")
            failed_batches.append(i)

    return job_ids, failed_batches
```

Now if batch 3 fails to start, batches 4-10 still get launched.

---

## Scene 5: Concurrent Monitoring (4 minutes)

### Visual: Multiple job timelines running in parallel

**[On camera - Instructor]**

This is the tricky part - monitoring multiple jobs simultaneously. Let me show you the wrong way first, then the right way.

### Visual: Sequential monitoring (crossed out)

**[Voice over visual]**

**Wrong approach:**
```python
# DON'T DO THIS!
for job_id in job_ids:
    wait_for_job(job_id)  # Blocks until THIS job finishes
```

This monitors jobs one at a time. You'd wait for job 1 to finish before checking job 2. This defeats the entire purpose of parallel execution!

### Visual: Concurrent monitoring diagram

**[On camera - Instructor]**

The right approach: check ALL jobs in each poll iteration.

### Screen recording - Concurrent monitoring

**[Voice over screen recording]**

```python
def wait_for_all_jobs(orchestrator_url, token, folder_id,
                       job_ids, poll_interval=3, timeout=600):
    """Monitor multiple jobs concurrently"""

    pending_jobs = set(job_ids)
    completed_jobs = {}
    start_time = time.time()

    while pending_jobs:
        # Check timeout
        if time.time() - start_time > timeout:
            raise TimeoutError(f"{len(pending_jobs)} jobs still pending")

        # Check each pending job
        for job_id in list(pending_jobs):
            job = get_job(orchestrator_url, token, folder_id, job_id)

            if job['State'] in ['Successful', 'Faulted', 'Stopped']:
                # Job finished - move to completed
                completed_jobs[job_id] = job
                pending_jobs.remove(job_id)
                print(f"Job {job_id}: {job['State']}")

        # Sleep before next poll
        if pending_jobs:
            time.sleep(poll_interval)

    return completed_jobs
```

### Visual: Highlight the loop structure

**[Voice over]**

See how it works? We keep a set of pending jobs. Each poll iteration, we check all pending jobs. When one finishes, we move it to completed and remove from pending. When pending is empty, we're done!

### Visual: Progress tracking addition

Let's add progress reporting:

```python
# Inside the loop, after checking jobs
completed_count = len(completed_jobs)
total_jobs = len(job_ids)
progress_pct = (completed_count / total_jobs) * 100

print(f"Progress: {completed_count}/{total_jobs} ({progress_pct:.1f}%)")
```

Now you see real-time updates as jobs complete.

---

## Scene 6: Optimized Monitoring with OData (2 minutes)

### Visual: API call comparison - many vs one

**[On camera - Instructor]**

There's a more efficient way to monitor - instead of N API calls per poll (one per job), use a single API call that fetches all jobs at once.

### Screen recording - OData filter

**[Voice over screen recording]**

```python
# Build OData filter for all pending jobs
id_list = "','".join(pending_jobs)
filter_query = f"Key in ('{id_list}')"

# Single API call for all jobs
response = requests.get(
    f"{orchestrator_url}/odata/Jobs",
    headers={
        "Authorization": f"Bearer {token}",
        "X-UIPATH-OrganizationUnitId": str(folder_id)
    },
    params={"$filter": filter_query}
)

jobs = response.json()['value']
```

### Visual: Performance comparison

**[On camera - Instructor]**

With 20 jobs and 5-second polling:
- Old way: 20 API calls every 5 seconds = 240 calls/minute
- New way: 1 API call every 5 seconds = 12 calls/minute

Twenty times fewer API calls! This reduces load on Orchestrator and speeds up your monitoring.

---

## Scene 7: Result Aggregation (3 minutes)

### Visual: Puzzle pieces coming together

**[On camera - Instructor]**

Once all jobs complete, we need to aggregate the results - combining outputs from successful jobs and collecting errors from failed ones.

### Screen recording - Aggregation function

**[Voice over screen recording]**

```python
def aggregate_batch_results(completed_jobs):
    """Aggregate results from all jobs"""

    aggregated = {
        'total_jobs': len(completed_jobs),
        'successful': 0,
        'faulted': 0,
        'stopped': 0,
        'outputs': [],
        'errors': []
    }

    for job_id, job in completed_jobs.items():
        state = job['State']

        if state == 'Successful':
            aggregated['successful'] += 1

            # Parse and store outputs
            outputs = get_job_outputs(job)
            if outputs:
                aggregated['outputs'].append({
                    'job_id': job_id,
                    'batch_id': outputs.get('batch_id'),
                    'data': outputs
                })

        elif state == 'Faulted':
            aggregated['faulted'] += 1

            # Parse and store error
            error = parse_error_info(job.get('Info', ''))
            aggregated['errors'].append({
                'job_id': job_id,
                'error': error
            })

        else:  # Stopped
            aggregated['stopped'] += 1

    return aggregated
```

### Visual: Result structure visualization

**[On camera - Instructor]**

This gives you a complete picture: how many succeeded, how many failed, all the output data, and detailed error information.

### Visual: Combining outputs

**[Voice over]**

Now combine the outputs:

```python
def combine_batch_outputs(aggregated_results):
    """Combine outputs from all successful jobs"""

    all_items = []
    total_count = 0

    for output in aggregated_results['outputs']:
        batch_data = output['data']

        items = batch_data.get('processed_items', [])
        count = batch_data.get('processed_count', 0)

        all_items.extend(items)
        total_count += count

    return {
        'combined_items': all_items,
        'total_processed': total_count
    }
```

Now you have all results in one place!

---

## Scene 8: Error Handling and Partial Failures (2.5 minutes)

### Visual: Some jobs succeeding, some failing

**[On camera - Instructor]**

In batch execution, some jobs might fail while others succeed. This is called partial failure, and you need a strategy to handle it.

### Visual: Success rate calculation

**[Voice over visual]**

```python
def analyze_batch_results(aggregated_results):
    """Analyze batch execution results"""

    total = aggregated_results['total_jobs']
    successful = aggregated_results['successful']
    success_rate = (successful / total) * 100

    if success_rate == 100:
        status = "✓ Complete Success"
    elif success_rate >= 80:
        status = "⚠ Partial Success (acceptable)"
    elif success_rate >= 50:
        status = "⚠ Partial Success (concerning)"
    else:
        status = "✗ Majority Failed"

    return {
        'status': status,
        'success_rate': success_rate,
        'successful': successful,
        'faulted': aggregated_results['faulted'],
        'total': total
    }
```

### Visual: Decision tree

**[On camera - Instructor]**

Based on the success rate, you can decide:
- 100%: Perfect, proceed with results
- 80-99%: Acceptable, but investigate failures
- 50-79%: Concerning, consider reprocessing
- Below 50%: Major issue, don't trust results

### Visual: Retry pattern

**[Voice over]**

You can even retry failed batches:

```python
# Identify which batches failed
failed_batch_ids = [
    error['batch_id']
    for error in aggregated_results['errors']
    if 'batch_id' in error
]

# Retry those specific batches
retry_job_ids = start_batch_jobs_safe(
    orchestrator_url, token, folder_id,
    release_key,
    [original_batches[i] for i in failed_batch_ids]
)
```

This gives you automatic retry capability!

---

## Scene 9: Complete Example (2 minutes)

### Screen recording - Full workflow

**[On camera - Instructor]**

Let me show you everything working together in a complete example.

**[Voice over screen recording]**

```python
from config import OrchestratorConfig
from auth import TokenManager

# Setup
config = OrchestratorConfig()
token_mgr = TokenManager(config.client_id, config.client_secret)
token = token_mgr.get_token()

# Data to process
invoice_ids = list(range(1, 1001))  # 1000 invoices

# 1. Partition data
print("Partitioning data...")
batches = partition_data(invoice_ids, num_partitions=10)

# 2. Start jobs
print("Starting batch jobs...")
job_ids, failed_starts = start_batch_jobs_safe(
    config.orchestrator_url, token, config.folder_id,
    config.release_key, batches
)
print(f"Started {len(job_ids)} jobs")

# 3. Monitor
print("Monitoring jobs...")
completed = wait_for_all_jobs(
    config.orchestrator_url, token, config.folder_id, job_ids
)

# 4. Aggregate
print("Aggregating results...")
aggregated = aggregate_batch_results(completed)
combined = combine_batch_outputs(aggregated)

# 5. Analyze
analysis = analyze_batch_results(aggregated)
print(f"\n{analysis['status']}")
print(f"Success Rate: {analysis['success_rate']:.1f}%")
print(f"Processed: {combined['total_processed']} invoices")
```

### Visual: Output showing success

```
Partitioning data...
Starting batch jobs...
Started 10 jobs
Monitoring jobs...
Job abc123: Successful
Job def456: Successful
...
Progress: 10/10 (100.0%)
Aggregating results...

✓ Complete Success
Success Rate: 100.0%
Processed: 1000 invoices
```

Perfect!

---

## Scene 10: Performance Optimization (1.5 minutes)

### Visual: Performance tuning dials

**[On camera - Instructor]**

A few quick tips to optimize batch performance.

### Visual: Batch size considerations

**[Voice over visual]**

**Batch Size:**
- Too small: Underutilizes robots
- Too large: System overload
- Sweet spot: Match available robot licenses

```python
# Dynamic batch sizing
available_robots = get_available_robots(url, token, folder_id)
num_batches = min(available_robots, len(items), 20)
```

### Visual: Polling optimization

**Polling Interval:**
- Short jobs (< 1 min): 2 seconds
- Medium jobs (1-10 min): 5 seconds
- Long jobs (> 10 min): 15 seconds

### Visual: Staged execution

**Staged Execution:**
For very large batches, start jobs in waves:
```python
# Start 5 jobs at a time
for wave in range(0, len(batches), 5):
    wave_batches = batches[wave:wave+5]
    start_batch_jobs(url, token, folder_id, key, wave_batches)
    time.sleep(2)  # Brief pause between waves
```

---

## Scene 11: Conclusion (30 seconds)

### Visual: Key takeaways

**[On camera - Instructor]**

Excellent work! You now know how to execute processes at scale using batch execution. Remember the key steps:

1. Partition your data evenly
2. Start multiple jobs with different inputs
3. Monitor all jobs concurrently
4. Aggregate results from successful jobs
5. Handle partial failures gracefully
6. Optimize for your environment

### Visual: Next module preview

Next up: Asset Retrieval - reading configuration values, credentials, and secrets from Orchestrator assets to make your processes flexible and secure.

### Visual: End card

Thanks for watching! You're now ready to process thousands of items in minutes instead of hours. See you in the next module!

---

## Production Notes

**Graphics needed:**
- Serial vs parallel processing timeline comparison
- Data partitioning visualization
- Concurrent monitoring diagram (all jobs checked in parallel)
- API call reduction comparison (many vs one)
- Result aggregation puzzle pieces
- Success rate gauge/meter
- Performance optimization dials

**Screen recordings:**
- Creating partition_data function
- Implementing start_batch_jobs with error handling
- Writing wait_for_all_jobs monitoring loop
- Building OData filter for batch queries
- Creating aggregate_batch_results function
- Complete end-to-end batch execution example
- Dynamic batch sizing based on available robots

**Code examples:**
- partition_data function
- start_batch_jobs_safe function
- wait_for_all_jobs function
- Optimized OData monitoring
- aggregate_batch_results function
- analyze_batch_results function
- Complete batch execution workflow

**Callouts/Annotations:**
- Highlight remainder handling in partitioning
- Mark the set operations for pending jobs
- Annotate OData filter syntax
- Point out success rate thresholds
- Show polling interval trade-offs

**Pacing notes:**
- Clear explanation of why serial monitoring is wrong
- Methodical walkthrough of concurrent monitoring
- Emphasize the efficiency gain with OData batching
- Practical examples with real numbers (1000 invoices, 10 batches)
- Smooth integration showing all pieces together

**Accessibility:**
- Accurate captions for all technical terms
- Describe visualizations verbally
- High-contrast code display
- Verbal description of diagrams and timelines
