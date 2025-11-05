# Video Script: Module 1.2.1.3 - Tenants and Folders

**Duration:** 28 minutes
**Module:** 1.2.1.3
**Prerequisites:** Module 1.2.1.1 (UiPath Orchestrator Architecture)

---

## Section 1: Introduction (2 minutes)

### Visual
- Title slide with module name
- Organizational hierarchy diagram preview

### Script

Welcome to Module 1.2.1.3: Tenants and Folders. In this module, we're going to dive deep into how UiPath Orchestrator organizes resources using a three-level hierarchy: Organizations, Tenants, and Folders.

If you've worked with Orchestrator before, you've probably encountered tenants and folders, but may not fully understand how they work together or when to use each one. By the end of this module, you'll understand the complete organizational structure and be able to design effective tenant and folder strategies for your automation projects.

We'll cover the organizational hierarchy, multi-tenancy strategies, folder permissions, and most importantly—how to work with folders in your API code. This last part is critical because forgetting to specify the folder context is one of the most common errors developers make when working with the Orchestrator API.

Let's get started by understanding the three-level hierarchy.

### Presenter Notes
- Emphasize that this is foundational knowledge for API development
- Mention that folder context errors are very common
- Set expectations that we'll have hands-on code examples

---

## Section 2: The Organizational Hierarchy (3 minutes)

### Visual
- Three-level hierarchy diagram
- Example: Acme Corporation with Dev/Prod tenants and Finance/HR folders
- Highlight each level with explanations

### Script

UiPath Cloud uses a three-level organizational hierarchy: Organization, Tenant, and Folder.

At the top level is the **Organization**. This represents your company—for example, "Acme Corporation." The Organization level handles billing, subscription management, license allocation, and organization-wide settings. Think of it as your company's account with UiPath.

Below the Organization, you have **Tenants**. A Tenant is an isolated workspace for automation resources. Each tenant is completely separate—it has its own users, its own data, its own robots and processes. If you create a robot in the Development tenant, it doesn't exist in the Production tenant. Tenants are isolated from each other by design.

Most companies use tenants to separate environments. You might have a Development tenant for testing, a Staging tenant for pre-production validation, and a Production tenant for live automation. Alternatively, you might use tenants to separate business units—a Finance tenant, an HR tenant, and an IT tenant, each completely independent.

At the lowest level, within each tenant, you have **Folders**. Folders organize resources within a tenant. For example, your Production tenant might have a Finance folder, an HR folder, and an IT folder. Each folder contains its own robots, processes, queues, and assets. Folders enable you to organize resources and control access through role-based permissions.

So to summarize: Organization handles billing and administration, Tenants provide isolation between environments or business units, and Folders organize resources within a tenant. The hierarchy is: Organization → Tenant → Folder → Resources.

Let's look at each level in more detail, starting with Tenants.

### Presenter Notes
- Use concrete examples: "Acme Corporation"
- Emphasize isolation: tenants can't see each other's data
- Preview the section structure: tenants first, then folders

---

## Section 3: Understanding Tenants (4 minutes)

### Visual
- Tenant isolation diagram showing separate databases
- Example tenant strategies (by environment vs by department)
- Multi-tenant authentication flow

### Script

Let's talk about Tenants in detail. A Tenant is an isolated workspace—think of it as a completely separate Orchestrator instance within your Organization.

Tenants are completely isolated from each other. Each tenant has its own database, its own users and permissions, its own security policies, and its own audit logs. If you're a user in the Development tenant, that doesn't automatically give you access to the Production tenant. You need separate credentials for each tenant.

This isolation is powerful. It means you can have completely different security policies in Dev versus Prod. You can grant developers full admin access in Dev, but restrict them to view-only in Prod. Each tenant is independent.

There are several common strategies for organizing tenants:

The most common is **environment-based separation**. You create a Development tenant, a Staging tenant, and a Production tenant. This gives you a clear promotion path: you test in Dev, validate in Staging, and deploy to Prod. Each environment is completely isolated, so testing in Dev can't possibly affect Prod.

Another strategy is **business unit separation**. If your Finance department and HR department are completely independent, you might create separate tenants for each. This gives each department complete autonomy—their own administrators, their own policies, their own data.

A third strategy, used by Managed Service Providers, is **customer separation**. If you're an MSP providing RPA services to multiple clients, you'd create a separate tenant for each client. This ensures complete data isolation between customers.

Now, here's an important point about authentication: each tenant requires separate authentication. When you authenticate to get an access token, that token is specific to one tenant. If you need to access multiple tenants, you need separate credentials and separate tokens for each. We'll see code examples of this later.

For most companies, the environment-based approach works best: Dev, Staging, and Production tenants. This gives you clear boundaries and a standard promotion process.

### Presenter Notes
- Emphasize complete isolation—critical for security
- Use real examples: "If you delete a queue in Dev, Prod is unaffected"
- Mention that most questions are about multi-tenant strategies

---

## Section 4: Understanding Folders (4 minutes)

### Visual
- Folder structure diagram within a tenant
- Folder permissions matrix (user x folder x role)
- Code example showing folder context

### Script

Now let's talk about Folders. While Tenants provide isolation, Folders provide organization within a tenant.

A Folder is a grouping of resources within a tenant. Each folder can contain robots, processes, queues, and assets. Resources in one folder are separate from resources in another folder within the same tenant.

Let's look at an example. Say you have a Production tenant, and within it you create three folders: Finance, HR, and IT. The Finance folder contains robots that process invoices, an Invoice Processing process, and an Invoice Queue. The HR folder contains robots for employee onboarding, and the IT folder contains helpdesk automation robots. Each folder is focused on a specific department's automation needs.

The key benefit of folders is **access control**. Folders use Role-Based Access Control, or RBAC. You can assign users different roles in different folders. For example, Alice might be an Admin in the Finance folder—she has full control, can create and delete resources. But in the HR folder, Alice might have only View permissions—she can see what's there, but can't modify anything. And she might have no access at all to the IT folder.

This enables fine-grained security. Each department can manage their own folder, and users only have access to the folders they need. This follows the principle of least privilege—giving users only the minimum permissions they need to do their job.

There are two types of folders in Orchestrator: Classic folders and Modern folders. Classic folders, used in older versions of Orchestrator, have basic permissions with just Admin and User roles. Modern folders, introduced in version 2020.10, have granular RBAC with many built-in roles, support for folder hierarchies with parent and child folders, and permission inheritance. For new deployments, you should use Modern folders—they give you much better access control and organization capabilities.

Now here's the critical part for API development: when you make an API call, you must specify which folder you're accessing. Unlike the web UI where you can select a folder from a dropdown, in the API you must explicitly tell Orchestrator which folder you want to work with. We do this using a special header called X-UIPATH-OrganizationUnitId. If you forget this header, your API calls will fail or return unexpected results. We'll look at code examples in the next section.

### Presenter Notes
- Use concrete examples: Finance, HR, IT folders
- Emphasize the access control benefit
- Foreshadow the API folder context section
- Mention that forgetting folder context is a common error

---

## Section 5: Multi-Tenancy Strategies (3 minutes)

### Visual
- Side-by-side comparison: Environment-based vs Department-based
- Decision tree for choosing a strategy
- Real-world examples

### Script

Let's talk about how to design your tenant and folder structure. The question is: how many tenants should you create, and how should you organize folders within them?

The most common and recommended strategy is **environment-based tenants with department folders**. You create three tenants: Development, Staging, and Production. Within each tenant, you create folders for each department: Finance, HR, IT, and so on.

This approach has several benefits. First, environment isolation is critical for stability—you can test freely in Dev without any risk to Prod. Second, it provides a clear promotion path: you develop in Dev, test in Staging, deploy to Prod. The folder structure is the same across all three tenants, which makes promotion straightforward. Third, departments share the same environment policies—everyone in Prod, regardless of department, follows the same security and deployment policies.

An alternative strategy is **department-based tenants with environment folders**. You create three tenants: Finance, HR, and IT. Within each tenant, you create folders for Dev, Staging, and Prod.

This approach works if your departments are highly independent. Maybe Finance and HR have completely different security policies, different administrators, and don't collaborate on automation projects. In that case, giving each department its own tenant makes sense. But this is less common—most companies prefer environment-based tenants.

As a general rule: use Tenants for isolation, and use Folders for organization. If you need complete data isolation, different security policies, or customer separation, use separate tenants. If you just need to organize resources or control access for different teams, use folders within the same tenant.

For example, don't create separate tenants for different projects within the same department—use folders instead. But do create separate tenants for Dev versus Prod—that isolation is critical.

### Presenter Notes
- Emphasize that environment-based is most common
- Provide clear decision criteria
- Mention that this is a design decision you make early on

---

## Section 6: Folder Permissions and RBAC (3 minutes)

### Visual
- Permission matrix showing users and their roles in different folders
- Code example of permission error handling
- Common roles diagram (Admin, Edit, View)

### Script

Let's dive into folder permissions. Folders use Role-Based Access Control, which means users are assigned roles, and each role has specific permissions.

There are several built-in roles. The Admin role has full control—create, read, update, delete resources, and manage folder settings. The Edit role can create and modify resources but can't manage folder settings. The View role is read-only—users can see resources but can't modify them. There are also more specific roles like Robot Create, Queue User, and so on.

The powerful part is that a user can have different roles in different folders. Let's look at an example.

Alice is the Finance Manager. She has the Admin role in the Finance folder—she can manage all Finance automation. But she also needs to monitor HR automation, so she has the View role in the HR folder—she can see what's running, but can't modify anything. And she has no access at all to the IT folder.

Bob is a Finance Developer. He has the Edit role in the Finance folder—he can create and modify processes, but can't manage folder settings or permissions. That's reserved for Alice as the Admin.

Carol is the CEO. She has the View role in all folders—she can monitor all automation across the company, but can't modify anything. This follows the principle of least privilege.

When you make an API call with a specific folder context, Orchestrator checks your permissions for that folder. If you don't have access, you get a 403 Forbidden error. Your code should handle these permission errors gracefully.

Best practice: give users the minimum permissions they need. Use the View role for monitoring and reporting. Reserve the Admin role for team leads and managers who need to manage permissions. Use Edit for developers who need to create and modify resources.

### Presenter Notes
- Use concrete examples with Alice, Bob, Carol
- Emphasize least privilege principle
- Mention error handling for permission errors

---

## Section 7: Using Folder Context in API Calls (5 minutes)

### Visual
- Code editor showing API call with X-UIPATH-OrganizationUnitId header
- Split screen: call without header (error) vs call with header (success)
- Live demo of getting folder ID and using it

### Script

Now let's talk about the most important part for developers: using folder context in your API calls. This is where many developers make mistakes, so pay close attention.

When you make an API call to Orchestrator, you must specify which folder you're accessing. You do this using the X-UIPATH-OrganizationUnitId header. This header contains the folder ID—a numeric identifier for the folder.

Let's look at a code example. Here's an API call to get all robots:

```python
headers = {
    'Authorization': f'Bearer {access_token}',
    'X-UIPATH-OrganizationUnitId': '12345'
}

response = requests.get(
    'https://cloud.uipath.com/org/tenant/orchestrator_/odata/Robots',
    headers=headers
)
```

Notice the X-UIPATH-OrganizationUnitId header with the value 12345. This tells Orchestrator to return robots from folder with ID 12345. Without this header, the request would fail or return unexpected results.

Now, you might be asking: where do I get the folder ID? You retrieve it using the Folders endpoint:

```python
# Get all folders
response = requests.get(
    f'{base_url}/odata/Folders',
    headers={'Authorization': f'Bearer {token}'}
)

folders = response.json()['value']

# Find the Finance folder
finance_folder = next(
    f for f in folders
    if f['DisplayName'] == 'Finance'
)

folder_id = finance_folder['Id']  # This is what you use in the header
```

You call GET /odata/Folders, which returns all folders you have access to. Each folder has an Id field and a DisplayName field. You find the folder you want by name, extract its Id, and use that in subsequent API calls.

Best practice: do this lookup once at the start of your script, not on every API call. Cache the folder IDs so you're not constantly querying for them.

Let's see a complete example:

```python
import requests

class OrchestratorClient:
    def __init__(self, base_url, token):
        self.base_url = base_url
        self.token = token
        # Get folders once and cache them
        self.folders = self._get_folders()

    def _get_folders(self):
        response = requests.get(
            f'{self.base_url}/odata/Folders',
            headers={'Authorization': f'Bearer {self.token}'}
        )
        # Return a dict: folder name -> folder ID
        return {
            f['DisplayName']: f['Id']
            for f in response.json()['value']
        }

    def get_robots(self, folder_name):
        folder_id = self.folders[folder_name]
        response = requests.get(
            f'{self.base_url}/odata/Robots',
            headers={
                'Authorization': f'Bearer {self.token}',
                'X-UIPATH-OrganizationUnitId': str(folder_id)
            }
        )
        return response.json()['value']
```

This client class fetches folders once in the constructor and caches them in a dictionary. Then when you call get_robots with a folder name, it looks up the folder ID and includes it in the request header.

This is the pattern you should follow: lookup folders once, cache the mappings, and always include the folder context in your API calls.

### Presenter Notes
- Emphasize that this is critical—many errors come from missing this
- Walk through the code slowly
- Mention that we'll practice this in exercises
- Show both the error case and the success case

---

## Section 8: Modern vs Classic Folders (2 minutes)

### Visual
- Comparison table: Classic vs Modern
- Migration path diagram
- Modern folder hierarchy example

### Script

Before we move on, let's briefly discuss Modern folders versus Classic folders.

Orchestrator has two folder types. Classic folders are the original implementation, used in versions before 2020.10. They have basic permissions with just Admin and User roles, limited nesting capabilities, and simple access control.

Modern folders, introduced in version 2020.10, are the current recommended approach. They have granular RBAC with many built-in roles and the ability to create custom roles. They support full folder hierarchies—you can have parent folders and child folders, with permissions inherited from parent to child. And they have better access control overall.

If you're starting a new Orchestrator deployment, use Modern folders. They give you much better organization and security capabilities. If you're on an older Orchestrator version with Classic folders, plan to migrate to Modern folders when you upgrade.

The good news is that from an API perspective, the folder context mechanism works the same way for both types—you always use the X-UIPATH-OrganizationUnitId header. The differences are in the permission model and folder structure, which you manage through the Orchestrator UI.

### Presenter Notes
- Keep this brief—just awareness of the two types
- Emphasize Modern is recommended
- Mention that API usage is the same

---

## Section 9: Best Practices (3 minutes)

### Visual
- Best practices checklist
- Code examples showing good vs bad patterns
- Folder naming conventions

### Script

Let's talk about best practices for working with tenants and folders.

First, **tenant strategy**: Use tenants for environment separation. Create Dev, Staging, and Prod tenants. Each environment is isolated, which protects production and enables safe testing. Don't use tenants for simple resource grouping—use folders for that.

Second, **folder strategy**: Organize folders by department, project, or team. Use clear, descriptive names. For example: Finance, Finance-Invoicing, Finance-Reporting, HR, HR-Onboarding. Keep the structure aligned with your organization's structure.

Third, **permissions**: Apply the principle of least privilege. Give users only the permissions they need. Use View for monitoring, Edit for development, Admin for team leads. Audit permissions regularly—remove access when people change roles.

Fourth, **API development**: Always include the X-UIPATH-OrganizationUnitId header in your API calls. Look up folder IDs dynamically by name—don't hard-code them, because folder IDs can change if folders are recreated. Cache folder mappings to avoid repeated lookups.

Fifth, **multi-tenant access**: If you need to access multiple tenants, keep credentials and tokens separate. Use clear naming: DEV_CLIENT_ID, PROD_CLIENT_ID. Log which tenant you're authenticated to. Consider creating separate client classes for each tenant.

Sixth, **error handling**: Handle folder permission errors gracefully. If you get a 403 Forbidden error, it means you don't have access to that folder. Your code should catch this and handle it appropriately—maybe skip that folder, or notify an administrator.

These best practices will help you avoid common pitfalls and create maintainable, secure automation solutions.

### Presenter Notes
- Present these as actionable guidelines
- Emphasize that these come from real-world experience
- Mention that we'll see these practices in code examples

---

## Section 10: Common Pitfalls (2 minutes)

### Visual
- Code examples of each pitfall with error messages
- "Before and After" corrections

### Script

Let's look at some common pitfalls developers encounter with tenants and folders.

**Pitfall number one: Missing folder context.** This is the most common error. You make an API call and forget to include the X-UIPATH-OrganizationUnitId header. The request fails with an error like "Folder context is required" or returns unexpected results. Always include this header.

**Pitfall number two: Hard-coding folder IDs.** You look up a folder ID once, hard-code it in your script as a constant, and everything works. Then later, someone deletes and recreates the folder, which generates a new ID. Your script breaks. Instead, look up folders by name dynamically. Names are stable, IDs are not.

**Pitfall number three: Tenant confusion.** You authenticate to the Dev tenant but then use the Prod base URL. Or you mix up credentials from different tenants. Always keep tenant contexts completely separate. Use clear variable names, and log which tenant you're working with.

**Pitfall number four: Ignoring permission errors.** Your script gets a 403 Forbidden error and crashes. Better approach: catch permission errors, log them, and handle them gracefully. Maybe your script processes multiple folders—if one fails due to permissions, log it and continue with the others.

These pitfalls are easy to avoid once you're aware of them. The key is to be explicit: always specify folder context, look up IDs dynamically, keep tenant contexts separate, and handle errors gracefully.

### Presenter Notes
- Use concrete error messages
- Emphasize that these are common—everyone encounters them
- Show how to fix each one

---

## Section 11: Hands-on Demo (3 minutes)

### Visual
- Live code execution showing:
  - Getting folders
  - Getting robots from specific folder
  - Handling multiple folders

### Script

Let's look at a complete hands-on example. I'm going to show you a script that authenticates to Orchestrator, retrieves all folders, and gets robots from each folder.

```python
import requests
import os

# Configuration
BASE_URL = os.getenv('ORCHESTRATOR_URL')
CLIENT_ID = os.getenv('CLIENT_ID')
CLIENT_SECRET = os.getenv('CLIENT_SECRET')
TOKEN_URL = os.getenv('TOKEN_URL')

# 1. Get access token
def get_token():
    response = requests.post(TOKEN_URL, data={
        'grant_type': 'client_credentials',
        'client_id': CLIENT_ID,
        'client_secret': CLIENT_SECRET,
        'scope': 'OR.Robots OR.Folders'
    })
    return response.json()['access_token']

# 2. Get all folders
def get_folders(base_url, token):
    response = requests.get(
        f'{base_url}/odata/Folders',
        headers={'Authorization': f'Bearer {token}'}
    )
    return response.json()['value']

# 3. Get robots in a specific folder
def get_robots(base_url, token, folder_id):
    response = requests.get(
        f'{base_url}/odata/Robots',
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id)
        }
    )
    return response.json()['value']

# Main execution
def main():
    # Authenticate
    token = get_token()
    print("✓ Authenticated successfully")

    # Get folders
    folders = get_folders(BASE_URL, token)
    print(f"✓ Found {len(folders)} folders")

    # Process each folder
    for folder in folders:
        folder_name = folder['DisplayName']
        folder_id = folder['Id']

        print(f"\nProcessing folder: {folder_name} (ID: {folder_id})")

        try:
            robots = get_robots(BASE_URL, token, folder_id)
            print(f"  - Found {len(robots)} robots")

            for robot in robots:
                print(f"    • {robot['Name']} ({robot['Type']})")

        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 403:
                print(f"  - No access to this folder")
            else:
                print(f"  - Error: {e}")

if __name__ == '__main__':
    main()
```

This script demonstrates all the concepts we've covered. It authenticates, retrieves folders, and for each folder, it gets the robots. Notice how it includes the folder ID in the X-UIPATH-OrganizationUnitId header. It also handles permission errors gracefully—if you don't have access to a folder, it logs that and continues with the next folder.

This is a pattern you can adapt for your own scripts. The key elements are: authenticate once, get folders once, cache the folder data, and always include folder context in your API calls.

### Presenter Notes
- Walk through the code section by section
- Run the code if possible to show real output
- Emphasize the error handling
- Point out the folder context header

---

## Section 12: Summary and Next Steps (2 minutes)

### Visual
- Summary slide with key points
- Next module preview
- Resources and links

### Script

Let's summarize what we've covered in this module.

We learned about the three-level organizational hierarchy in UiPath: Organization, Tenant, and Folder. Organizations handle billing and subscriptions. Tenants provide isolated workspaces for different environments or business units. Folders organize resources within a tenant and enable role-based access control.

We discussed multi-tenancy strategies. The most common approach is environment-based tenants—Dev, Staging, Prod—with department-based folders within each tenant. This provides clear environment boundaries and a standard promotion path.

We covered folder permissions using RBAC. Users can have different roles in different folders, enabling fine-grained access control.

Most importantly for developers, we learned how to use folder context in API calls. Always include the X-UIPATH-OrganizationUnitId header with the folder ID. Look up folder IDs dynamically by name and cache them for reuse.

We discussed best practices: use tenants for isolation, folders for organization, always specify folder context, and handle permission errors gracefully. And we looked at common pitfalls like missing folder context and hard-coding folder IDs.

You're now ready to work with multi-tenant Orchestrator environments and properly structure your API code with folder context. This is foundational knowledge that you'll use in every API script you write.

In the next module, we'll compare Cloud versus On-Premises deployments of Orchestrator. We'll look at differences in API endpoints, authentication, and when to use each deployment model.

Thanks for watching, and I'll see you in the next module!

### Presenter Notes
- Quick recap of main points
- Emphasize the folder context as the key takeaway
- Encourage students to practice with the quiz and exercises
- Preview next module briefly

---

## Additional Teaching Notes

### Common Student Questions

**Q: Can I access resources across folders?**
A: Not directly. Each API call targets a specific folder via the X-UIPATH-OrganizationUnitId header. To access resources from multiple folders, you need to make separate API calls for each folder.

**Q: How do I move resources between folders?**
A: Some resources like processes can be deployed to multiple folders. Others may need to be exported from one folder and imported to another. The Orchestrator UI provides folder management capabilities.

**Q: What happens if I don't specify a folder in my API call?**
A: The behavior depends on the endpoint and Orchestrator version. Some endpoints will return an error, others might default to a specific folder. Always explicitly specify the folder to avoid unexpected behavior.

**Q: Can a robot belong to multiple folders?**
A: Modern folders support robot accounts that can be assigned to multiple folders. Classic folders typically have robots associated with a single folder.

**Q: Should I use environment variables for folder IDs?**
A: It's better to look up folders dynamically by name. Folder IDs can change if folders are deleted and recreated, but names are more stable. If you must use environment variables, use folder names, not IDs.

### Demo Prerequisites
- Access to UiPath Cloud Orchestrator
- At least 2 folders created in a tenant
- External Application credentials with appropriate scopes
- Python environment with requests library

### Additional Examples
The script examples in this module can be extended with:
- Filtering folders by specific criteria
- Creating folder-specific reports
- Comparing resources across folders
- Automated folder permission auditing

### Assessment Tips
- The quiz includes questions on organizational hierarchy, folder context, and best practices
- Emphasize that real-world API development always requires folder context
- Practice exercises should include multi-folder scenarios and error handling
