# Video Script: Asset Updates
## Module 2.2.2.2 - SDK Fundamentals

**Duration:** 17 minutes
**Target Audience:** Developers implementing dynamic configuration management
**Prerequisites:** Module 2.2.2.1 (asset retrieval)
**Learning Outcomes:** Update assets programmatically via API

---

## Scene 1: Introduction (1 minute)

### Visual: Title slide with update/refresh icon

**[On camera - Instructor]**

Welcome to Module 2.2.2.2: Asset Updates. In the previous module, we learned to read assets. Now let's learn to modify them programmatically.

### Visual: Use case examples

**[Voice over visual]**

Why update assets via code? Here are real-world scenarios:
- Automated credential rotation for security
- Updating configuration based on runtime conditions
- Storing process state between runs
- Synchronizing configuration across environments

### Visual: Learning objectives slide

**[On camera - Instructor]**

In this video, you'll learn:
- The HTTP PUT method for asset updates
- Updating different asset types
- Validation and error handling
- Bulk update patterns
- Credential rotation patterns
- Handling concurrency

Let's make your configuration dynamic!

---

## Scene 2: Update API Basics (2 minutes)

### Visual: API endpoint diagram

**[On camera - Instructor]**

Asset updates use the HTTP PUT method. This is different from POST - PUT updates an existing resource.

### Visual: Endpoint structure

**[Voice over visual]**

The endpoint is:
```
PUT /odata/Assets({asset_id})
```

Notice you need the asset ID - a numeric identifier for the specific asset you're updating.

### Screen recording - Basic update function

**[Voice over screen recording]**

```python
import requests

def update_asset(orchestrator_url, token, folder_id, asset_id, asset_data):
    """Update an asset"""

    response = requests.put(
        f"{orchestrator_url}/odata/Assets({asset_id})",
        headers={
            "Authorization": f"Bearer {token}",
            "X-UIPATH-OrganizationUnitId": str(folder_id),
            "Content-Type": "application/json"
        },
        json=asset_data
    )

    response.raise_for_status()
    return response.json()
```

### Visual: Highlight headers

**[On camera - Instructor]**

Three critical headers:
1. Authorization with Bearer token
2. X-UIPATH-OrganizationUnitId for folder context
3. Content-Type as application/json

Missing any of these will cause the update to fail.

---

## Scene 3: The GET-Modify-PUT Pattern (2.5 minutes)

### Visual: Three-step workflow diagram

**[On camera - Instructor]**

The safest way to update assets follows a three-step pattern. Let me show you why this matters.

### Visual: Wrong approach (crossed out)

**[Voice over]**

**Wrong approach:**
```python
# DON'T DO THIS!
asset_data = {
    'Name': 'MyAsset',
    'Value': 'new value'
}
update_asset(url, token, folder_id, 12345, asset_data)
```

This fails because asset objects have many required fields you're not providing.

### Screen recording - Correct pattern

**[On camera - Instructor]**

The right way:

**[Voice over screen recording]**

```python
def update_text_asset(orchestrator_url, token, folder_id,
                      asset_name, new_value):
    """Update a text asset value"""

    # Step 1: GET current asset
    asset = get_asset(orchestrator_url, token, folder_id, asset_name)

    print(f"Current value: {asset['Value']}")

    # Step 2: MODIFY the value
    asset['Value'] = new_value

    # Step 3: PUT it back
    updated = update_asset(
        orchestrator_url, token, folder_id,
        asset['Id'],  # Use ID from GET
        asset         # Full object with all required fields
    )

    print(f"✓ Updated to: {new_value}")
    return updated
```

### Visual: Highlight the pattern

**[On camera - Instructor]**

GET retrieves the full object with all fields. We modify only what we need, then PUT the complete object back. This ensures all required fields are present.

### Visual: Running the code

```python
# Usage
update_text_asset(
    orchestrator_url, token, folder_id,
    "DatabaseURL",
    "https://new-db.company.com"
)
```

Perfect - DatabaseURL is now updated!

---

## Scene 4: Updating Different Asset Types (2.5 minutes)

### Visual: Four asset types icons

**[On camera - Instructor]**

Each asset type has slightly different update patterns. Let me show you each one.

### Screen recording - Bool asset

**[Voice over screen recording]**

**Boolean Assets:**
```python
def update_bool_asset(orchestrator_url, token, folder_id,
                      asset_name, new_value):
    """Update a boolean asset"""

    asset = get_asset(orchestrator_url, token, folder_id, asset_name)

    if asset['ValueType'] != 'Bool':
        raise TypeError(f"Asset {asset_name} is not a Bool asset")

    # Update with boolean
    asset['Value'] = bool(new_value)

    return update_asset(
        orchestrator_url, token, folder_id,
        asset['Id'], asset
    )
```

Notice the type check - we verify it's actually a Bool asset before updating.

### Visual: Integer asset

```python
def update_integer_asset(orchestrator_url, token, folder_id,
                         asset_name, new_value):
    """Update an integer asset"""

    asset = get_asset(orchestrator_url, token, folder_id, asset_name)

    if asset['ValueType'] != 'Integer':
        raise TypeError(f"Asset {asset_name} is not an Integer asset")

    asset['Value'] = int(new_value)  # Ensure it's an int

    return update_asset(
        orchestrator_url, token, folder_id,
        asset['Id'], asset
    )
```

### Visual: Credential asset

**[On camera - Instructor]**

Credentials are special - they have username and password fields:

```python
def update_credential_asset(orchestrator_url, token, folder_id,
                            asset_name, username=None, password=None):
    """Update credential asset"""

    asset = get_asset(orchestrator_url, token, folder_id, asset_name)

    # Update username if provided
    if username is not None:
        asset['CredentialUsername'] = username

    # Update password if provided
    if password is not None:
        asset['CredentialPassword'] = password

    return update_asset(
        orchestrator_url, token, folder_id,
        asset['Id'], asset
    )
```

You can update username, password, or both.

---

## Scene 5: Validation (2 minutes)

### Visual: Validation checkpoint icon

**[On camera - Instructor]**

Before updating, validate both the type and business rules. This prevents errors and maintains data integrity.

### Screen recording - Type validation

**[Voice over screen recording]**

```python
def validate_asset_update(asset, new_value):
    """Validate update before sending"""

    asset_type = asset['ValueType']

    if asset_type == 'Text':
        if not isinstance(new_value, str):
            raise TypeError(f"Expected str, got {type(new_value)}")

    elif asset_type == 'Bool':
        if not isinstance(new_value, bool):
            raise TypeError(f"Expected bool, got {type(new_value)}")

    elif asset_type == 'Integer':
        if not isinstance(new_value, int):
            raise TypeError(f"Expected int, got {type(new_value)}")

    return True
```

### Visual: Business rule validation

**[On camera - Instructor]**

Add business rules specific to your application:

```python
def validate_business_rules(asset_name, new_value):
    """Validate business constraints"""

    if asset_name == "MaxRetries":
        if new_value < 0 or new_value > 10:
            raise ValueError("MaxRetries must be between 0 and 10")

    elif asset_name == "TimeoutSeconds":
        if new_value < 1 or new_value > 600:
            raise ValueError("TimeoutSeconds must be 1-600 seconds")

    return True
```

### Visual: Safe update with validation

```python
def safe_update_asset(orchestrator_url, token, folder_id,
                      asset_name, new_value):
    """Update with full validation"""

    asset = get_asset(orchestrator_url, token, folder_id, asset_name)

    # Validate type
    validate_asset_update(asset, new_value)

    # Validate business rules
    validate_business_rules(asset_name, new_value)

    # Perform update
    old_value = asset['Value']
    asset['Value'] = new_value

    updated = update_asset(
        orchestrator_url, token, folder_id,
        asset['Id'], asset
    )

    print(f"✓ Updated {asset_name}: {old_value} → {new_value}")
    return updated
```

---

## Scene 6: Bulk Updates (2 minutes)

### Visual: Multiple assets being updated

**[On camera - Instructor]**

Often you need to update multiple assets at once. Let's build a bulk update function.

### Screen recording - Bulk update

**[Voice over screen recording]**

```python
def bulk_update_assets(orchestrator_url, token, folder_id, updates):
    """
    Update multiple assets

    Args:
        updates: Dict of {asset_name: new_value}
    """

    results = {
        'successful': [],
        'failed': []
    }

    for asset_name, new_value in updates.items():
        try:
            safe_update_asset(
                orchestrator_url, token, folder_id,
                asset_name, new_value
            )
            results['successful'].append(asset_name)
            print(f"✓ {asset_name}")
        except Exception as e:
            results['failed'].append({
                'asset': asset_name,
                'error': str(e)
            })
            print(f"✗ {asset_name}: {e}")

    return results
```

### Visual: Usage example

```python
# Define updates
updates = {
    "DatabaseURL": "https://new-db.company.com",
    "MaxRetries": 5,
    "EnableNotifications": True,
    "TimeoutSeconds": 60
}

# Perform bulk update
results = bulk_update_assets(url, token, folder_id, updates)

print(f"\nSuccessful: {len(results['successful'])}")
print(f"Failed: {len(results['failed'])}")
```

### Visual: Output

```
✓ DatabaseURL
✓ MaxRetries
✓ EnableNotifications
✓ TimeoutSeconds

Successful: 4
Failed: 0
```

All four assets updated successfully!

---

## Scene 7: Credential Rotation (2.5 minutes)

### Visual: Password rotation cycle diagram

**[On camera - Instructor]**

A common use case for asset updates is automated credential rotation. Let me show you a production-ready pattern.

### Screen recording - Password generation

**[Voice over screen recording]**

First, generate a secure password:

```python
import secrets
import string

def generate_secure_password(length=16):
    """Generate cryptographically secure password"""
    alphabet = string.ascii_letters + string.digits + string.punctuation
    return ''.join(secrets.choice(alphabet) for _ in range(length))
```

### Visual: Rotation workflow

**[On camera - Instructor]**

The rotation workflow has a critical order: update the external system FIRST, then update Orchestrator.

**[Voice over]**

```python
def rotate_credential(orchestrator_url, token, folder_id,
                      credential_name, external_system_update_fn):
    """Rotate a credential asset"""

    print(f"Rotating password for {credential_name}...")

    # Generate new password
    new_password = generate_secure_password()

    # Step 1: Update external system FIRST
    try:
        external_system_update_fn(new_password)
        print("✓ Updated password in external system")
    except Exception as e:
        print(f"✗ Failed to update external system: {e}")
        raise  # Stop here if external system fails

    # Step 2: Update Orchestrator asset
    try:
        update_credential_asset(
            orchestrator_url, token, folder_id,
            credential_name,
            password=new_password
        )
        print(f"✓ Updated credential asset in Orchestrator")
    except Exception as e:
        print(f"✗ Failed to update Orchestrator: {e}")
        # Consider rolling back external system change
        raise

    print(f"✓ Credential rotation completed")
```

### Visual: Usage example

```python
def update_db_password(new_password):
    # Update password in database system
    db.execute("ALTER USER api_user PASSWORD = ?", new_password)

# Rotate database credential
rotate_credential(
    url, token, folder_id,
    "DatabaseCredential",
    update_db_password
)
```

### Visual: Success output

This ensures robots immediately have the new password when the external system starts requiring it.

---

## Scene 8: Concurrency Considerations (1.5 minutes)

### Visual: Race condition diagram

**[On camera - Instructor]**

Important warning: Orchestrator has no built-in locking for asset updates. Last write wins.

### Visual: Timeline showing conflict

**[Voice over visual]**

Here's what can happen:

```
Time    Process A                Process B
T1      GET asset (value=10)
T2                                GET asset (value=10)
T3      value = 10 + 5 = 15
T4                                value = 10 + 3 = 13
T5      PUT asset (15)
T6                                PUT asset (13)

Final value: 13
Process A's update is lost!
```

### Visual: Solutions

**[On camera - Instructor]**

Mitigation strategies:

1. **Minimize time between GET and PUT**
2. **Use application-level locking** if multiple processes update
3. **Design assets to be set, not incremented** where possible

```python
# ✓ Good: Set to specific value
update_asset(url, token, folder_id, "Status", "COMPLETED")

# ⚠ Careful: Read-modify-write creates race conditions
current = get_asset_value(url, token, folder_id, "Counter")
update_asset(url, token, folder_id, "Counter", current + 1)
```

For the second pattern, use locking if multiple processes might update.

---

## Scene 9: Complete Example (1.5 minutes)

### Screen recording - Full workflow

**[On camera - Instructor]**

Let me show you a complete example: updating configuration for a specific environment.

**[Voice over screen recording]**

```python
from config import OrchestratorConfig
from auth import TokenManager

def update_environment_config(env_name):
    """Update configuration for environment"""

    config = OrchestratorConfig(env=env_name)
    token_mgr = TokenManager(config.client_id, config.client_secret)
    token = token_mgr.get_token()

    print(f"Updating {env_name} environment...")

    # Define updates
    updates = {
        "DatabaseURL": f"https://{env_name}-db.company.com",
        "TimeoutSeconds": 60 if env_name == "prod" else 30,
        "EnableNotifications": env_name == "prod",
        "MaxRetries": 5
    }

    # Perform updates
    results = bulk_update_assets(
        config.orchestrator_url, token, config.folder_id,
        updates
    )

    print(f"\n✓ Configuration updated")
    print(f"  Successful: {len(results['successful'])}")
    print(f"  Failed: {len(results['failed'])}")

# Update staging environment
update_environment_config("staging")
```

### Visual: Output

```
Updating staging environment...
✓ DatabaseURL
✓ TimeoutSeconds
✓ EnableNotifications
✓ MaxRetries

✓ Configuration updated
  Successful: 4
  Failed: 0
```

Perfect - staging environment is now configured!

---

## Scene 10: Conclusion (30 seconds)

### Visual: Key takeaways

**[On camera - Instructor]**

Excellent work! You now know how to update assets programmatically. Remember:

- Use PUT method with asset ID
- Follow GET-Modify-PUT pattern
- Validate both type and business rules
- Handle errors gracefully
- Be aware of concurrency issues
- Update external systems before Orchestrator in rotation

### Visual: Next module preview

Next up: Asset CRUD Operations - creating and deleting assets programmatically for complete lifecycle management.

### Visual: End card

Thanks for watching! Your configuration is now fully dynamic and manageable. See you next time!

---

## Production Notes

**Graphics needed:**
- PUT method icon and workflow
- GET-Modify-PUT three-step diagram
- Four asset types with update patterns
- Validation checkpoint flowchart
- Credential rotation workflow (external system → Orchestrator)
- Race condition timeline
- Bulk update visualization

**Screen recordings:**
- Writing update_asset function
- Implementing GET-Modify-PUT pattern
- Type-specific update functions
- Building validation functions
- Creating bulk_update_assets
- Complete credential rotation example
- Environment configuration update workflow

**Code examples:**
- update_asset function
- update_text_asset / update_bool_asset / update_integer_asset
- update_credential_asset
- validate_asset_update and validate_business_rules
- bulk_update_assets
- rotate_credential pattern
- Complete environment update script

**Callouts/Annotations:**
- Highlight asset ID in endpoint path
- Mark required headers
- Annotate full object preservation in PUT
- Show type validation checks
- Emphasize order in credential rotation
- Point out race condition window

**Pacing notes:**
- Clear explanation of PUT vs POST
- Emphasize importance of GET-Modify-PUT pattern
- Methodical walkthrough of each asset type
- Practical validation examples
- Real-world credential rotation pattern
- Warning about concurrency issues

**Accessibility:**
- Accurate captions for all technical terms
- Describe workflow diagrams verbally
- High-contrast code display
- Verbal description of race condition timeline
