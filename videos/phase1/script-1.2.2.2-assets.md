# Video Script: Module 1.2.2.2 - Assets (Variables and Credentials)

**Duration:** 28 minutes
**Module:** 1.2.2.2
**Prerequisites:** Module 1.2.2.1 (Processes and Jobs), Module 1.1.3.2 (Secret Handling)

---

## Section 1: Introduction (2 minutes)

### Visual
- Title slide
- Configuration management diagram
- Hard-coded vs Asset-based comparison

### Script

Welcome to Module 1.2.2.2: Assets. In this module, we're going to learn how to manage configuration values and credentials in UiPath Orchestrator using Assets.

If you've built any automation, you've encountered the configuration problem: your workflow needs an API URL, a database connection string, file paths, and maybe some passwords. Where do you put these values?

The worst approach is hard-coding them directly in your workflow—if the URL changes, you have to republish the entire workflow. And hard-coding passwords is a security nightmare.

The solution is Assets. Assets are Orchestrator's configuration and secrets management system. They let you store configuration values and credentials centrally, outside your workflows. Your workflows read these values at runtime from Orchestrator. When a value needs to change, you update the asset—no workflow republishing required.

Assets also enable environment-specific configuration. Your Dev environment can have one database URL, Staging another, and Production a third—all using the same workflow, just different asset values per folder.

In this module, you'll learn about asset types—Text assets for configuration and Credential assets for secrets. You'll see how to retrieve asset values via the API, how to update them, and most importantly, how to handle credentials securely.

Let's start by understanding what assets are and why you need them.

### Presenter Notes
- Emphasize the configuration management problem
- Set expectation that assets solve multiple problems
- Preview that credentials have special secure handling

---

## Section 2: Understanding Assets (3 minutes)

### Visual
- Asset types diagram
- Example assets in Orchestrator UI
- Configuration lifecycle: create → read → update

### Script

Assets are named values stored in Orchestrator that your robots and API scripts can read. Think of them as environment variables combined with a secrets vault.

There are four value types. Text assets store plain text strings—things like URLs, email addresses, file paths, server names. Bool assets store true/false flags—useful for feature toggles or enabling debug mode. Integer assets store numeric values like thresholds, retry counts, or timeouts. And Credential assets store username and password pairs—these are special because the password is encrypted.

The two most important types are Text for configuration and Credential for secrets. We'll focus on these.

Assets have a scope—either Global or Per Robot. Global scope means one value shared by all robots. For example, if you have an asset called "API_URL" with Global scope and the value "https://api.example.com", every robot that reads this asset gets that same URL. Per Robot scope means each robot can have a different value. For example, an asset called "LocalFolder" might be "C:\Data\Robot01" for Robot 01, "C:\Data\Robot02" for Robot 02, and so on. Each robot gets its own value.

Most assets are Global scope—they're configuration that all robots share. Per Robot is less common, used for machine-specific settings like local file paths or printer names.

Assets belong to folders. When you create an asset, you create it in a specific folder, and it's accessible to workflows running in that folder. This enables environment-specific configuration—the Dev folder has one set of asset values, the Prod folder has different values.

Let me show you some examples of when to use each asset type. For an API base URL—that's a Text asset, Global scope. Everyone uses the same API. For a database login—that's a Credential asset, Global scope. The username and password are the same for all robots, but the password needs to be encrypted. For a local downloads folder—that's a Text asset, Per Robot scope, because each robot machine has a different path.

Assets solve the configuration management problem elegantly. No hard-coded values, easy to change, environment-specific, and secure credential storage.

Now let's look at how to work with assets via the API.

### Presenter Notes
- Use concrete examples for each asset type
- Emphasize Global vs Per Robot distinction
- Preview that credential passwords are encrypted

---

## Section 3: Retrieving Text Assets (4 minutes)

### Visual
- Code editor showing get_asset function
- API endpoint structure breakdown
- Example output from asset retrieval

### Script

Let's see how to retrieve asset values via the API. We'll start with Text assets, which are the simplest.

To get a Global asset by name, you use a special OData function called GetAssetByName. The full endpoint is: GET /odata/Assets/UiPath.Server.Configuration.OData.GetAssetByName, and you pass the asset name as a parameter in the URL.

Here's a complete example:

```python
def get_asset(base_url, token, folder_id, asset_name):
    """Get a global asset value"""

    response = requests.get(
        f"{base_url}/odata/Assets/UiPath.Server.Configuration.OData.GetAssetByName(assetName='{asset_name}')",
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id)
        }
    )

    response.raise_for_status()
    asset = response.json()
    return asset['Value']
```

Let's break this down. The URL includes the base URL, then /odata/Assets/, then the function name UiPath.Server.Configuration.OData.GetAssetByName, then in parentheses the parameter assetName equals the asset name in single quotes. Yes, it's a long endpoint name—that's the OData standard. Don't memorize it, just copy it.

As always, we include the Authorization header with our bearer token and the X-UIPATH-OrganizationUnitId header to specify which folder. Assets are folder-specific, so folder context is required.

The response is a JSON object representing the asset. It has fields like Name, Value, ValueType, ValueScope, and more. We extract the Value field—that's the actual configuration value.

Usage is simple:

```python
api_url = get_asset(base_url, token, folder_id, 'API_URL')
print(f"API URL: {api_url}")  # "https://api.example.com"

max_retries = int(get_asset(base_url, token, folder_id, 'MaxRetries'))
print(f"Max Retries: {max_retries}")  # 5
```

For Integer assets, remember to convert the value to an int—Orchestrator returns everything as strings in the Value field.

You can also list all assets in a folder using GET /odata/Assets without any parameters. This returns an array of all assets. You might do this to get an overview of what assets exist, or to find an asset's Id for update operations.

That's Text assets. Now let's look at Credential assets, which have special handling.

### Presenter Notes
- Walk through the URL structure slowly
- Emphasize folder context requirement
- Show how to handle Integer assets
- Preview that Credentials are similar but with differences

---

## Section 4: Handling Credential Assets (5 minutes)

### Visual
- Credential asset structure
- Encrypted password visualization
- Security design explanation diagram

### Script

Credential assets store username and password pairs, but with a critical difference: the password is encrypted.

Let's retrieve a credential asset:

```python
def get_credential(base_url, token, folder_id, asset_name):
    """Get a credential asset"""

    response = requests.get(
        f"{base_url}/odata/Assets/UiPath.Server.Configuration.OData.GetAssetByName(assetName='{asset_name}')",
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id)
        }
    )

    response.raise_for_status()
    asset = response.json()

    return {
        'username': asset.get('CredentialUsername'),
        'password': asset.get('CredentialPassword')
    }
```

The endpoint is the same—GetAssetByName. But instead of the Value field, we extract CredentialUsername and CredentialPassword.

Now here's the critical part: let's see what this returns:

```python
db_cred = get_credential(base_url, token, folder_id, 'DB_Login')
print(f"Username: {db_cred['username']}")  # "sa"
print(f"Password: {db_cred['password']}")  # "Rz4fG7...encrypted string..."
```

The username is plain text—you can read it. But the password is an encrypted string. You cannot decrypt this password via the API. This is not a limitation—it's intentional security design.

Why? Because allowing plain text password retrieval via API would be a security risk. Anyone with API access could harvest all credentials. API calls might be logged, exposing passwords. Scripts might inadvertently print passwords to logs. All of this would be a security disaster.

Instead, Orchestrator encrypts credential passwords and only allows robots—during workflow execution—to decrypt them. When a robot runs a workflow that uses Get Credential activity, the robot securely decrypts the password and uses it. But external scripts cannot decrypt it.

So what can you do with credentials via the API? You can create credentials, update them, delete them, read the username, and see that a password exists. But you cannot extract plain text passwords. If you need to programmatically use a credential—say, connect to a database—you have two options. One, have a robot do it—start a job that performs the database operation. Two, manage the credential separately from Orchestrator—perhaps use environment variables in your Python script. But don't expect to extract Orchestrator credentials for use in Python—that's not possible and for good security reasons.

This might seem restrictive, but it's actually a strength. Your credentials are protected. Even if someone compromises your API token, they can't harvest all your passwords. The security model is: Orchestrator stores credentials, robots use credentials, humans and scripts manage credentials but don't access values.

Let's move on to updating assets.

### Presenter Notes
- Clearly explain that encrypted passwords are intentional, not a bug
- Emphasize security benefits
- Give alternatives for Python scripts that need credentials
- Reiterate that this protects the organization

---

## Section 5: Updating and Creating Assets (4 minutes)

### Visual
- Code showing update workflow: get Id → update
- PUT vs PATCH comparison
- Create asset payload structure

### Script

To update an asset value, you use PUT or PATCH to the /odata/Assets endpoint with the asset's Id.

The process has two steps: first, get the asset's Id by name. Second, update using that Id.

```python
def update_asset(base_url, token, folder_id, asset_name, new_value):
    """Update an asset's value"""

    # Step 1: Find asset by name to get Id
    assets_response = requests.get(
        f'{base_url}/odata/Assets',
        params={'$filter': f"Name eq '{asset_name}'"},
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id)
        }
    )

    assets = assets_response.json()['value']
    if not assets:
        raise ValueError(f"Asset '{asset_name}' not found")

    asset_id = assets[0]['Id']

    # Step 2: Update using PATCH
    response = requests.patch(
        f'{base_url}/odata/Assets({asset_id})',
        json={'Value': new_value},
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id),
            'Content-Type': 'application/json'
        }
    )

    response.raise_for_status()
    return response.json()
```

Step 1: We query /odata/Assets with an OData filter to find the asset by name. Assets have a Name field and an Id field. The Name is what you use in workflows—human readable, like "API_URL". The Id is a numeric identifier—what the API uses for updates.

Step 2: We PATCH to /odata/Assets with the Id in parentheses. We send a JSON payload with the new Value. PATCH is a partial update—we're only changing the Value field, leaving everything else as-is.

You can also use PUT for a full update, where you send the entire asset object with all fields. PATCH is simpler when you're just changing the value.

To create a new asset:

```python
def create_asset(base_url, token, folder_id, name, value, value_type='Text', scope='Global'):
    """Create a new asset"""

    payload = {
        'Name': name,
        'ValueType': value_type,
        'ValueScope': scope,
        'Value': value,
        'Description': ''
    }

    response = requests.post(
        f'{base_url}/odata/Assets',
        json=payload,
        headers={
            'Authorization': f'Bearer {token}',
            'X-UIPATH-OrganizationUnitId': str(folder_id),
            'Content-Type': 'application/json'
        }
    )

    response.raise_for_status()
    return response.json()

# Usage
create_asset(base_url, token, folder_id, 'API_URL', 'https://api.example.com')
```

POST to /odata/Assets with a payload containing Name, ValueType, ValueScope, and Value. The asset is created in the folder specified by the folder context header.

For Credential assets, the payload is slightly different—you provide CredentialUsername and CredentialPassword instead of Value. When you create a credential, you provide the password in plain text, and Orchestrator encrypts it.

This gives you full CRUD operations: Create with POST, Read with GET or GetAssetByName, Update with PATCH or PUT, Delete with DELETE. Standard REST operations.

### Presenter Notes
- Emphasize two-step process for updates
- Show PATCH as simpler than PUT
- Mention credential creation with plain text password
- Note that deleting is straightforward DELETE

---

## Section 6: Environment Configuration Strategy (3 minutes)

### Visual
- Multi-environment diagram: Dev, Staging, Prod folders
- Same workflow, different asset values
- Deployment flow visualization

### Script

One of the most powerful uses of assets is environment-specific configuration. Let me show you a best practice pattern.

Let's say you have three environments: Development, Staging, and Production. Each has its own Orchestrator folder. And each environment connects to a different API and database.

The naive approach is to create different workflows for each environment, or to pass environment-specific values as input arguments every time. But there's a better way.

Create assets with the same names in each folder, but with different values:

Development folder:
- Asset "API_URL" value: "https://api.dev.example.com"
- Asset "DB_Connection" value: "Server=dev-db;Database=..."

Staging folder:
- Asset "API_URL" value: "https://api.staging.example.com"
- Asset "DB_Connection" value: "Server=staging-db;Database=..."

Production folder:
- Asset "API_URL" value: "https://api.prod.example.com"
- Asset "DB_Connection" value: "Server=prod-db;Database=..."

Your workflow always reads the asset named "API_URL". But it gets a different value depending on which folder it runs in. When running in Dev, it gets the Dev API URL. When running in Prod, it gets the Prod API URL.

This means you can deploy the exact same workflow to all three environments. No code changes, no republishing. The workflow is environment-agnostic—it just reads "API_URL" and gets the right value for wherever it's running.

This pattern is incredibly powerful. It makes promoting code from Dev to Staging to Prod trivial. You publish once, deploy to Dev, test it, then deploy that same package to Staging, test it, then deploy to Prod. No changes, no variation, no risk of introducing bugs during promotion.

And from an API perspective, if you need to change an API URL in Prod, you update the "API_URL" asset in the Prod folder. The workflow immediately sees the new value next time it runs. No republishing, no downtime.

This is the standard pattern for enterprise automation. Use it.

### Presenter Notes
- Emphasize that this is a best practice
- Use concrete example: API URLs per environment
- Explain the promotion workflow benefits
- Mention that this is how enterprises manage environments

---

## Section 7: Security Best Practices (3 minutes)

### Visual
- Security checklist
- Text vs Credential decision tree
- API key storage example

### Script

Let's talk about security best practices when working with assets.

Rule number one: Use Credential assets for all secrets—passwords, API keys, database credentials, service account credentials. Never store secrets in Text assets. Text assets are not encrypted. They're visible in the Orchestrator UI to anyone with access. They appear in logs if you print them. Credential assets encrypt the password, protecting it.

Here's a specific example: storing an API key. You might think, "It's just a key, not a full credential, so I'll use a Text asset." Wrong. API keys are secrets. Store them as Credential assets. Use a dummy username like "api_key" and put the API key as the password:

```python
create_asset(
    base_url, token, folder_id,
    name='External_API_Key',
    value={'username': 'api_key', 'password': 'sk_live_abc123...'},
    value_type='Credential',
    scope='Global'
)
```

When your robot workflow retrieves this credential, it gets the API key from the password field. It's encrypted in storage, protected from casual viewing, and won't appear in logs.

Rule number two: Never log credential values—even the encrypted ones. Don't print passwords to console or write them to files. Even though they're encrypted, you're establishing a bad pattern. Just don't log credentials.

Rule number three: Rotate credentials regularly. Passwords shouldn't live forever. Have a policy for rotating database passwords, API keys, and service account credentials. Use your API scripts to automate rotation—update the asset value on a schedule.

Rule number four: Apply least privilege to asset access. Use folder permissions to control who can view and edit assets. Not everyone needs to edit production credentials. Use role-based access control.

Rule number five: Audit asset changes. Orchestrator logs who changed which asset when. Review these logs periodically to ensure no unauthorized changes.

These practices protect your organization from credential leaks, unauthorized access, and security incidents. Assets are secure by design, but you need to use them correctly.

### Presenter Notes
- Emphasize Text vs Credential distinction
- Show concrete API key example
- Mention rotation and auditing
- Stress that security is a combination of tool and practice

---

## Section 8: Practical Patterns (3 minutes)

### Visual
- Configuration class code
- Bulk update example
- Caching pattern visualization

### Script

Let me show you some practical patterns for working with assets in your Python scripts.

Pattern one: Configuration class with caching. Instead of calling get_asset every time you need a configuration value, create a configuration class that fetches and caches values:

```python
class Config:
    def __init__(self, base_url, token, folder_id):
        self.base_url = base_url
        self.token = token
        self.folder_id = folder_id
        self._cache = {}

    def get(self, asset_name):
        if asset_name not in self._cache:
            self._cache[asset_name] = self._fetch_asset(asset_name)
        return self._cache[asset_name]

    def _fetch_asset(self, asset_name):
        # ... GetAssetByName call ...
        return asset['Value']

# Usage
config = Config(base_url, token, folder_id)
api_url = config.get('API_URL')  # Fetches from Orchestrator
max_retries = int(config.get('MaxRetries'))  # Fetches from Orchestrator
api_url_again = config.get('API_URL')  # Returns cached value
```

This pattern avoids repeated API calls. You fetch each asset once and cache it. Much more efficient than calling the API every time.

Pattern two: Bulk asset update. If you need to update multiple assets—maybe you're refreshing all production URLs—batch them together:

```python
updates = {
    'API_URL': 'https://api.newdomain.com',
    'DB_Connection': 'Server=new-db;...',
    'Email_Sender': 'noreply@newdomain.com'
}

for asset_name, new_value in updates.items():
    update_asset(base_url, token, folder_id, asset_name, new_value)
    print(f"Updated {asset_name}")
```

This is useful for environment migrations or when multiple related values need to change together.

Pattern three: Environment migration. If you're setting up a new environment, copy assets from an existing one:

```python
def copy_assets(source_folder_id, target_folder_id):
    # Get all assets from source
    source_assets = get_all_assets(base_url, token, source_folder_id)

    # Create in target
    for asset in source_assets:
        create_asset(
            base_url, token, target_folder_id,
            name=asset['Name'],
            value=asset['Value'],
            value_type=asset['ValueType'],
            scope=asset['ValueScope']
        )
```

This quickly populates a new environment with the same asset structure. You'd then update values as needed for that environment.

These patterns make working with assets efficient and maintainable.

### Presenter Notes
- Emphasize caching to reduce API calls
- Show bulk operations for efficiency
- Mention environment migration as practical use case

---

## Section 9: Troubleshooting (2 minutes)

### Visual
- Common error messages
- Troubleshooting flowchart
- Quick fixes list

### Script

Let's quickly cover common issues and how to fix them.

Issue one: "Asset not found." This usually means either the asset doesn't exist in that folder, or your folder context header is wrong. Check that X-UIPATH-OrganizationUnitId matches the folder where the asset actually lives. Asset names are case-sensitive, so also verify spelling and capitalization.

Issue two: "Cannot update asset—Forbidden." This is a permissions issue. Your API credentials don't have Edit permission on that folder. Check the External Application's scopes and the folder permissions for the user account.

Issue three: "Credential password is encrypted and I can't use it." As we've discussed, this is expected behavior, not an error. You can't decrypt credentials via API. If you need to programmatically use a credential, either have a robot do it, or manage the credential outside Orchestrator. Don't expect to extract passwords.

Issue four: "Asset value is too long." Text asset values have limits—5,000 characters for Global scope, 450 for Per Robot scope. If you hit this limit, either shorten your value, split it into multiple assets, or consider storing it elsewhere (like a file or database) and storing just a reference in the asset.

Issue five: API rate limiting. If you're fetching assets very frequently—say in a tight loop—you might hit rate limits. Solution: cache asset values. Fetch once, reuse many times. Don't query Orchestrator for every iteration of a loop.

Those are the most common issues. Most are easily fixed by checking folder context, permissions, and understanding the security design.

### Presenter Notes
- Keep this brief—just awareness of common issues
- Emphasize that many issues are configuration/permissions
- Reiterate that encrypted credentials are not an issue to fix

---

## Section 10: Summary and Next Steps (2 minutes)

### Visual
- Summary slide with key points
- Asset management workflow diagram
- Next module preview

### Script

Let's summarize what we've covered about assets.

Assets are Orchestrator's configuration and secrets management system. They store values separately from workflows, enabling easy updates and environment-specific configuration.

There are two main asset types: Text assets for plain text configuration values like URLs and paths, and Credential assets for encrypted username/password pairs. Use Credential assets for all secrets—passwords, API keys, service account credentials.

Assets have scope: Global means one value for all robots, Per Robot means different values per robot. Most assets are Global scope.

To retrieve assets via API, use the GetAssetByName function for Global assets or GetRobotAssetByName for Per Robot assets. Update assets using PATCH or PUT to /odata/Assets with the asset's Id. Create new assets with POST to /odata/Assets.

Credential passwords are encrypted and cannot be decrypted via API. This is intentional security design. Only robots can decrypt credentials during workflow execution.

Best practice for environments: use the same asset names across all environments, with different values per folder. This lets you deploy the same workflow everywhere without changes.

Cache asset values in your scripts to reduce API calls. Don't fetch on every use—fetch once and reuse.

You now have the knowledge to manage configuration and secrets effectively using Orchestrator assets. This is essential for building maintainable, secure automation.

In the next module, we'll cover Queues and Transactions—a powerful pattern for distributing work across multiple robots and ensuring reliable processing of large data sets. Queues enable scalable, fault-tolerant automation.

Thanks for watching, and I'll see you in the next module!

### Presenter Notes
- Quick recap of main points
- Emphasize practical takeaways
- Build anticipation for queues—a powerful feature

---

## Additional Teaching Notes

### Common Student Questions

**Q: Can I retrieve plain text passwords from Credential assets?**
A: No, this is by design for security. Only robots can decrypt credentials during execution. API clients see encrypted passwords.

**Q: Should I use assets or input arguments for configuration?**
A: Use assets for configuration that's environment-specific or changes occasionally. Use input arguments for data that varies per job execution (like invoice ID, order number).

**Q: How do I use a credential in my Python script?**
A: You generally don't—robots use credentials. If you need to use a credential in Python, manage it separately (environment variables, secrets manager, etc.). Don't expect to extract Orchestrator credentials for use outside robots.

**Q: Can I have the same asset name in different folders?**
A: Yes! This is actually the recommended pattern—same names, different values per environment/folder.

**Q: What's the maximum length for asset values?**
A: Text assets: 5,000 characters (Global), 450 characters (Per Robot). For larger data, use external storage.

### Demo Prerequisites
- Access to UiPath Orchestrator
- Folder with some existing assets (Text and Credential)
- Python environment with requests library
- Authentication credentials

### Additional Examples
- Configuration management for multi-environment setups
- Automated credential rotation scripts
- Asset auditing and reporting
- Migration scripts for environment setup

### Assessment Tips
- Quiz emphasizes both Text and Credential assets
- Understanding of Global vs Per Robot scope is critical
- Security questions about why credentials are encrypted
- Practical questions about when to use each asset type
- Environment strategy questions test real-world application
