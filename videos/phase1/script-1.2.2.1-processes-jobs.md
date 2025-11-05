# Video Script: Module 1.2.2.1 - Processes and Jobs

**Duration:** 30 minutes
**Module:** 1.2.2.1
**Prerequisites:** Module 1.2.1.2 (Authentication), 1.2.1.3 (Tenants and Folders)

---

## Section 1: Introduction (2 minutes)

### Visual
- Title slide
- Process and Job relationship diagram
- Preview of what we'll build

### Script

Welcome to Module 1.2.2.1: Processes and Jobs. This is where things get exciting—we're going to start actually running automations programmatically!

Up to this point, we've covered the foundational topics: authentication, tenants, folders, deployment models. Now we're going to use that knowledge to do what Orchestrator is all about: executing automation.

In this module, you'll learn about Processes and Jobs—the core concepts for running automation. A Process is the automation itself—the workflow you created in UiPath Studio and published to Orchestrator. A Job is an execution of that Process—an instance of it running.

We'll cover how to start jobs via the API, how to pass parameters to them, how to monitor their execution, and how to retrieve results when they complete. By the end of this module, you'll be able to programmatically trigger automation and handle the entire execution lifecycle.

This is fundamental knowledge—every automation script you write will involve starting and monitoring jobs. So pay close attention, and let's get started!

### Presenter Notes
- Build excitement—this is where we actually run automation
- Emphasize that this is foundational for everything that follows
- Set clear expectations for what students will be able to do

---

## Section 2: Understanding Processes (3 minutes)

### Visual
- Studio to Orchestrator publishing flow diagram
- Process list in Orchestrator UI (screenshot)
- API Releases endpoint response

### Script

Let's start by understanding what a Process is.

When you create an automation workflow in UiPath Studio, you build it, test it locally, and then publish it. Publishing creates a package file—a .nupkg file—that contains your workflow and all its dependencies.

You then deploy this package to Orchestrator. Once deployed, it becomes a Process. In the Orchestrator UI, you see it listed under Processes in your folder. It has a name, a version number, and it's ready to be executed by robots.

A Process contains everything needed to run the automation: the workflow XAML with your automation logic, any dependencies like libraries or packages, configuration settings, and metadata like the entry point—which workflow file should start first.

Think of a Process like a program or application. It's the code, ready to run, but not running yet. Just like you have Microsoft Word installed on your computer—that's the program. When you open it, that's an instance of the program running. Same concept: Process is the automation, Job is it running.

In the API, Processes are called "Releases". This is important to know because when you're working with the API, you'll see the term "Release" everywhere. Don't be confused—Release and Process are the same thing.

You retrieve Processes using the /odata/Releases endpoint. Each Process has a Key field—a GUID that uniquely identifies it. This Key is called the ReleaseKey, and you'll need it to start jobs. We'll see how to get it in a moment.

Processes can have multiple versions. For example, you might have "Invoice Processing" version 1.0.0, and later publish version 1.0.1 with bug fixes. Each version is a separate Release in Orchestrator with its own ReleaseKey.

Now let's talk about Jobs—the execution of these Processes.

### Presenter Notes
- Use the software application analogy—very relatable
- Emphasize Release = Process (API terminology)
- Preview that we'll need the ReleaseKey to start jobs

---

## Section 3: Understanding Jobs (3 minutes)

### Visual
- Job lifecycle diagram
- Job list in Orchestrator UI with different states
- One Process → Many Jobs illustration

### Script

A Job is a single execution of a Process by a robot. When you start a Process, you create a Job. That Job tracks everything about that specific execution from start to finish.

One Process can have many Jobs. For example, let's say you have an "Invoice Processing" process. You run it Monday—that's Job #1. You run it again Tuesday—that's Job #2. You run it 100 times over a month—that's 100 jobs, all from the same Process.

Each Job is independent. Job #1 might succeed while Job #2 fails. They can run in parallel on different robots. They can have different input parameters. But they're all executions of the same Process.

Jobs have a lifecycle. When you start a job, it's created in the "Pending" state—waiting for an available robot. Once a robot is assigned and starts executing, the state changes to "Running". When execution finishes, the state changes to "Successful" if it completed without errors, "Faulted" if it failed with an error, or "Stopped" if someone manually stopped it.

Each Job tracks detailed information: which Process was executed—identified by the ReleaseKey, which Robot executed it, when it started and when it ended, its current State, any Input Arguments that were passed to it, any Output Arguments it returned, logs from the execution, and if it failed, error information.

This information is critical for monitoring and troubleshooting. If an automation fails, you can look at the Job to see exactly what happened: what time did it fail, what was the error message, what were the logs, what were the inputs.

Jobs are accessed through the /odata/Jobs endpoint. Each job has a unique Id—a numeric identifier you'll use to query for status and results.

So to summarize: Processes are the automation definitions, Jobs are the executions. One Process, many Jobs. Processes are what you deploy, Jobs are what you monitor.

Now let's see how to actually start a job.

### Presenter Notes
- Use concrete examples: "Invoice Processing" process with multiple job runs
- Emphasize the one-to-many relationship
- Preview that we'll use Job Id for monitoring

---

## Section 4: Starting Jobs via API (5 minutes)

### Visual
- StartJobs API endpoint documentation
- Code editor showing complete start job function
- Payload structure with annotations

### Script

To start a job via the API, you use the StartJobs endpoint. This is a special endpoint with a long name: POST /odata/Jobs/UiPath.Server.Configuration.OData.StartJobs.

This is what's called an OData Action—a special operation with a specific input structure. The long name with namespaces is part of the OData standard. Don't try to memorize it—just copy it from the documentation or your code templates.

To start a job, you need several pieces of information. First, the ReleaseKey—this identifies which Process to execute. Second, the folder context—the X-UIPATH-OrganizationUnitId header that specifies which folder this job should run in. Third, a robot selection strategy—how Orchestrator should choose which robot to run the job. And optionally, input arguments, priority, and other configuration.

Let's look at a complete example. I'm going to walk through this code step by step:

```python
def start_job(base_url, token, folder_id, release_key, input_args=None):
    """Start a job with optional input arguments"""

    payload = {
        "startInfo": {
            "ReleaseKey": release_key,
            "Strategy": "JobsCount",
            "RobotIds": [],
            "NoOfRobots": 0,
            "InputArguments": json.dumps(input_args or {})
        }
    }

    response = requests.post(
        f'{base_url}/odata/Jobs/UiPath.Server.Configuration.OData.StartJobs',
        json=payload,
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id),
            'Content-Type': 'application/json'
        }
    )

    response.raise_for_status()
    return response.json()['value']  # Returns list of created jobs
```

Let's break this down. The function takes five parameters: base_url and token for authentication, folder_id to specify which folder, release_key to specify which process, and optional input_args for parameters.

The payload has a startInfo object. ReleaseKey is the process to run—we'll see how to get this in a moment. Strategy is set to "JobsCount"—this tells Orchestrator to use load balancing, assigning the job to the robot with the fewest running jobs. RobotIds is empty because we're using JobsCount, not Specific. NoOfRobots is 0, which means "use one robot"—you'd set this to a higher number if you wanted to start multiple parallel jobs. InputArguments are the parameters—we'll cover these in detail shortly.

The API call is a POST to the StartJobs endpoint. We include our authentication token, the folder context header, and Content-Type: application/json. We send the payload as JSON.

The response contains a list of created jobs—yes, a list, because you can create multiple jobs in one call. We return this list. Each job object has an Id field you'll use for monitoring.

But wait—where do we get the ReleaseKey? Let's see:

```python
def get_release_key(base_url, token, folder_id, process_name):
    """Get ReleaseKey for a process by name"""

    response = requests.get(
        f'{base_url}/odata/Releases',
        params={'$filter': f"ProcessKey eq '{process_name}'"},
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id)
        }
    )

    releases = response.json()['value']
    if not releases:
        raise ValueError(f"Process '{process_name}' not found")

    return releases[0]['Key']  # This is the ReleaseKey
```

We query the /odata/Releases endpoint, filtering by the ProcessKey, which is the process name. We get back a list of releases—there might be multiple versions. We take the first one and extract its Key field. That's the ReleaseKey we need for StartJobs.

In practice, you'd look up the ReleaseKey once at the start of your script and reuse it for multiple job starts. Don't look it up every time—that's inefficient.

Now you have everything you need to start a job. But how do you know when it's done?

### Presenter Notes
- Walk through the code slowly—this is complex for beginners
- Emphasize the payload structure
- Point out the folder context header—easy to forget
- Show how to get the ReleaseKey

---

## Section 5: Robot Selection Strategies (3 minutes)

### Visual
- Diagram showing three strategies with robot assignment
- Use cases for each strategy
- Code examples side by side

### Script

Let's talk about robot selection strategies. When you start a job, you need to tell Orchestrator which robot should execute it. There are three strategies.

The first is "Specific". With this strategy, you explicitly choose which robot(s) to use by providing their IDs. In the payload, you set RobotIds to a list of robot IDs, like [101, 102]. This is useful when you have dedicated robots for specific tasks. For example, maybe you have a high-priority robot for urgent processing, or a robot with special permissions for sensitive data.

The second strategy is "JobsCount". This is load balancing. You leave RobotIds empty, and Orchestrator automatically assigns the job to the robot with the fewest currently running jobs. If you have five robots and four of them are busy but one is idle, the idle one gets the job. This is the most common strategy for production—it distributes work evenly across your robot pool.

The third strategy is "Random". Orchestrator picks a random robot from those available. This is rarely used—there's usually no benefit to random selection. JobsCount is almost always better.

In your payload, you specify the strategy with the Strategy field:

```python
# JobsCount example
payload = {
    "startInfo": {
        "Strategy": "JobsCount",
        "RobotIds": [],  # Empty - let Orchestrator choose
        "NoOfRobots": 1  # How many robots to use
    }
}

# Specific example
payload = {
    "startInfo": {
        "Strategy": "Specific",
        "RobotIds": [101, 102],  # These specific robots
        "NoOfRobots": 0  # Ignored when using Specific
    }
}
```

For most use cases, use JobsCount. It's simple, effective, and scales well. Only use Specific when you have a specific reason to target certain robots.

One more thing about JobsCount: you can set NoOfRobots to create multiple jobs at once. If you set NoOfRobots to 3, Orchestrator creates three jobs and assigns each to a different robot. This is useful for parallel processing—run the same automation three times in parallel.

Now let's talk about passing parameters to your jobs.

### Presenter Notes
- Clarify that JobsCount is the recommended default
- Give concrete examples of when to use Specific
- Explain NoOfRobots for parallel execution

---

## Section 6: Passing Input Arguments and Retrieving Outputs (4 minutes)

### Visual
- Workflow showing in/out arguments
- JSON structure of InputArguments
- Code showing argument serialization and retrieval

### Script

Most automations need input parameters. For example, if you have an "Invoice Processing" automation, you probably need to pass it the invoice ID, the amount, maybe a priority level. These are called Input Arguments.

In your UiPath workflow, you define arguments with names and types. For example: in_InvoiceID as a String, in_Amount as an Int32, in_Priority as a String. The "in_" prefix is a naming convention—not required, but common.

When starting a job via the API, you pass these arguments in the InputArguments field. Now here's the tricky part: InputArguments is a JSON string, not a Python dict. You need to serialize your arguments to JSON:

```python
# Define your arguments as a dict
input_args = {
    "in_InvoiceID": "INV-12345",
    "in_Amount": 1500,
    "in_Priority": "High"
}

# Serialize to JSON string
import json
input_args_json = json.dumps(input_args)

# Use in payload
payload = {
    "startInfo": {
        "ReleaseKey": release_key,
        "Strategy": "JobsCount",
        "InputArguments": input_args_json  # JSON string
    }
}
```

Important: the argument names must match your workflow exactly, including the "in_" prefix if you used it. If your workflow expects "in_InvoiceID" and you send "InvoiceID", it won't work. Case matters too.

After the job completes, you might want to retrieve output results. These are Output Arguments—values the workflow returns. For example, maybe your workflow returns out_TotalProcessed, out_Status, and out_ErrorCount.

Output arguments are in the OutputArguments field of the job, also as a JSON string. After the job completes, you query the job and parse this field:

```python
# Get the completed job
job = get_job(base_url, token, folder_id, job_id)

if job['State'] == 'Successful':
    # Parse output arguments
    outputs = json.loads(job['OutputArguments'])

    print(f"Total Processed: {outputs['out_TotalProcessed']}")
    print(f"Status: {outputs['out_Status']}")
    print(f"Error Count: {outputs['out_ErrorCount']}")
```

Both inputs and outputs use the same pattern: JSON strings that you serialize (json.dumps) for inputs and deserialize (json.loads) for outputs.

One common mistake: trying to pass a Python dict directly instead of a JSON string. The API requires a string, so always use json.dumps for inputs.

Now let's talk about monitoring job execution.

### Presenter Notes
- Emphasize the JSON string requirement—common source of errors
- Show the serialization and deserialization explicitly
- Mention that argument names must match exactly

---

## Section 7: Monitoring Job Execution (5 minutes)

### Visual
- Job lifecycle state diagram with timing
- Code showing polling loop
- Console output of monitoring in action

### Script

After you start a job, you need to monitor it to know when it completes and whether it succeeded or failed.

The approach is polling: you periodically query the job by its ID and check the State field. You keep polling until the State is a terminal state: Successful, Faulted, or Stopped.

Let's look at a monitoring function:

```python
import time

def wait_for_job(base_url, token, folder_id, job_id, poll_interval=5, timeout=3600):
    """Poll job until completion or timeout"""

    start_time = time.time()

    while True:
        # Check timeout
        if time.time() - start_time > timeout:
            raise TimeoutError(f"Job {job_id} timed out after {timeout}s")

        # Get job status
        job = get_job(base_url, token, folder_id, job_id)
        state = job['State']

        # Check if completed
        if state in ['Successful', 'Faulted', 'Stopped']:
            return job  # Completed

        # Still running
        print(f"Job {job_id}: {state}")
        time.sleep(poll_interval)
```

Let's walk through this. We take the job ID and a polling interval—how many seconds to wait between checks. I'm using 5 seconds, which is reasonable. Don't poll every second—that's too aggressive and wastes API calls. But don't wait too long either—5 to 10 seconds is a good balance.

We also have a timeout—maximum time to wait. If the job doesn't complete within this time, we raise an error. This prevents infinite loops if something goes wrong.

In the loop, we first check if we've exceeded the timeout. Then we get the job status by calling get_job—a simple GET to /odata/Jobs with the job ID. We check the State field.

If the state is Successful, Faulted, or Stopped, we're done—the job has completed. We return the job object.

If the state is Pending or Running, we print a status message and sleep for the polling interval, then loop again.

Here's the get_job function for reference:

```python
def get_job(base_url, token, folder_id, job_id):
    """Get job details by ID"""

    response = requests.get(
        f'{base_url}/odata/Jobs({job_id})',
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id)
        }
    )

    response.raise_for_status()
    return response.json()
```

Simple: GET to /odata/Jobs with the job ID in parentheses. Don't forget the folder context header.

Now let's put this all together into a complete execution function:

```python
def execute_process(base_url, token, folder_id, release_key, input_args):
    """Start job, wait for completion, return results"""

    # 1. Start job
    print("Starting job...")
    jobs = start_job(base_url, token, folder_id, release_key, input_args)
    job_id = jobs[0]['Id']
    print(f"Job ID: {job_id}")

    # 2. Monitor until completion
    print("Monitoring job...")
    final_job = wait_for_job(base_url, token, folder_id, job_id)

    # 3. Check result
    if final_job['State'] == 'Successful':
        outputs = json.loads(final_job['OutputArguments'])
        print("Job completed successfully!")
        return outputs
    else:
        error_info = final_job['Info']
        raise Exception(f"Job failed: {error_info}")

# Usage
result = execute_process(
    base_url, token, folder_id, release_key,
    input_args={"in_InvoiceID": "INV-001"}
)
print(f"Result: {result}")
```

This function encapsulates the entire workflow: start the job, monitor it, and return results. If it fails, it raises an exception with the error info.

This is a pattern you'll use constantly. Many of your automation scripts will be variations of this: start job, wait, check result.

Now let's talk about error handling.

### Presenter Notes
- Emphasize polling interval choice—5-10 seconds is good
- Walk through the timeout logic
- Show the complete pattern: start, monitor, check result

---

## Section 8: Handling Job Errors (3 minutes)

### Visual
- Job with Faulted state showing error information
- Code retrieving and displaying robot logs
- Error troubleshooting flowchart

### Script

When a job fails, its State becomes "Faulted". At that point, you need to understand what went wrong so you can fix it or alert someone.

The job object provides several sources of error information. The quickest is the Info field—a summary of the error:

```python
if job['State'] == 'Faulted':
    print(f"Error: {job['Info']}")
```

This gives you a high-level message, like "System.NullReferenceException: Object reference not set to an instance of an object."

For more detail, you can retrieve the robot logs. Logs contain the complete execution trace, including all the steps the robot took and detailed error messages:

```python
def get_job_logs(base_url, token, folder_id, job_key):
    """Get logs for a failed job"""

    response = requests.get(
        f'{base_url}/odata/RobotLogs',
        params={
            '$filter': f"JobKey eq {job_key}",
            '$orderby': 'TimeStamp desc'
        },
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id)
        }
    )

    return response.json()['value']

# Usage
if job['State'] == 'Faulted':
    logs = get_job_logs(base_url, token, folder_id, job['Key'])

    # Print error logs
    for log in logs:
        if log['Level'] == 'Error':
            print(f"{log['TimeStamp']}: {log['Message']}")
```

Note that we filter by JobKey, not JobId. JobKey is a GUID that uniquely identifies the job. Also note we're querying RobotLogs, not the job itself.

Common reasons jobs fail: invalid input arguments—missing a required parameter, or wrong data type; missing dependencies—the robot doesn't have a required package installed; permission errors—the robot doesn't have access to a file or system; business logic errors—the automation itself has a bug.

If a job stays in Pending for a long time and never starts, that's a different issue. It means no robots are available. Common causes: all robots are busy with other jobs, no robots have access to this folder, no robot has the required environment for this process, or all robots are disconnected.

You can check robot availability before starting a job:

```python
robots = get_available_robots(base_url, token, folder_id)
if not robots:
    raise Exception("No available robots")
```

Error handling is critical. Don't just assume jobs will succeed—always check the State and handle failures appropriately.

### Presenter Notes
- Show both quick error check (Info) and detailed check (logs)
- Explain common failure reasons
- Distinguish between Faulted (started but failed) and Pending forever (didn't start)

---

## Section 9: Job Scheduling (2 minutes)

### Visual
- Schedule creation UI and API comparison
- Cron expression examples
- Use cases for schedules vs immediate jobs

### Script

So far we've been starting jobs immediately—run this job right now. But sometimes you want jobs to run automatically on a schedule—every day at 9 AM, every hour, first day of the month, and so on.

For scheduled execution, you create a Schedule using the ProcessSchedules endpoint. A schedule defines when and how often to run a job:

```python
schedule_payload = {
    "Name": "Daily Invoice Processing",
    "ReleaseKey": release_key,
    "Enabled": True,
    "TimeZoneId": "UTC",
    "StartProcessCron": "0 9 * * *",  # Daily at 9 AM
    "InputArguments": json.dumps({"Mode": "Daily"})
}

response = requests.post(
    f'{base_url}/odata/ProcessSchedules',
    json=schedule_payload,
    headers=headers
)
```

The key field is StartProcessCron—a cron expression defining when to run. Cron expressions use the format: minute hour day month day-of-week. For example, "0 9 * * *" means minute 0, hour 9, any day, any month, any day of week—so daily at 9 AM.

Other examples: "0 */4 * * *" runs every 4 hours; "30 8 * * 1-5" runs weekdays at 8:30 AM; "0 0 1 * *" runs the first day of each month.

If you're not familiar with cron expressions, use crontab.guru—a website that explains cron expressions in plain English.

Schedules run automatically. Once created and enabled, Orchestrator creates jobs according to the schedule without any API calls from you. This is different from immediate jobs, where you explicitly start each one.

Choose schedules for recurring automation—daily reports, hourly data sync, monthly processing. Use immediate jobs for on-demand automation—triggered by user action, event-driven, or ad-hoc runs.

### Presenter Notes
- Keep this brief—focus is on immediate jobs
- Show a couple cron examples
- Mention crontab.guru tool
- Clarify schedules vs immediate

---

## Section 10: Best Practices (3 minutes)

### Visual
- Checklist of best practices
- Code examples showing good vs bad patterns
- Performance and error handling tips

### Script

Let's wrap up with some best practices for working with jobs.

First, always validate input arguments before starting a job. Check that required parameters are present and have the right types. Don't start a job just to have it fail immediately because of bad inputs.

Second, use the JobsCount strategy for load balancing unless you have a specific reason to choose robots manually. JobsCount scales better and distributes work evenly.

Third, set appropriate timeouts when monitoring jobs. Don't wait forever—if a job doesn't complete within a reasonable time, something is wrong. Timeout and investigate.

Fourth, don't poll too frequently. Polling every second is wasteful. Every 5 to 10 seconds is sufficient for most use cases. The job state won't change that quickly, so frequent polling doesn't help.

Fifth, cache release keys. Look up the ReleaseKey once at startup and reuse it. Don't query for it every time you start a job—that's inefficient.

Sixth, implement retry logic with exponential backoff for starting jobs. Network errors happen, API rate limits happen. If a job start fails, wait a bit and try again. But don't retry forever—after 3 or 4 attempts, give up and alert someone.

Seventh, log job IDs for tracking. When you start a job, log the job ID along with context—what invoice, what user triggered it, whatever is relevant. This makes troubleshooting much easier later.

Eighth, monitor your job queue. If you have a lot of jobs in Pending state, that's a warning sign—you might not have enough robots, or there's a bottleneck somewhere. Monitor this and scale accordingly.

And finally, always handle errors gracefully. Check the job State, retrieve error information, log it, and decide what to do—retry, alert, skip and continue. Don't let one failed job crash your entire script.

These practices will help you build robust, production-quality automation scripts.

### Presenter Notes
- Present as actionable guidelines
- Emphasize error handling and monitoring
- Mention that these are learned from production experience

---

## Section 11: Summary and Next Steps (2 minutes)

### Visual
- Summary slide with key points
- Process → Job → Result flow diagram
- Next module preview

### Script

Let's summarize what we've covered about Processes and Jobs.

A Process is a published automation package—the workflow ready to execute. A Job is an execution instance of that Process. One Process can have many Jobs.

To start a job, use the StartJobs API endpoint with a payload specifying the ReleaseKey, robot strategy, and optional input arguments. The most common strategy is JobsCount for load balancing.

After starting a job, monitor it by polling the State field until it reaches a terminal state: Successful, Faulted, or Stopped. Retrieve output arguments from successful jobs, and error information from failed jobs.

For recurring automation, create Schedules using the ProcessSchedules endpoint with cron expressions. For on-demand automation, start jobs immediately with StartJobs.

Best practices: validate inputs, use appropriate polling intervals, implement timeouts, cache release keys, handle errors gracefully, and monitor your job queue.

You now have the knowledge to programmatically run automation and handle the entire job lifecycle. This is the foundation for building automation solutions with the Python SDK.

In the next module, we'll cover Assets—how to store and retrieve configuration values and secrets that your automations need. Assets are critical for managing environment-specific settings without hard-coding them in your workflows.

Thanks for watching, and I'll see you in the next module!

### Presenter Notes
- Quick recap of main points
- Emphasize that this is foundational knowledge
- Preview assets as next topic

---

## Additional Teaching Notes

### Common Student Questions

**Q: How long should a job stay in Pending?**
A: Typically seconds if robots are available. If Pending for more than a minute or two, check robot availability, folder permissions, and connectivity.

**Q: Can I cancel a running job?**
A: Yes, use the StopJob endpoint: POST /odata/Jobs({id})/UiPath.Server.Configuration.OData.StopJob. The job State will become 'Stopped'.

**Q: Can I restart a failed job?**
A: Not directly—you start a new job with the same parameters. Use the failed job's InputArguments to create an identical new job.

**Q: What if the ReleaseKey changes?**
A: ReleaseKeys change when you publish a new version of the process. Either update your code with the new key, or look it up dynamically by process name.

**Q: How do I run jobs in parallel?**
A: Set NoOfRobots to the number of parallel executions you want, or start multiple jobs in quick succession. Ensure you have enough available robots.

### Demo Prerequisites
- Access to UiPath Orchestrator (Cloud or On-Premises)
- At least one process deployed and one robot connected
- Python environment with requests library
- Authentication credentials (External Application)

### Additional Examples
- Batch processing: starting many jobs with different inputs
- Pipeline: passing output from one job as input to another
- Error recovery: automatically retrying failed jobs
- Reporting: generating job execution reports

### Assessment Tips
- Quiz emphasizes practical knowledge: how to start jobs, monitor them, handle errors
- Students should understand the complete workflow: start → monitor → result
- Common mistakes: forgetting folder context, not serializing input arguments, polling too fast
