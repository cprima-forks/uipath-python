# Video Script: Module 1.2.2.4 - Action Center (Human-in-the-Loop)

**Duration:** 25 minutes
**Module:** 1.2.2.4
**Prerequisites:** Previous modules in Phase 1

---

## Section 1: Introduction (2 minutes)

### Visual
- Title slide
- Human-in-the-loop concept diagram
- Action Center UI screenshots

### Script

Welcome to Module 1.2.2.4: Action Center, the final module of Phase 1! In this module, we're going to learn about human-in-the-loop automation using UiPath Action Center.

Throughout Phase 1, we've focused on fully automated processes—robots executing workflows end-to-end without human intervention. But in the real world, not everything can or should be fully automated. Sometimes you need human judgment: a manager needs to approve a high-value expense, a compliance officer needs to review a filing, a human needs to validate uncertain OCR results.

Action Center is UiPath's system for integrating human input into automation workflows. It allows automation to pause, request human review or approval, wait for the human's decision, and then continue based on that input.

The power of Action Center is in handling exceptions. In a typical automation scenario, maybe 80% of cases are straightforward and can be fully automated. But 20% are exceptions that require human review. Without Action Center, you'd either process everything manually—which defeats the purpose of automation—or the exceptions would block your automation. With Action Center, you automate the 80%, create Actions for the 20%, and humans handle exceptions on their schedule. Best of both worlds.

In this module, you'll learn what Actions are, how they're created from workflows, how to wait for human input, and how to monitor Actions via the API. This is the final piece of the orchestration puzzle—combining the speed and consistency of automation with the judgment and expertise of humans.

Let's get started.

### Presenter Notes
- Emphasize that this completes Phase 1
- Use the 80/20 rule as the key concept
- Build excitement—this is about making automation practical for real-world scenarios

---

## Section 2: Understanding Action Center (4 minutes)

### Visual
- Action Center UI walkthrough
- Workflow → Action → Human → Continue flow
- Example: Invoice approval scenario

### Script

Action Center is UiPath's human-in-the-loop system. It's where automation workflows create tasks—called Actions—for humans to review and complete.

Here's how it works. A robot is processing invoices. Most invoices are standard—correct format, reasonable amounts, known vendors. The robot processes these automatically. But five invoices have amounts over $25,000, which exceeds the automatic approval limit. For these five, the robot creates Actions in Action Center. Each Action says: "This invoice needs manager approval. Here are the details. Please approve or reject."

The Action appears in Action Center, which is a web application and mobile app. The manager receives a notification, opens Action Center, sees the five invoices flagged for review, reviews the details, and for each one, clicks Approve or Reject and optionally adds comments. The robot, which was waiting for these decisions, receives the results and continues processing: approved invoices move forward, rejected ones are logged and set aside.

This is human-in-the-loop automation. The robot does the routine work, but humans make the judgment calls.

An Action is essentially a task assigned to a human. It contains:
- Title and description—what needs to be done
- Data to review—invoice details, document, whatever is relevant
- A form with fields for human input—Approve/Reject buttons, text fields for comments, dropdowns for options
- Assignment—which user or group should handle it
- Priority—High, Normal, Low
- Optional deadline—an SLA for completion

Actions are created by robot workflows, not manually. The workflow determines when a human is needed and creates the Action automatically. Humans complete Actions via the Action Center web UI or mobile app.

Actions flow through states. When created, an Action is Unassigned. When assigned to a user, it becomes Pending—waiting for the user to act. When the user opens it and starts working, it becomes InProgress. When the user submits their decision, it becomes Completed. If the deadline passes without completion, it becomes Expired. If the workflow or an admin cancels it, it becomes Canceled.

The workflow that created the Action can wait for it to be completed, retrieve the human's input, and continue processing based on that input. Alternatively, the workflow can create the Action and end, and a separate workflow can be triggered when the Action is completed. This async pattern is useful for long-running approvals where you don't want to tie up a robot waiting.

Action Center is powerful because it provides structure. Unlike ad-hoc processes like "email someone and ask for approval," Action Center provides tracking, SLA enforcement, audit trails, escalation, mobile access, and seamless integration with automation. It's built specifically for this use case.

Now let's look at when to use Action Center.

### Presenter Notes
- Walk through the invoice example step by step
- Show Action Center UI if possible
- Emphasize structure and integration benefits

---

## Section 3: When to Use Human-in-the-Loop (3 minutes)

### Visual
- Use case examples with icons
- Decision tree: automate fully vs use Action Center
- 80/20 visualization

### Script

When should you use Action Center versus full automation or manual processing?

The key principle is: automate what you can, use humans for what you must. Action Center is for exceptions and judgment calls, not routine processing.

Use Action Center for:
- Approvals. A purchase over $10,000 needs manager approval. An expense claim needs finance approval. A document needs compliance approval.
- Validation. OCR extracted data from a document, but confidence is low. A human validates and corrects the data.
- Exception handling. The automation encountered something it can't process automatically—a missing customer in the database, an unusual data format, an ambiguous situation. A human investigates and resolves it.
- Quality control. You're automating 1000 transactions. You randomly flag 50 for human spot-check to ensure quality.
- Complex decisions. Situations requiring judgment that's too complex or nuanced for rules-based logic.

Don't use Action Center for:
- Routine processing that can be fully automated. If 100% of invoices can be processed automatically, do that. Don't add human review just because.
- High-volume tasks where every item needs review. If you need human review for all 1000 items, Action Center won't help—you need a different solution or better automation.
- Immediate synchronous responses. If the human must respond right now, in real time, use attended automation instead. Action Center is for async—human responds on their schedule.

The 80/20 rule applies here. In most processes, 80% of cases are straightforward and can be automated. 20% are exceptions. Action Center is perfect for that 20%. It lets you achieve 80% straight-through processing while gracefully handling the exceptions.

Let's look at a few concrete examples. Invoice processing: 800 of 1000 invoices are standard → fully automated. 200 have issues → create Actions for review. Order fulfillment: 950 of 1000 orders can be processed automatically. 50 have customer data issues → create Actions for investigation. Document processing: 90% of documents extract perfectly → automated. 10% have low OCR confidence → create Actions for validation.

The pattern is always the same: automate the majority, use Actions for exceptions.

Now let's talk about how Actions are created.

### Presenter Notes
- Present clear decision criteria
- Use concrete examples
- Emphasize 80/20 principle repeatedly

---

## Section 4: Creating Actions from Workflows (4 minutes)

### Visual
- Studio form designer screenshot
- Create Form Task activity
- Wait for Task Completion activity
- Code flow diagram

### Script

Actions are created from UiPath Studio workflows, not directly via API. This is important to understand—you can't POST to an API endpoint to create an Action. Actions are created as part of automation workflows.

In Studio, you design a form that defines what the human will see and interact with. This is done in the Form Designer. You specify fields: text inputs, dropdowns, radio buttons, checkboxes, file uploads, data tables, whatever you need. You add labels, help text, and validation rules. This form becomes a reusable template.

Then in your workflow, you use the Create Form Task activity. You specify:
- Which form template to use
- The task title: "Invoice Approval Required"
- Data to pass to the form—invoice details that the human needs to see
- Who it's assigned to—a specific user, a group, or a role
- Priority: High, Normal, or Low
- Optional deadline: for example, due in 24 hours

When the workflow executes and reaches this activity, an Action is created in Action Center with all these details. The Action appears in the assigned person's queue.

The workflow can then wait for the Action to be completed using the Wait for Task Completion activity. This suspends the workflow—the robot pauses—until either the human completes the Action, the deadline expires, or the Action is canceled. When the Action completes, the workflow resumes and retrieves the human's input: what they selected, what they entered, whether they approved or rejected.

The workflow then continues processing based on that input. If approved, continue with the invoice. If rejected, log it and move on to the next one.

Here's a simple workflow structure:

```
1. Process Invoice
2. Check amount
3. If amount > $25,000:
     a. Create Form Task (approval form)
     b. Wait for Task Completion
     c. Get result (Approved/Rejected)
     d. If Approved: Continue processing
     e. If Rejected: Log and skip
4. Else: Process automatically (no Action needed)
```

This is synchronous human-in-the-loop. The workflow waits for the human. This works well for time-sensitive processes where you want the entire transaction to complete within a defined timeframe.

Alternatively, you can use an asynchronous pattern. The workflow creates the Action and then ends—doesn't wait. The human completes the Action at their convenience. A separate workflow, triggered by a webhook or schedule when Actions are completed, picks up the completed Actions and continues processing.

The async pattern is better for long-running approvals—like a manager approval that might take a day or two. You don't want a robot tied up waiting for a day. Instead, the robot creates the Action and moves on. When the manager eventually approves, a new robot execution picks up and finishes the work.

From an API perspective, you can't create Actions, but you can query them. You can check how many Actions are pending, which users have Actions assigned, what the status of each Action is, and so on. We'll look at that next.

### Presenter Notes
- Explain that API doesn't create Actions—workflows do
- Distinguish sync vs async patterns clearly
- Show the workflow structure visually if possible

---

## Section 5: Querying Actions via API (4 minutes)

### Visual
- API endpoint structure
- Code example showing query
- Action dashboard mockup

### Script

While Actions are created from workflows, you can query and monitor them via the API. This is useful for building custom dashboards, monitoring workloads, and tracking metrics.

Actions are accessed via the TaskItems endpoint:

```
GET /odata/TaskItems
```

TaskItems is the API name for Actions. Each TaskItem represents one Action.

You can filter by various fields. For example, get all Pending Actions assigned to a specific user:

```python
def get_pending_actions(base_url, token, folder_id, user_id=None):
    """Get pending Actions, optionally for specific user"""

    params = {
        '$filter': "Status eq 'Pending'"
    }

    if user_id:
        params['$filter'] += f" and AssignedToUserId eq {user_id}"

    response = requests.get(
        f'{base_url}/odata/TaskItems',
        params=params,
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id)
        }
    )

    response.raise_for_status()
    return response.json()['value']
```

This returns all pending Actions, or if you specify a user ID, just the ones assigned to that user.

Each TaskItem has fields like Id, Title, Status, Priority, AssignedToUserId, CreatedTime, DueDate, and Data. The Data field contains the form data—both the input data provided when the Action was created and the output data entered by the human when completing it.

You can use this to build reports. How many Actions are pending? How many are overdue? What's the average time to complete an Action? Which users have the most Actions assigned?

Here's an example that calculates metrics:

```python
def get_action_metrics(base_url, token, folder_id):
    """Calculate Action Center metrics"""

    all_actions = get_all_actions(base_url, token, folder_id)

    metrics = {
        'total': len(all_actions),
        'pending': 0,
        'completed': 0,
        'expired': 0
    }

    for action in all_actions:
        status = action['Status']
        metrics[status.lower()] = metrics.get(status.lower(), 0) + 1

    return metrics
```

You can also track completion times, identify bottlenecks, monitor workload distribution, and more.

This monitoring capability is important for operational management. You want to know if Actions are piling up, if certain users are overwhelmed, if SLAs are being missed. API access lets you build the monitoring and alerting you need.

One limitation: you can query Actions but you can't programmatically complete them or update them in significant ways via API. Actions are completed by humans via the Action Center UI. The API is primarily for reading and monitoring.

### Presenter Notes
- Walk through the code examples
- Emphasize monitoring and metrics use case
- Clarify that API is for reading, not completing Actions

---

## Section 6: Assignment, Priority, and Deadlines (3 minutes)

### Visual
- Assignment methods comparison
- SLA and escalation flow
- Priority queue visualization

### Script

When creating an Action, you specify who should handle it, how urgent it is, and when it's due. These factors determine how quickly the Action gets attention.

Assignment has three options. You can assign to a specific user by ID. Only that user sees the Action. This is rarely used—too rigid. You can assign to a group. All members of the group see the Action. The first person to claim it works on it. This is good for distributing work. Or you can assign to a role, like "Managers" or "Approvers." All users with that role see the Action. This is the most flexible—you don't need to know specific user IDs, and it automatically includes new users assigned that role.

Best practice: use roles or groups, not specific user IDs. This makes your workflows maintainable and flexible.

Priority can be High, Normal, or Low. High-priority Actions appear at the top of users' queues and typically get handled first. Use priority for genuinely urgent items—don't make everything High or it loses meaning.

Deadlines are optional but recommended. You set a due date—for example, due in 24 hours or due by end of day Friday. If the deadline passes without the Action being completed, the Action's state changes to Expired. You can configure what happens then: send escalation notifications, reassign to a senior approver, execute a default action, or just flag it for review.

Escalation is powerful. You might configure: if this approval isn't completed in 24 hours, email the user's manager and reassign the Action to the manager. Or if an Action is pending for 48 hours, mark it as critical and alert the operations team. Escalation ensures time-sensitive Actions don't get stuck.

Monitoring these metrics is important. If you see that certain users consistently have Actions expire, they may be overwhelmed or not checking Action Center regularly. You can address that through training, workload redistribution, or delegation setup.

From an API perspective, you can query Actions by due date and status to identify overdue or at-risk Actions:

```python
# Get Actions overdue
overdue = get_actions(filter="DueDate lt now() and Status eq 'Pending'")
```

This lets you build alerts: "10 Actions are overdue—investigate."

### Presenter Notes
- Stress using roles vs user IDs
- Explain escalation as safety net
- Mention that monitoring is key to preventing SLA misses

---

## Section 7: Best Practices and Common Pitfalls (3 minutes)

### Visual
- Best practices checklist
- Do's and Don'ts
- Example scenarios

### Script

Let's talk about best practices for using Action Center effectively.

Best practice one: Use Action Center for exceptions, not routine work. If you can automate it, automate it. Only use Actions when human judgment is truly needed. Creating an Action for every single transaction defeats the purpose of automation.

Best practice two: Keep forms simple and focused. Don't overwhelm users with 20 fields to fill out. Show only the data they need to make a decision. Use clear labels and provide context. The easier the form, the faster and more accurately humans will complete Actions.

Best practice three: Use roles and groups for assignment, not specific user IDs. This makes workflows flexible and maintainable. Don't hard-code "assign to user 123"—that breaks when that person leaves the company.

Best practice four: Set realistic deadlines and configure escalation. Don't set a 1-hour deadline if realistically it takes 4 hours. And always have an escalation plan for when deadlines are missed.

Best practice five: Monitor Action metrics. Track how many Actions are pending, how long they take to complete, which users are overloaded, and what the failure/expiry rate is. Use this data to optimize.

Best practice six: Train users. Humans need to know how to use Action Center—how to access it, how to complete Actions, what the expectations are. Poor user training leads to Actions sitting unattended.

Best practice seven: Log Action IDs with business transactions. When you create an Action for invoice INV-001, log the Action ID with that invoice record. This creates traceability. If someone asks "Why was invoice INV-001 rejected?" you can look up the Action and see exactly what the reviewer said.

Now common pitfalls. Pitfall one: Creating too many Actions. If 80% of your process requires Actions, rethink your automation. You're just digitizing manual work, not automating.

Pitfall two: No escalation or timeout handling. If an Action is never completed, your process is stuck. Always have a plan for timeouts.

Pitfall three: Assigning to specific users who might be unavailable. Use groups or roles so work can be redistributed.

Pitfall four: Unclear forms. If the human doesn't understand what they're being asked to do, they'll make mistakes or skip the Action. Make forms self-explanatory.

Pitfall five: Not monitoring. You create Actions and assume they're being handled. Without monitoring, you won't know if there are problems until it's too late.

Avoid these pitfalls and follow best practices, and Action Center will be a powerful tool for making automation practical and robust.

### Presenter Notes
- Present as actionable guidelines
- Use concrete examples of pitfalls
- Emphasize monitoring and training

---

## Section 8: Summary and Phase 1 Completion (2 minutes)

### Visual
- Summary slide with key points
- Phase 1 completion celebration
- Preview of Phase 2

### Script

Let's summarize what we've covered about Action Center.

Action Center is UiPath's human-in-the-loop system. It enables automation to pause and request human input—approvals, validations, exception handling, quality checks. Actions are tasks assigned to humans, created from workflows, completed via the Action Center UI.

Actions flow through states: Unassigned to Pending to InProgress to Completed. The workflow can wait for completion, retrieve the human's decision, and continue processing. Or the workflow can create the Action and end, with processing resuming asynchronously when the Action is completed.

You can query Actions via the API using GET /odata/TaskItems, which enables monitoring, metrics, and custom dashboards. Use assignment by role or group for flexibility. Set priorities and deadlines with escalation. And always use Action Center for exceptions, not routine work—aim for 80% automation, 20% Actions.

Best practices: keep forms simple, use roles for assignment, set realistic SLAs, monitor metrics, train users, and log Action IDs for traceability.

And with that, congratulations—you've completed Phase 1: Foundations!

Let's recap what you've learned across all of Phase 1. You started with Python fundamentals and async programming. You learned environment management and secret handling. You explored UiPath Orchestrator architecture, authentication, tenants, and folders. You learned how to start and monitor jobs, manage configuration with Assets, implement queue-based distributed processing, and integrate human judgment with Action Center.

You now have a solid foundation in automation orchestration and the groundwork for working with the UiPath Python SDK. You understand how the pieces fit together—robots, jobs, queues, assets, human tasks—and how to coordinate them via API.

In Phase 2, we'll introduce the UiPath Python SDK itself. You'll learn how to install it, authenticate, and use it to interact with Orchestrator services more easily than raw API calls. The SDK abstracts much of the complexity we've covered in Phase 1, but now you understand what's happening under the hood, which makes you a better developer.

Thank you for completing Phase 1, and I'll see you in Phase 2!

### Presenter Notes
- Celebrate the completion milestone
- Provide comprehensive recap of Phase 1
- Build anticipation for Phase 2
- Acknowledge the effort—Phase 1 is substantial

---

## Additional Teaching Notes

### Common Student Questions

**Q: Can I create Actions via API, not from workflows?**
A: No, Actions are created from Studio workflows using activities like Create Form Task. The API is for querying and monitoring, not creating.

**Q: What happens if a human never completes an Action?**
A: If a deadline is set, the Action expires after the deadline. You can configure escalation (reassign, notify, default action). Without a deadline, it stays Pending indefinitely.

**Q: Can I programmatically complete Actions via API?**
A: Not in the standard way. Actions are designed to be completed by humans via the Action Center UI. There may be ways to submit form data programmatically, but it's not the intended use case.

**Q: How do I test Actions during development?**
A: Create test Actions from your workflow in a dev environment. You can assign them to yourself and complete them to test the flow. Action Center works in all environments (dev, staging, prod).

**Q: Action Center vs Forms in a webpage—what's the difference?**
A: Action Center is integrated with automation workflows, provides structured tracking and SLA management, has mobile support, and maintains an audit trail. A webpage form is just data entry without automation context.

### Demo Prerequisites
- Access to UiPath Studio to show form designer and activities
- Access to Action Center UI to show human perspective
- Example workflow that creates and waits for Actions
- Ability to query TaskItems API

### Additional Examples
- Multi-step approval workflows (requires 2 managers)
- Document validation with corrections
- Exception handling with data enrichment
- Quality control sampling

### Assessment Tips
- Quiz emphasizes when to use Action Center vs alternatives
- Understanding of Action creation (from workflows, not API)
- Knowledge of states and lifecycle
- Best practices around assignment and SLA
- Architectural decisions about automation vs human-in-the-loop
