# Video Script: Process Invocation Basics
## Module 2.2.1.1 - SDK Fundamentals

**Duration:** 28 minutes
**Target Audience:** Developers starting jobs programmatically
**Prerequisites:** Modules 2.1.1.1-2.1.2.2, OAuth authentication, Orchestrator access
**Learning Outcomes:** Start and manage process jobs via API

---

## Scene 1: Introduction (2 minutes)

### Visual: Title slide with automation workflow

**[On camera - Instructor]**

Welcome to Module 2.2.1.1: Process Invocation Basics. This is where everything comes together - we finally get to actually RUN automation processes from Python code!

### Visual: Diagram showing API → Orchestrator → Robot → Process

**[Voice over visual]**

Up until now, we've been setting up infrastructure: installing the SDK, managing dependencies, authenticating with OAuth, configuring environments. Now we put it all to use by programmatically starting UiPath processes.

### Visual: Learning objectives slide

**[On camera - Instructor]**

In this video, you'll learn:
- How to identify and reference processes using ReleaseKeys
- The complete StartJobs API workflow
- Different job execution strategies
- How to pass input arguments to processes
- Error handling and best practices

Let's start running some automation!

---

## Scene 2: Understanding Processes vs Jobs (3 minutes)

### Visual: Diagram showing Process Package → Job Instance

**[On camera - Instructor]**

Before we write any code, we need to understand the relationship between processes and jobs.

### Visual: Animated comparison

**[Voice over animation]**

Think of a **Process** as a recipe - it's your published automation workflow sitting in Orchestrator, waiting to be executed. It has a name, a version, and contains all the steps your robot will perform.

A **Job** is like actually cooking from that recipe - it's one specific execution of the process. You can start multiple jobs from the same process, just like you can cook the same recipe multiple times.

### Visual: Process details showing versions

**[Voice over]**

Processes are versioned - you might have MyProcess version 1.0.0, 1.0.1, 1.1.0. Each version is a separate "release" in Orchestrator.

**[On camera - Instructor]**

To start a job, you need to tell Orchestrator two things: WHICH process to run (identified by a ReleaseKey), and HOW to run it (which robots, what inputs). Let's tackle the "which" first.

---

## Scene 3: ReleaseKey - Process Identification (3 minutes)

### Visual: ReleaseKey highlighted in Orchestrator UI

**[On camera - Instructor]**

The ReleaseKey is a unique identifier - a GUID - for a specific version of a process package.

### Screen recording - Orchestrator UI

**[Voice over screen recording]**

Let me show you how to find it. I'm in Orchestrator, navigating to the Processes page.

Here's my process called "DataProcessing". I'll click on it to see details.

### Visual: Highlight ReleaseKey field

See this ReleaseKey field? That's a GUID that uniquely identifies this version of the process. Copy this - we'll need it in our Python code.

**[On camera - Instructor]**

But hardcoding ReleaseKeys isn't ideal - what if you update the process? Let me show you how to retrieve it programmatically.

### Screen recording - Python script

**[Voice over screen recording]**

We can query the Releases endpoint:

```python
import requests

# Query for releases by process name
response = requests.get(
    f"{ORCHESTRATOR_URL}/odata/Releases",
    headers={
        "Authorization": f"Bearer {token}",
        "X-UIPATH-OrganizationUnitId": str(folder_id)
    },
    params={
        "$filter": "ProcessKey eq 'DataProcessing'",
        "$orderby": "Created desc"  # Latest first
    }
)

releases = response.json()['value']
if releases:
    release_key = releases[0]['Key']
    version = releases[0]['ProcessVersion']
    print(f"Found release {version}: {release_key}")
```

### Visual: Output showing ReleaseKey

Perfect! We now have the ReleaseKey programmatically. This will get the latest version, or you can filter by specific version if needed.

---

## Scene 4: StartJobs API - The Basics (4 minutes)

### Visual: API endpoint documentation

**[On camera - Instructor]**

Now let's start a job. The endpoint is a bit of a mouthful, but you'll get used to it.

### Visual: Endpoint URL highlighted

**[Voice over]**

```
POST /odata/Jobs/UiPath.Server.Configuration.OData.StartJobs
```

That "UiPath.Server.Configuration.OData" part is the OData namespace. It's verbose but it's the standard.

### Screen recording - Building the request

**[Voice over screen recording]**

Let's build our first job start request. We need three things: the URL, headers, and payload.

```python
import json

# Endpoint
url = f"{ORCHESTRATOR_URL}/odata/Jobs/UiPath.Server.Configuration.OData.StartJobs"

# Headers
headers = {
    "Authorization": f"Bearer {access_token}",
    "X-UIPATH-OrganizationUnitId": str(folder_id),
    "Content-Type": "application/json"
}
```

### Visual: Highlight folder context header

**[Voice over]**

That X-UIPATH-OrganizationUnitId header is critical - it tells Orchestrator which folder we're working in. Remember, processes and robots are folder-specific.

### Visual: Payload structure

```python
# Payload
payload = {
    "startInfo": {
        "ReleaseKey": release_key,
        "Strategy": "JobsCount",
        "RobotIds": [],
        "NoOfRobots": 1,
        "InputArguments": "{}"
    }
}
```

### Visual: Annotate each field

**[Voice over]**

Let's break down this startInfo object:

- **ReleaseKey**: Which process to run
- **Strategy**: How to select robots (more on this shortly)
- **RobotIds**: List of specific robot IDs (empty for JobsCount)
- **NoOfRobots**: How many jobs to start
- **InputArguments**: Data to pass to the process (as JSON string)

**[On camera - Instructor]**

Now let's actually send this request and see what happens.

---

## Scene 5: Executing the Request (3 minutes)

### Screen recording - Complete example

**[On camera - Instructor]**

Watch carefully - I'm going to run this for the first time.

**[Voice over screen recording]**

```python
# Make the request
response = requests.post(url, headers=headers, json=payload)

# Check for errors
response.raise_for_status()

# Parse response
result = response.json()
jobs = result['value']

print(f"Created {len(jobs)} job(s)")
for job in jobs:
    print(f"  Job ID: {job['Key']}")
    print(f"  State: {job['State']}")
    print(f"  Robot: {job['Robot']['Name']}")
```

### Visual: Response output

**[Voice over output]**

```
Created 1 job(s)
  Job ID: 7d8f9a12-3b4c-5d6e-7f8g-9h0i1j2k3l4m
  State: Pending
  Robot: Robot1
```

**[On camera - Instructor]**

Success! Notice the State is "Pending" - that's important. The API returns immediately after CREATING the job. The actual execution happens asynchronously. The robot will pick up this pending job and execute it.

This is a critical concept: StartJobs doesn't wait for completion. It just creates the job and returns. We'll learn how to monitor job progress in the next module.

---

## Scene 6: Job Strategies (4 minutes)

### Visual: Three strategy diagrams

**[On camera - Instructor]**

Let's talk about the three job execution strategies: JobsCount, Specific, and All.

### Visual: JobsCount strategy diagram

**[Voice over]**

**JobsCount** - the most common strategy. You specify how many jobs to start, and Orchestrator picks any available robots in the folder.

Use this when you don't care which robot runs the job - you just need it done.

```python
"Strategy": "JobsCount",
"RobotIds": [],  # Empty
"NoOfRobots": 3,  # Start 3 jobs
```

This creates 3 jobs on any 3 available robots.

### Visual: Specific strategy diagram

**[Voice over]**

**Specific** - for when you need jobs on particular robots. Maybe Robot1 has special database access, or you're testing on a specific machine.

```python
"Strategy": "Specific",
"RobotIds": [123, 456],  # Specific robot IDs
"NoOfRobots": 0,  # Ignored for Specific
```

This creates jobs ONLY on robots 123 and 456.

### Visual: All strategy diagram

**[Voice over]**

**All** - broadcasts the job to every available robot in the folder. Useful for mass updates or parallel processing across your entire robot fleet.

```python
"Strategy": "All",
"RobotIds": [],
"NoOfRobots": 0,  # Both ignored for All
```

### Screen recording - Demonstrating strategies

**[On camera - Instructor]**

Let me show you the difference in practice. I'll start jobs with each strategy and show the results.

**[Voice over demo]**

First, JobsCount with 2 robots... Created 2 jobs on Robot1 and Robot3.

Now Specific with robot IDs 123 and 456... Jobs created on exactly those two robots.

Finally, All... Jobs created on all 5 robots in my folder.

---

## Scene 7: Input Arguments (3 minutes)

### Visual: Process with input parameters

**[On camera - Instructor]**

Most real processes need input data - customer names, file paths, configuration flags. Let's pass arguments from Python.

### Visual: Studio showing input arguments definition

**[Voice over]**

In Studio, your process might have input arguments like "in_CustomerName" and "in_Priority". We need to pass values for these from our API call.

### Screen recording - Preparing inputs

**[Voice over screen recording]**

The trick is that InputArguments must be a JSON *string*, not a Python dictionary.

```python
import json

# Create input data as dictionary
inputs = {
    "in_CustomerName": "Acme Corporation",
    "in_Priority": "High",
    "in_OrderCount": 5
}

# Convert to JSON string
input_json = json.dumps(inputs)
print(f"InputArguments: {input_json}")
```

### Visual: Output showing JSON string

```
InputArguments: {"in_CustomerName":"Acme Corporation","in_Priority":"High","in_OrderCount":5}
```

Now use this in the payload:

```python
payload = {
    "startInfo": {
        "ReleaseKey": release_key,
        "Strategy": "JobsCount",
        "RobotIds": [],
        "NoOfRobots": 1,
        "InputArguments": input_json  # JSON string
    }
}
```

### Visual: Running job with inputs

**[Voice over]**

When the job runs, the robot receives these values and can access them as input arguments in the workflow.

**[On camera - Instructor]**

Common mistake: forgetting to stringify the JSON. The API expects a string, not a dictionary.

---

## Scene 8: Error Handling (3 minutes)

### Visual: Common error scenarios

**[On camera - Instructor]**

Let's talk about what goes wrong and how to handle it.

### Screen recording - Error scenarios

**[Voice over screen recording]**

Error 1: Invalid ReleaseKey

```python
# Using wrong ReleaseKey
payload["startInfo"]["ReleaseKey"] = "invalid-guid"
response = requests.post(url, headers=headers, json=payload)
```

### Visual: Error message

```
400 Bad Request: Release not found
```

**[Voice over]**

This means either the ReleaseKey is wrong, or the process isn't deployed to the specified folder.

Error 2: No robots available

```python
# All robots busy or disconnected
payload["startInfo"]["NoOfRobots"] = 10
response = requests.post(url, headers=headers, json=payload)
```

### Visual: Empty response

```
Created 0 jobs
```

**[Voice over]**

The API succeeds but returns an empty job array. No robots were available to start the jobs.

### Visual: Robust error handling code

**[On camera - Instructor]**

Always implement proper error handling:

```python
try:
    response = requests.post(url, headers=headers, json=payload)
    response.raise_for_status()

    jobs = response.json()['value']

    if not jobs:
        print("Warning: No jobs created - no robots available")
        return None

    return jobs

except requests.exceptions.HTTPError as e:
    if e.response.status_code == 400:
        error_msg = e.response.json().get('message', '')
        if 'Release not found' in error_msg:
            raise ValueError("Invalid ReleaseKey or process not deployed")
    raise
```

---

## Scene 9: Practical Pattern - Reusable Function (2 minutes)

### Screen recording - Building helper function

**[On camera - Instructor]**

Let me show you a production-ready pattern for starting jobs.

**[Voice over screen recording]**

```python
def start_process_job(
    orchestrator_url,
    token,
    folder_id,
    release_key,
    input_args=None,
    robot_count=1,
    strategy="JobsCount"
):
    """
    Start a UiPath process job

    Args:
        orchestrator_url: Orchestrator base URL
        token: OAuth access token
        folder_id: Folder ID
        release_key: Process ReleaseKey
        input_args: Dictionary of input arguments
        robot_count: Number of jobs to start
        strategy: Job strategy (JobsCount/Specific/All)

    Returns:
        List of created job objects
    """
    url = f"{orchestrator_url}/odata/Jobs/UiPath.Server.Configuration.OData.StartJobs"

    headers = {
        "Authorization": f"Bearer {token}",
        "X-UIPATH-OrganizationUnitId": str(folder_id),
        "Content-Type": "application/json"
    }

    payload = {
        "startInfo": {
            "ReleaseKey": release_key,
            "Strategy": strategy,
            "RobotIds": [],
            "NoOfRobots": robot_count,
            "InputArguments": json.dumps(input_args or {})
        }
    }

    response = requests.post(url, headers=headers, json=payload)
    response.raise_for_status()

    return response.json()['value']
```

### Visual: Using the function

```python
# Easy to use!
jobs = start_process_job(
    orchestrator_url=ORCHESTRATOR_URL,
    token=access_token,
    folder_id=12345,
    release_key="550e8400-...",
    input_args={"CustomerName": "Acme"},
    robot_count=1
)
```

---

## Scene 10: Complete Demo (2 minutes)

### Screen recording - End-to-end example

**[On camera - Instructor]**

Let me put it all together with a complete, working example.

**[Voice over screen recording]**

```python
from config import OrchestratorConfig
from auth import TokenManager

# Load configuration
config = OrchestratorConfig(env_file='.env.dev')

# Authenticate
token_mgr = TokenManager(config.client_id, config.client_secret)
token = token_mgr.get_token()

# Get ReleaseKey for process
release_key = get_release_key_by_name(
    config.orchestrator_url,
    token,
    config.folder_id,
    process_name="DataProcessing"
)

# Prepare inputs
inputs = {
    "in_FilePath": "/data/input.xlsx",
    "in_Priority": "High"
}

# Start job
jobs = start_process_job(
    orchestrator_url=config.orchestrator_url,
    token=token,
    folder_id=config.folder_id,
    release_key=release_key,
    input_args=inputs
)

# Display results
for job in jobs:
    print(f"✓ Job {job['Key']} started on {job['Robot']['Name']}")
    print(f"  State: {job['State']}")
    print(f"  You can monitor this job in Orchestrator")
```

### Visual: Output showing successful job creation

**[On camera - Instructor]**

There we go - a production-ready job start implementation!

---

## Scene 11: Hands-On Exercise Preview (1 minute)

### Visual: Exercise instructions

**[On camera - Instructor]**

Time for hands-on practice! Pause and complete this exercise:

1. Get the ReleaseKey for one of your processes
2. Implement the start_process_job function
3. Start a job without input arguments
4. Start a job WITH input arguments
5. Try different strategies (JobsCount, Specific if you have robot IDs)
6. Implement error handling
7. Start multiple jobs in parallel

This should take about 30 minutes. Solution in course materials.

---

## Scene 12: Conclusion (1 minute)

### Visual: Key takeaways slide

**[On camera - Instructor]**

Excellent work! You can now programmatically start UiPath processes from Python. Let's recap the key points:

**[Voice over]**

- ReleaseKey uniquely identifies process versions
- StartJobs API creates jobs asynchronously
- Choose strategy based on robot selection needs
- Input arguments must be JSON strings
- Folder context is required via header
- Always handle errors appropriately

### Visual: Next module preview

In Module 2.2.1.2, we'll learn how to monitor these jobs - polling for status, waiting for completion, and retrieving results.

### Visual: End card

Thank you for watching! You're making great progress through the Core Services section. See you in the next module!

---

## Production Notes

**Graphics needed:**
- Process vs Job diagram
- ReleaseKey visualization
- Job execution flow (API → Orchestrator → Robot)
- Three strategies comparison table
- Async execution timeline
- Error scenarios flowchart
- Folder context diagram

**Screen recordings:**
- Finding ReleaseKey in Orchestrator UI
- Querying releases via API
- Complete job start request construction
- Running jobs with different strategies
- Passing input arguments
- Error scenarios (invalid ReleaseKey, no robots)
- Complete end-to-end demo

**Code examples:**
- Get ReleaseKey query
- Basic StartJobs request
- All three strategies
- Input arguments preparation
- Error handling
- Reusable start_process_job function
- Complete integration example

**Callouts/Annotations:**
- Highlight ReleaseKey in UI and code
- Point out folder context header
- Mark InputArguments as JSON string
- Annotate strategy differences
- Show Pending state in response

**Pacing notes:**
- Clear explanation of Process vs Job concept
- Methodical request construction
- Emphasize async execution
- Practical demonstrations of each strategy
- Real error scenarios
- Smooth flow to complete example

**Accessibility:**
- Accurate captions for technical terms
- Describe Orchestrator UI navigation verbally
- High-contrast code display
- Verbal description of all diagrams
