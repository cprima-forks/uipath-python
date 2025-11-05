# Video Script: Asset Retrieval
## Module 2.2.2.1 - SDK Fundamentals

**Duration:** 18 minutes
**Target Audience:** Developers implementing configuration management
**Prerequisites:** Module 2.2.1.4 (batch execution)
**Learning Outcomes:** Retrieve and manage Orchestrator assets effectively

---

## Scene 1: Introduction (1.5 minutes)

### Visual: Title slide with configuration icon

**[On camera - Instructor]**

Welcome to Module 2.2.2.1: Asset Retrieval. We're moving from job management to configuration management with Orchestrator Assets.

### Visual: Hardcoded config vs assets comparison

**[Voice over visual]**

Imagine this scenario: You have a robot process that connects to a database. During development, it uses your dev database. But in production, it needs to connect to the production database. How do you handle this without changing code?

### Visual: Learning objectives slide

**[On camera - Instructor]**

That's where Assets come in. In this video, you'll learn:
- What Orchestrator Assets are and why they matter
- How to retrieve different asset types via API
- Understanding asset scoping (Global, Folder, Robot)
- Secure credential handling
- Efficient asset caching patterns

Let's make your processes configurable!

---

## Scene 2: What Are Assets? (2.5 minutes)

### Visual: Assets icon in Orchestrator

**[On camera - Instructor]**

Assets are centralized configuration values stored in Orchestrator. Think of them as environment variables, but stored centrally and accessible to all your robots.

### Visual: Code showing hardcoded values (crossed out)

**[Voice over visual]**

The old way - hardcoding configuration:

```python
# DON'T DO THIS!
database_url = "https://prod-db.company.com"
api_key = "abc123secret"
max_retries = 3
```

Problems with this approach:
- Different environments require code changes
- Secrets exposed in source code
- No centralized management
- Redeployment needed for config changes

### Visual: Assets in Orchestrator UI

**[On camera - Instructor]**

The modern way - using Assets. You define configuration values in Orchestrator once, and all robots can access them.

### Screen recording - Creating an asset

**[Voice over screen recording]**

In Orchestrator, navigate to Assets. Let's create a few:

1. Click "Add Asset"
2. Name: "DatabaseURL"
3. Type: Text
4. Value: "https://prod-db.company.com"
5. Save

Now create another:
- Name: "MaxRetries"
- Type: Integer
- Value: 3

And one more:
- Name: "APICredential"
- Type: Credential
- Username: "api_user"
- Password: "secure_password"

### Visual: Benefits list

**[On camera - Instructor]**

Now your processes retrieve these values at runtime. Change the DatabaseURL in Orchestrator, and all robots use the new value immediately - no code changes, no redeployment!

---

## Scene 3: Asset Types (2 minutes)

### Visual: Four asset type icons

**[On camera - Instructor]**

Orchestrator supports four asset types. Let me show you each one.

### Visual: Asset types with examples

**[Voice over visual]**

**Text Assets:**
- String values
- URLs: "https://app.company.com"
- File paths: "/data/invoices/"
- Any text configuration

**Bool Assets:**
- True/false values
- Feature flags: EnableNotifications = True
- Debug modes: DebugMode = False

**Integer Assets:**
- Whole numbers
- Timeouts: TimeoutSeconds = 30
- Limits: MaxRetries = 3
- Thresholds: MinimumAmount = 100

**Credential Assets:**
- Username + password pairs
- API credentials
- Database logins
- System accounts

### Visual: Credential security callout

**[On camera - Instructor]**

Important note about credentials: When you retrieve a credential asset via API, you get the username but NEVER the password. This is a security feature - passwords are only accessible to robots during process execution.

---

## Scene 4: Asset Scoping (2.5 minutes)

### Visual: Scope hierarchy diagram

**[On camera - Instructor]**

Assets can be scoped at three levels, and this is a powerful feature for managing multiple environments.

### Visual: Scope levels visualization

**[Voice over visual]**

**Global Scope:**
- Available to all robots in all folders
- Your default, organization-wide configuration

**Per Folder Scope:**
- Specific to one folder (environment)
- Overrides global values in that folder

**Per Robot Scope:**
- Specific to individual robots
- Highest precedence, most specific

### Visual: Precedence example

**[On camera - Instructor]**

Here's how precedence works. Let's say you have an asset named "DatabaseURL":

**[Voice over]**

```
Global: DatabaseURL = "https://db.company.com"

Development Folder: DatabaseURL = "https://dev-db.company.com"

Production Folder: (no override, uses global)

Robot-Specific: DatabaseURL = "https://robot5-db.company.com"
```

Resolution order: Per Robot → Per Folder → Global

So a robot in the Development folder with no robot-specific override gets "https://dev-db.company.com".

### Screen recording - Setting scopes in Orchestrator

**[Voice over screen recording]**

When creating an asset, you choose the scope:
- Select "Global" for organization-wide defaults
- Select "Per Folder" and choose the folder
- Select "Per Robot" and choose specific robots

This gives you powerful, flexible configuration management.

---

## Scene 5: Retrieving Assets via API (3.5 minutes)

### Visual: API endpoint

**[On camera - Instructor]**

Now let's write code to retrieve assets. The endpoint is simple: GET /odata/Assets.

### Screen recording - Basic retrieval

**[Voice over screen recording]**

```python
import requests

def get_asset(orchestrator_url, token, folder_id, asset_name):
    """Retrieve a single asset by name"""

    response = requests.get(
        f"{orchestrator_url}/odata/Assets",
        headers={
            "Authorization": f"Bearer {token}",
            "X-UIPATH-OrganizationUnitId": str(folder_id)
        },
        params={
            "$filter": f"Name eq '{asset_name}'"
        }
    )

    response.raise_for_status()
    data = response.json()

    # Check if asset exists
    if not data['value']:
        raise ValueError(f"Asset '{asset_name}' not found")

    return data['value'][0]
```

### Visual: Highlight folder_id header

**[Voice over]**

Notice the X-UIPATH-OrganizationUnitId header. This is critical - it tells Orchestrator which folder context to use for asset resolution. This determines which scoped assets are available.

### Visual: Response structure

**[On camera - Instructor]**

The response includes several fields. Here's what you get back:

```json
{
  "Name": "DatabaseURL",
  "ValueType": "Text",
  "Value": "https://prod-db.company.com",
  "ValueScope": "Global",
  "Id": 12345
}
```

### Screen recording - Extracting values

**[Voice over]**

Extract the value:

```python
asset = get_asset(orchestrator_url, token, folder_id, "DatabaseURL")
url = asset['Value']  # "https://prod-db.company.com"

print(f"Database URL: {url}")
```

This works for all types - Text, Bool, Integer. The 'Value' field contains the actual value.

### Visual: Credential example

```python
# Credential asset
cred = get_asset(orchestrator_url, token, folder_id, "APICredential")

username = cred['CredentialUsername']  # "api_user"
password = cred['CredentialPassword']  # "" (empty - never returned!)

print(f"Username: {username}")
print(f"Password: {password}")  # Will show empty string
```

Remember - passwords are never returned via API!

---

## Scene 6: Error Handling and Defaults (2 minutes)

### Visual: Error scenarios

**[On camera - Instructor]**

What happens if an asset doesn't exist? Let's handle that gracefully.

### Screen recording - Error handling

**[Voice over screen recording]**

```python
def get_asset_with_default(orchestrator_url, token, folder_id,
                            asset_name, default_value):
    """Get asset or return default if not found"""

    try:
        asset = get_asset(orchestrator_url, token, folder_id, asset_name)
        return asset['Value']
    except ValueError:
        # Asset not found, use default
        print(f"Asset '{asset_name}' not found, using default: {default_value}")
        return default_value
```

### Visual: Usage example

```python
# If asset exists, use it; otherwise use default
timeout = get_asset_with_default(
    orchestrator_url, token, folder_id,
    "TimeoutSeconds", default_value=30
)

print(f"Using timeout: {timeout} seconds")
```

### Visual: Type validation

**[On camera - Instructor]**

You can also validate the asset type:

```python
def get_asset_value(orchestrator_url, token, folder_id,
                     asset_name, expected_type=None):
    """Get asset with type validation"""

    asset = get_asset(orchestrator_url, token, folder_id, asset_name)

    if expected_type and asset['ValueType'] != expected_type:
        raise TypeError(
            f"Asset '{asset_name}' is {asset['ValueType']}, "
            f"expected {expected_type}"
        )

    return asset['Value']

# Usage
retries = get_asset_value(url, token, folder_id,
                          "MaxRetries", expected_type="Integer")
```

This prevents bugs from wrong asset types.

---

## Scene 7: Asset Caching (2.5 minutes)

### Visual: Performance comparison - cached vs uncached

**[On camera - Instructor]**

Assets change rarely but are used often. Fetching them on every use is inefficient. Let's implement caching.

### Screen recording - Cache class

**[Voice over screen recording]**

```python
class AssetCache:
    def __init__(self, orchestrator_url, token, folder_id):
        self.orchestrator_url = orchestrator_url
        self.token = token
        self.folder_id = folder_id
        self._cache = {}

    def get(self, asset_name, force_refresh=False):
        """Get asset from cache or fetch if not cached"""

        if force_refresh or asset_name not in self._cache:
            # Fetch from Orchestrator
            asset = get_asset(
                self.orchestrator_url, self.token,
                self.folder_id, asset_name
            )
            self._cache[asset_name] = asset['Value']

        return self._cache[asset_name]

    def clear(self):
        """Clear the cache"""
        self._cache.clear()
```

### Visual: Usage demonstration

```python
# Create cache
cache = AssetCache(orchestrator_url, token, folder_id)

# First call - fetches from Orchestrator
url = cache.get("DatabaseURL")  # API call

# Second call - returns from cache
url = cache.get("DatabaseURL")  # No API call!

# Force refresh when needed
url = cache.get("DatabaseURL", force_refresh=True)  # API call
```

### Visual: Performance metrics

**[On camera - Instructor]**

With caching:
- First access: ~200ms (API call)
- Cached access: <1ms (memory lookup)

That's 200x faster! For applications that access assets frequently, this is a significant improvement.

---

## Scene 8: Configuration Class Pattern (2 minutes)

### Visual: Clean architecture diagram

**[On camera - Instructor]**

Let's create a clean configuration class that uses assets.

### Screen recording - Configuration class

**[Voice over screen recording]**

```python
class AppConfiguration:
    """Application configuration using Orchestrator assets"""

    def __init__(self, orchestrator_url, token, folder_id):
        self.cache = AssetCache(orchestrator_url, token, folder_id)

    @property
    def database_url(self):
        return self.cache.get("DatabaseURL")

    @property
    def max_retries(self):
        return self.cache.get("MaxRetries")

    @property
    def timeout_seconds(self):
        return self.cache.get("TimeoutSeconds")

    @property
    def notifications_enabled(self):
        return self.cache.get("EnableNotifications")

    def refresh(self):
        """Refresh all cached assets"""
        self.cache.clear()
```

### Visual: Using the configuration class

```python
# Initialize once
config = AppConfiguration(orchestrator_url, token, folder_id)

# Access as properties throughout your code
print(f"Connecting to: {config.database_url}")
print(f"Max retries: {config.max_retries}")

# Use in logic
if config.notifications_enabled:
    send_notification("Process started")

# Refresh when config changes
config.refresh()
```

### Visual: Benefits callout

**[On camera - Instructor]**

This pattern gives you:
- Clean, readable code
- Automatic caching
- Type hints possible
- Easy testing (mock the config class)

---

## Scene 9: Complete Example (1 minute)

### Screen recording - Full workflow

**[On camera - Instructor]**

Let me show you a complete example using everything we've learned.

**[Voice over screen recording]**

```python
from config import OrchestratorConfig
from auth import TokenManager
from assets import AssetCache

# Setup
config = OrchestratorConfig()
token_mgr = TokenManager(config.client_id, config.client_secret)
token = token_mgr.get_token()

# Initialize asset cache
asset_cache = AssetCache(config.orchestrator_url, token, config.folder_id)

# Get configuration from assets
input_folder = asset_cache.get("InvoiceInputFolder")
output_folder = asset_cache.get("InvoiceOutputFolder")
max_retries = asset_cache.get("ProcessRetries")
email_enabled = asset_cache.get("SendEmailNotifications")

# Use configuration
print(f"📁 Input folder: {input_folder}")
print(f"📁 Output folder: {output_folder}")
print(f"🔄 Max retries: {max_retries}")
print(f"📧 Email notifications: {email_enabled}")

# Process with configuration
for retry in range(max_retries):
    try:
        process_invoices(input_folder, output_folder)
        if email_enabled:
            send_completion_email()
        print("✓ Processing complete")
        break
    except Exception as e:
        print(f"Attempt {retry + 1} failed: {e}")
```

### Visual: Output showing success

Everything configured through assets - no hardcoded values!

---

## Scene 10: Conclusion (30 seconds)

### Visual: Key takeaways

**[On camera - Instructor]**

Excellent work! You now know how to manage configuration using Orchestrator Assets. Remember:

- Assets centralize configuration in Orchestrator
- Four types: Text, Bool, Integer, Credential
- Three scopes: Global, Per Folder, Per Robot
- Credential passwords never returned via API
- Cache assets for performance
- Use configuration classes for clean code

### Visual: Next module preview

Next up: Asset Updates - modifying asset values programmatically, bulk operations, and change management.

### Visual: End card

Thanks for watching! Your processes are now flexible and configurable. See you next time!

---

## Production Notes

**Graphics needed:**
- Assets icon and concept visualization
- Four asset types with icons
- Scope hierarchy diagram (Global → Folder → Robot)
- Precedence resolution flowchart
- API endpoint structure
- Cache performance comparison chart
- Configuration class architecture

**Screen recordings:**
- Creating assets in Orchestrator UI
- Setting different scope levels
- Writing get_asset function
- Implementing error handling
- Building AssetCache class
- Creating AppConfiguration class
- Complete end-to-end example

**Code examples:**
- get_asset function
- get_asset_with_default function
- AssetCache class
- AppConfiguration class
- Complete usage example

**Callouts/Annotations:**
- Highlight X-UIPATH-OrganizationUnitId header importance
- Mark that credential passwords are never returned
- Annotate scope precedence
- Show cache performance gains
- Point out property decorators

**Pacing notes:**
- Clear explanation of what assets are and why they matter
- Methodical walkthrough of scoping with visual examples
- Emphasize credential security limitation
- Smooth progression from basic retrieval to caching
- Practical examples throughout

**Accessibility:**
- Accurate captions for all technical terms
- Describe Orchestrator UI verbally
- High-contrast code display
- Verbal description of diagrams and hierarchies
