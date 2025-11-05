# Video Script: UiPath Orchestrator Architecture

**Module:** 1.2.1.1
**Duration:** 30 minutes
**Target Audience:** UiPath Agent Developers (Beginner to Intermediate)
**Prerequisites:** Core Python and Async modules

---

## Section 1: Introduction (3 minutes)

**[VISUAL: Title slide with UiPath Orchestrator logo]**

**Presenter:**

"Welcome to Module 1.2.1.1: UiPath Orchestrator Architecture. This is our first module diving into UiPath-specific concepts, and it's a critical one.

Up to this point, we've covered Python fundamentals—core Python, async programming, and environment management. Now we're going to learn about UiPath Orchestrator, the centralized platform that makes enterprise automation possible.

If you've used UiPath Studio to build workflows, you know that's where automation is created. But Studio is just the beginning. Orchestrator is where you deploy, manage, monitor, and scale your automations across your entire organization.

In this module, we'll cover:
- What Orchestrator is and why it matters
- The core components: robots, processes, queues, and assets
- The organizational structure: tenants and folders
- The difference between Cloud and On-Premises deployment
- How to navigate the Orchestrator UI

By the end, you'll understand Orchestrator's architecture and be ready to interact with it programmatically using the Python SDK.

Let's start with the big picture."

**[VISUAL: Transition to ecosystem diagram]**

---

## Section 2: Orchestrator in the UiPath Ecosystem (3 minutes)

**[VISUAL: Diagram showing Studio → Orchestrator → Robots]**

**Presenter:**

"To understand Orchestrator, you need to understand where it fits in the UiPath ecosystem.

There are three main components:

**[VISUAL: Highlight Studio]**

UiPath Studio—this is where developers build automations. It's the IDE, the development environment. You drag and drop activities, write expressions, test workflows. When your automation is ready, you publish it as a package.

**[VISUAL: Highlight Orchestrator]**

That package goes to Orchestrator—the central management platform. Orchestrator is like the brain of your automation infrastructure. It stores your processes, manages your robots, maintains queues of work, and stores configuration in assets. It schedules when things run, monitors execution, and provides analytics.

**[VISUAL: Highlight Robots]**

Finally, Robots—these are the execution agents that actually run your automations. A robot is software that connects to Orchestrator, receives instructions about what process to execute, runs it, and reports back the results.

**[VISUAL: Show the flow]**

The flow is: Build in Studio → Publish to Orchestrator → Deploy to Robots → Execute.

Studio is development. Orchestrator is management. Robots are execution.

**[VISUAL: Analogy slide]**

Think of it like a restaurant: Studio is the recipe development kitchen. Orchestrator is the head chef coordinating everything. Robots are the line cooks actually preparing the food.

In this module, we're focusing entirely on Orchestrator—understanding its architecture, components, and how it organizes everything."

---

## Section 3: Deployment Models (4 minutes)

**[VISUAL: Cloud vs On-Premises comparison]**

**Presenter:**

"Orchestrator comes in two flavors: Cloud and On-Premises.

**[VISUAL: Show Cloud Orchestrator]**

Cloud Orchestrator is hosted by UiPath at cloud.uipath.com. It's a fully managed service. UiPath handles all the infrastructure—servers, databases, networking, security. They ensure high availability, perform updates, and provide support. You just log in and use it.

The benefits of Cloud:
- Quick setup—create an organization and you're ready
- No infrastructure to manage
- Automatic updates—you always have the latest features
- High availability with 99.9% uptime SLA
- Global access from anywhere
- Pay-as-you-go licensing

**[VISUAL: Show On-Premises Orchestrator]**

On-Premises Orchestrator is self-hosted in your own data center. You install it on your servers, configure your database, set up networking, and manage everything yourself.

The benefits of On-Premises:
- Full control over infrastructure
- Data stays entirely within your network
- Custom network configurations
- Integration with isolated systems
- Compliance requirements satisfied
- One-time licensing (though you pay for infrastructure)

**[VISUAL: Decision matrix]**

When should you choose each?

Choose Cloud when:
- You want quick setup
- You have standard requirements
- You want a managed service
- You're okay with data in UiPath's cloud
- You want minimal IT overhead

Choose On-Premises when:
- You have strict data sovereignty requirements
- You operate in an isolated network
- You have specific compliance needs
- You need custom integrations
- You want complete control

**[VISUAL: Statistics]**

Most organizations today are choosing Cloud because it's simpler and UiPath handles everything. But if you're in healthcare, banking, or government, On-Premises might be required for regulatory reasons.

For this course, we'll focus primarily on Cloud Orchestrator since that's what most developers will use. The concepts are the same—only the hosting differs."

---

## Section 4: Organizational Hierarchy (4 minutes)

**[VISUAL: Hierarchy diagram]**

**Presenter:**

"Now let's understand how Orchestrator organizes everything. There's a clear hierarchy: Organization → Tenant → Folder → Resources.

**[VISUAL: Highlight Organization]**

At the top is the Organization. This represents your entire company. It's managed through the UiPath Cloud Portal. An organization can contain multiple tenants. Licensing and billing happen at the organization level.

**[VISUAL: Highlight Tenant]**

Next level down: Tenants. A tenant is an isolated workspace within your organization. Think of it as a completely separate Orchestrator instance, even though they're running on the same platform.

Tenants provide complete data isolation. Each tenant has:
- Its own users and permissions
- Its own robots
- Its own processes, assets, queues
- Logical separation in the database

**[VISUAL: Show example]**

When would you use multiple tenants?

If you're a large company with completely separate business units—say Finance, HR, and Sales—and they should never see each other's data, give each its own tenant.

If you're a service provider with multiple clients, give each client their own tenant.

If you want complete separation between production and non-production, use separate tenants.

The key principle: if they should NEVER access each other's data, use separate tenants.

**[VISUAL: Highlight Folder]**

Within a tenant, you have Folders. Folders organize resources by department, project, or environment. Unlike tenants, folders within the same tenant can see each other (with proper permissions) and share tenant-level resources.

**[VISUAL: Show example]**

Common folder structures:

By department: HR-Folder, Finance-Folder, IT-Folder
By environment: Dev-Folder, Staging-Folder, Prod-Folder
By project: Project-A, Project-B, Project-C

Folders provide granular access control. You can give a developer access to Dev-Folder but not Prod-Folder.

**[VISUAL: Comparison slide]**

Tenant vs Folder—when to use which?

If you need complete data isolation with no data sharing → Use separate tenants.
If you want organization within the same security domain → Use folders.

Think of tenants as separate buildings. Folders are rooms within a building.

**[VISUAL: Show the full hierarchy]**

So the complete picture:
- Organization: Acme Corp
  - Tenant: Production
    - Folder: Finance
      - Robots, Processes, Assets, Queues
    - Folder: HR
      - Robots, Processes, Assets, Queues
  - Tenant: Non-Production
    - Folder: Dev
    - Folder: Test

This structure provides flexibility, security, and organization at scale."

---

## Section 5: Core Components - Robots (3 minutes)

**[VISUAL: Robot types diagram]**

**Presenter:**

"Let's dive into the core components of Orchestrator, starting with Robots.

Robots are the execution agents—the workers that actually run your automations. There are two main types: Unattended and Attended.

**[VISUAL: Show Unattended robot]**

Unattended robots run autonomously without any human supervision. They're typically triggered by schedules or queue items. They run on dedicated machines or virtual machines, often in a server room or data center.

Unattended robots are for back-office automation—things like:
- Processing invoices overnight
- Extracting data from emails every hour
- Updating databases on a schedule
- Clearing queues of work items

These robots can run 24/7. They don't need anyone logged in. They're fully automated.

**[VISUAL: Show Attended robot]**

Attended robots work alongside humans on their workstations. They require a user to be logged in. The user manually starts the automation, and it might interact with the user during execution.

Attended robots are for front-office automation—things like:
- Helping customer service reps look up information
- Assisting with data entry
- Automating repetitive desktop tasks
- Augmenting human work

**[VISUAL: Comparison table]**

Key differences:

Unattended:
- No user presence needed
- Triggered automatically
- Run on dedicated machines
- Can scale by adding machines
- Higher licensing cost

Attended:
- User must be logged in
- Started manually
- Run on user workstations
- Scale limited by number of users
- Lower licensing cost

**[VISUAL: Show robot in Orchestrator]**

In Orchestrator, each robot has:
- A name
- A machine assignment
- A type (Unattended/Attended)
- Access to specific folders
- A status (Online, Offline, Busy)

You manage robots through the Orchestrator UI or API."

---

## Section 6: Core Components - Processes (3 minutes)

**[VISUAL: Process lifecycle diagram]**

**Presenter:**

"Next component: Processes.

A process is a deployed automation workflow. It's the packaged version of what you built in Studio.

**[VISUAL: Show the workflow]**

The lifecycle goes like this:

Step 1: Develop in UiPath Studio. Build your workflow, test it locally, make sure it works.

Step 2: Publish to Orchestrator. This creates a .nupkg file—a package containing your workflow and all dependencies—and uploads it to Orchestrator.

Step 3: Deploy to folders. Publishing makes the process available; deploying puts it in specific folders where robots can access it.

Step 4: Assign to robots. Specify which robots should execute this process.

Step 5: Execute. Trigger the process via schedule, queue, or API.

**[VISUAL: Show process package]**

A process package contains:
- The compiled workflow
- All dependencies and libraries
- Version information
- Configuration

**[VISUAL: Show versioning]**

Processes use semantic versioning: 1.0.0, 1.1.0, 2.0.0.

You can have multiple versions deployed simultaneously. For example:
- Version 1.0.0 in Production Folder
- Version 2.0.0 in Test Folder

This lets you test new versions before promoting to production.

**[VISUAL: Show Orchestrator UI]**

In Orchestrator, you see all your processes. You can:
- View process details and dependencies
- See which folders it's deployed to
- Trigger manual executions
- View execution history
- Update deployment versions

**[VISUAL: Emphasize the point]**

Important distinction: Publishing a process doesn't make it run. It just makes it available. You must explicitly trigger execution."

---

## Section 7: Core Components - Queues (3 minutes)

**[VISUAL: Queue diagram]**

**Presenter:**

"Queues are one of Orchestrator's most powerful features. They enable work distribution across multiple robots.

**[VISUAL: Show the concept]**

Here's the scenario: You have 1,000 invoices to process. You have 10 robots. How do you distribute the work?

Answer: Use a queue.

**[VISUAL: Show the process]**

How it works:

Step 1: A 'dispatcher' process adds all 1,000 invoices to a queue as queue items.

Step 2: Multiple 'performer' robots connect to the queue and pull items.

Step 3: Each robot processes its item independently.

Step 4: The robot reports success or failure back to the queue.

Step 5: If an item fails, it's automatically retried up to a configurable limit.

**[VISUAL: Show queue states]**

Queue items go through these states:
- New: Waiting to be processed
- In Progress: Currently being processed by a robot
- Successful: Completed successfully
- Failed: Failed after all retries

**[VISUAL: Show benefits]**

Why use queues?

Benefit 1: Load balancing. Work is automatically distributed evenly across available robots.

Benefit 2: Resilience. If a robot crashes, another robot can pick up the failed item and retry it.

Benefit 3: Scalability. Need faster processing? Add more robots. They'll automatically start pulling from the queue.

Benefit 4: Transaction handling. Each queue item is a transaction. Partial failures don't corrupt the whole batch.

**[VISUAL: Show Orchestrator UI]**

In Orchestrator, you can:
- View queue status and backlog
- Monitor item processing in real-time
- Set priorities for items
- Configure retry counts
- Review failed items

Queues are essential for scalable, resilient automation."

---

## Section 8: Core Components - Assets (3 minutes)

**[VISUAL: Assets overview]**

**Presenter:**

"Assets store configuration values and credentials that your robots need.

**[VISUAL: Show asset types]**

There are four types of assets:

Text: String values like URLs, file paths, email addresses

Bool: True/False flags for feature toggles

Integer: Numeric values like retry counts, timeouts

Credential: Username/password pairs for authentication

**[VISUAL: Show the key benefit]**

The power of assets: you can change them without redeploying your process.

Let's say your workflow calls an API. The API URL is stored as an asset. In development, the asset points to the test API. In production, it points to the prod API.

Same workflow package, different asset values.

Need to change the API URL? Just update the asset in Orchestrator. No need to modify code, republish, or redeploy.

**[VISUAL: Show environment-specific assets]**

Assets are folder-specific. You can have different values per folder:

Dev Folder: API_URL = 'https://api-test.example.com'
Prod Folder: API_URL = 'https://api.example.com'

This is how you handle environment-specific configuration.

**[VISUAL: Show credential asset]**

Credential assets are special. They store username/password pairs encrypted in Orchestrator. The robot can retrieve and use them, but the password is never visible in logs or the UI.

This is how you handle secrets in UiPath—much better than hard-coding passwords in workflows.

**[VISUAL: Best practices slide]**

Best practices for assets:
- Use them for all environment-specific values
- Use descriptive names (API_URL not URL1)
- Document what each asset is for
- Rotate credential assets regularly
- Never hard-code values in workflows

**[VISUAL: Show usage in workflow]**

In your workflow, you use the 'Get Asset' activity:

```
Get Asset 'API_URL' → apiUrl variable
```

The robot fetches the value from Orchestrator at runtime.

Assets make your automation configurable and portable."

---

## Section 9: Schedules and Monitoring (2 minutes)

**[VISUAL: Schedules overview]**

**Presenter:**

"Two more quick components: Schedules and Monitoring.

**[VISUAL: Show schedule types]**

Schedules automate process execution. You can create:

Time-based schedules: 'Run every day at 8 AM'
Recurring schedules: 'Run every Monday at 9 PM'
Queue-based triggers: 'Start when queue has items'

Schedules assign processes to robots at specified times. This enables lights-out automation—processes run automatically without human intervention.

**[VISUAL: Show monitoring dashboard]**

Monitoring provides visibility into your automation.

Real-time monitoring shows:
- Which robots are online, offline, or busy
- Which processes are currently running
- Queue backlogs
- Recent failures

Historical analytics show:
- Process success rates over time
- Average execution durations
- Resource utilization
- Trends and patterns

**[VISUAL: Show logs]**

Every execution generates detailed logs:
- Start and end times
- Log messages from the workflow
- Screenshots (if enabled)
- Exceptions and stack traces
- Execution context

These logs are invaluable for debugging and auditing.

Orchestrator gives you complete visibility into your automation ecosystem."

---

## Section 10: Navigating the Orchestrator UI (3 minutes)

**[VISUAL: Orchestrator UI screenshot]**

**Presenter:**

"Let's walk through the Orchestrator UI so you know how to navigate it.

**[VISUAL: Highlight top bar]**

At the top, you'll see:
- Organization selector (if you have multiple organizations)
- Tenant selector (if you have multiple tenants)
- User menu for profile and settings

**[VISUAL: Highlight folder dropdown]**

Very important: The folder dropdown. This determines which folder's resources you're viewing. Always be aware of your current folder context.

**[VISUAL: Highlight left sidebar]**

The left sidebar has main sections:

**Admin** (tenant-level):
- Users and permissions
- Licenses
- Settings and configuration

**Monitoring**:
- Dashboards
- Alerts
- Jobs (running and completed)

**Orchestrator** (folder-level):
- Robots
- Processes
- Assets
- Queues
- Triggers (schedules)
- Logs

**Insights**:
- Analytics and reports

**[VISUAL: Show navigation example]**

Typical workflow:
1. Select your tenant
2. Switch to the appropriate folder
3. Navigate to the resource (e.g., Processes)
4. Perform your action

**[VISUAL: Show URL structure]**

For Cloud Orchestrator, URLs look like:
```
https://cloud.uipath.com/[org]/[tenant]/
```

Each tenant has its own URL. Bookmark the ones you use frequently."

---

## Section 11: Best Practices and Summary (2 minutes)

**[VISUAL: Best practices slide]**

**Presenter:**

"Let me leave you with key best practices:

**For tenant structure:**
- Use separate tenants only when you need complete isolation
- Most organizations use one tenant with multiple folders

**For folders:**
- Organize by environment (Dev, Test, Prod) or department
- Use clear naming conventions
- Grant least-privilege access

**For robots:**
- Name robots descriptively (FINANCE-BOT-01, not Robot1)
- Monitor robot health regularly
- Don't over-provision—unused robots cost money

**For processes:**
- Use semantic versioning
- Test in lower environments first
- Clean up old versions periodically

**For queues:**
- Keep queue items focused and small
- Set appropriate retry counts
- Monitor queue backlog

**For assets:**
- Use assets for all environment-specific config
- Rotate credentials regularly
- Document asset purposes

**[VISUAL: Key takeaways slide]**

Key takeaways:

✅ Orchestrator is the central management platform
✅ Hierarchy: Organization → Tenant → Folder → Resources
✅ Cloud is managed by UiPath, On-Premises is self-hosted
✅ Robots execute, Processes define workflows, Queues distribute work, Assets store config
✅ Understanding the architecture is essential for API development

**[VISUAL: Next module preview]**

In the next module, we'll cover Authentication and Authorization—how to connect to Orchestrator using OAuth 2.0, generate access tokens, and authenticate API requests. This is where we start writing Python code against Orchestrator.

For now, I encourage you to log into Orchestrator—use cloud.uipath.com or request access to your organization's instance—and explore the UI. Get familiar with the layout, navigate through folders, look at processes and robots.

Hands-on experience is the best way to internalize this architecture.

Thank you for watching!"

---

## Presenter Notes

### Key Teaching Points
1. **Start with big picture**: Show how Orchestrator fits in UiPath ecosystem
2. **Emphasize hierarchy**: Organization → Tenant → Folder is critical
3. **Distinguish tenant vs folder**: Many get this wrong
4. **Show real UI**: Use screenshots or screen recording of actual Orchestrator
5. **Connect to API usage**: Everything we discuss is accessible via API

### Common Questions to Address
- "What's the difference between tenant and folder?" → Complete isolation vs organizational unit
- "Should I use Cloud or On-Premises?" → Cloud for most, On-Prem for specific requirements
- "How many robots do I need?" → Depends on concurrency and workload
- "Can I mix attended and unattended?" → Yes, but on separate machines

### Demo Tips
- Show actual Orchestrator UI (cloud.uipath.com)
- Navigate through different sections
- Show a process, queue, asset in action
- Point out folder selector and how it changes context

### Troubleshooting Common Issues
- If students can't access Orchestrator: Direct them to trial signup
- If confused about hierarchy: Draw diagram, use analogies
- If unclear about robot types: Show use cases for each

### Time Management
- Section 1-2: Introduction and ecosystem (6 minutes)
- Section 3-4: Deployment and hierarchy (8 minutes)
- Section 5-8: Core components (12 minutes)
- Section 9-11: UI and best practices (4 minutes)

Total: 30 minutes

### Visual Aids to Prepare
- Ecosystem diagram (Studio → Orchestrator → Robots)
- Hierarchy diagram (Org → Tenant → Folder)
- Cloud vs On-Premises comparison
- Screenshots of Orchestrator UI
- Queue flow diagram
- Asset types and examples

### Connection to Python SDK
- Everything shown can be accessed via API
- Next module covers authentication
- Subsequent modules show API usage
- This architectural knowledge is foundation
