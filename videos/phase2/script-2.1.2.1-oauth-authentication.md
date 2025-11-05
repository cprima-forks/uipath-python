# Video Script: OAuth Authentication Flow
## Module 2.1.2.1 - SDK Fundamentals

**Duration:** 25 minutes
**Target Audience:** Developers implementing Orchestrator authentication
**Prerequisites:** Modules 2.1.1.1-2.1.1.3, Orchestrator access
**Learning Outcomes:** Implement OAuth authentication for UiPath Orchestrator

---

## Scene 1: Introduction (2 minutes)

### Visual: Title slide with security lock icon

**[On camera - Instructor]**

Welcome to Module 2.1.2.1: OAuth Authentication Flow. This is where we move from installing the SDK to actually using it to connect with UiPath Orchestrator.

### Visual: Split screen - insecure vs secure authentication

**[Voice over visual]**

Authentication might seem like a tedious topic, but it's absolutely critical. Poor authentication security has led to some of the biggest data breaches in history. We're going to do this right from the start.

### Visual: Learning objectives slide

**[On camera - Instructor]**

In this video, you'll learn:
- Why OAuth 2.0 is the modern authentication standard
- How to create External Applications in Orchestrator
- The complete OAuth token flow in Python
- Token management and expiration handling
- Security best practices you must follow

Let's dive into the world of secure authentication!

---

## Scene 2: Why OAuth 2.0? (3 minutes)

### Visual: Code showing username/password authentication (crossed out)

**[On camera - Instructor]**

Let me start by showing you what NOT to do.

### Visual: Bad example code

**[Voice over code]**

```python
# DON'T DO THIS!
username = "admin"
password = "MySecretPassword123"

response = orchestrator_login(username, password)
```

This approach has serious problems.

### Visual: List of problems appearing

**[Voice over]**

**Problem 1:** Your password is exposed in source code. Anyone with code access has full admin rights.

**Problem 2:** No permission limiting. This password might have full admin access when your script only needs to read jobs.

**Problem 3:** Hard to revoke. If compromised, you need to change the password everywhere it's used.

**Problem 4:** No expiration. A stolen password works forever until manually changed.

### Visual: OAuth token flow diagram

**[On camera - Instructor]**

OAuth 2.0 solves all these problems. Instead of using your password directly, you create credentials specifically for your application - called an External Application. Your app uses these credentials to get a temporary access token with limited permissions.

### Visual: Comparison table

**[Voice over table]**

With OAuth:
- Your password never appears in code
- Tokens have specific, limited permissions
- Tokens expire automatically (typically after 1 hour)
- Easy to revoke without changing your password
- Each application has its own credentials

This is why every modern API uses OAuth 2.0. Let's implement it.

---

## Scene 3: Creating an External Application (4 minutes)

### Visual: Orchestrator UI - Admin section

**[On camera - Instructor]**

Before you can authenticate, you need to create an External Application in Orchestrator. Let me show you the process.

### Screen recording - Orchestrator navigation

**[Voice over screen recording]**

I'm logged into UiPath Cloud Orchestrator. Navigate to the Admin section - you'll see it in the top-right corner.

Click on "External Applications" in the left sidebar.

### Visual: External Applications list

You'll see any existing applications here. Let's create a new one. Click "Add Application."

### Visual: Create application form

**[Voice over form]**

Fill in the details:

**Name:** Give it a descriptive name like "Python SDK Development App." You'll thank yourself later when you have multiple applications.

**Application Type:** For learning, select "Non-production." For real deployments, use "Production."

**Scopes:** This is crucial - you're selecting which permissions this application will have.

### Visual: Scopes section highlighted

For our examples, we'll need:
- OR.Jobs - to read and manage jobs
- OR.Queues - to work with queue items
- OR.Assets - to read assets

Remember the principle of least privilege - only select what you actually need.

### Visual: Save and credentials display

**[Voice over]**

Click "Add" to save.

Now pay very close attention to this screen. You'll see two pieces of information:

**App ID (Client ID)** - This is like a username for your application. It's not secret.

**App Secret (Client Secret)** - This is like a password. It's shown ONLY ONCE and cannot be retrieved later.

### Visual: Warning sign flashing

**[On camera - Instructor]**

Stop! Before clicking anything else, copy that Client Secret. Save it somewhere secure. If you lose it, you'll need to regenerate it.

I'm going to store mine in a .env file, which we'll set up in just a moment.

---

## Scene 4: OAuth Token Endpoint (2 minutes)

### Visual: Token endpoint diagram

**[On camera - Instructor]**

To get an access token, your application makes a request to the OAuth token endpoint. The endpoint URL depends on whether you're using Cloud or on-premises Orchestrator.

### Visual: Cloud endpoint

**[Voice over]**

For UiPath Cloud:
```
https://cloud.uipath.com/identity_/connect/token
```

Notice two things: First, there's an underscore after "identity" - that's not a typo. Second, the path is "/connect/token."

### Visual: Other regions

If you're using a different region like Alpha:
```
https://alpha.uipath.com/identity_/connect/token
```

### Visual: On-premises endpoint

**[On camera - Instructor]**

For on-premises Orchestrator, the URL is:
```
https://your-orchestrator-domain/identity/connect/token
```

Key difference: NO underscore after "identity" for on-premises.

---

## Scene 5: Implementing Token Request (5 minutes)

### Visual: Python code editor

**[On camera - Instructor]**

Now let's write Python code to get an access token. I'll walk through each step.

### Screen recording - creating .env file

**[Voice over screen recording]**

First, create a .env file to store your credentials securely:

```bash
# .env
UIPATH_CLIENT_ID=your_client_id_here
UIPATH_CLIENT_SECRET=your_client_secret_here
```

Replace with your actual values from the External Application.

**Critical:** Add .env to your .gitignore file so it's never committed:

```bash
echo ".env" >> .gitignore
```

### Visual: Python script creation

Now create our authentication script:

```python
# oauth_auth.py
import os
import requests
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Configuration
TOKEN_URL = "https://cloud.uipath.com/identity_/connect/token"
CLIENT_ID = os.getenv("UIPATH_CLIENT_ID")
CLIENT_SECRET = os.getenv("UIPATH_CLIENT_SECRET")

print(f"Using Client ID: {CLIENT_ID}")
```

### Visual: Highlight environment variable loading

Notice we're using python-dotenv to load credentials from the .env file. The credentials never appear in the source code.

### Visual: Token request code

**[Voice over]**

Now the token request:

```python
# Request access token
response = requests.post(
    TOKEN_URL,
    data={
        "grant_type": "client_credentials",
        "client_id": CLIENT_ID,
        "client_secret": CLIENT_SECRET,
        "scope": "OR.Jobs OR.Queues"
    }
)

# Check for errors
response.raise_for_status()

# Parse response
token_data = response.json()
access_token = token_data["access_token"]
expires_in = token_data["expires_in"]

print(f"Token obtained!")
print(f"Expires in: {expires_in} seconds")
```

### Visual: Run the script

Let's run it:

```bash
python oauth_auth.py
```

### Visual: Output showing success

**[Voice over output]**

Perfect! We got our token. The `expires_in` value shows 3600 seconds - that's 1 hour.

---

## Scene 6: Using the Access Token (3 minutes)

### Visual: API request diagram

**[On camera - Instructor]**

Now that we have a token, let's use it to make API calls to Orchestrator.

### Screen recording - extending the script

**[Voice over screen recording]**

Add this to your script:

```python
# Use token to call Orchestrator API
ORCHESTRATOR_URL = "https://cloud.uipath.com/yourorg/yourtenant/orchestrator_"

headers = {
    "Authorization": f"Bearer {access_token}",
    "Content-Type": "application/json"
}

# Get list of jobs
jobs_response = requests.get(
    f"{ORCHESTRATOR_URL}/odata/Jobs",
    headers=headers
)

jobs = jobs_response.json()
print(f"Found {len(jobs['value'])} jobs")
```

### Visual: Highlight Authorization header

**[Voice over]**

The key is the Authorization header with the format "Bearer" followed by the token. This is the OAuth 2.0 standard.

### Visual: Running updated script

Run it again:

```bash
python oauth_auth.py
```

### Visual: Output showing jobs

**[On camera - Instructor]**

Excellent! We're successfully authenticated and retrieving data from Orchestrator.

---

## Scene 7: Token Management (4 minutes)

### Visual: Token lifecycle diagram

**[On camera - Instructor]**

Access tokens expire. After an hour, that token we got stops working. Let's implement proper token management with caching and automatic refresh.

### Screen recording - TokenManager class

**[Voice over screen recording]**

Create a TokenManager class:

```python
import time

class TokenManager:
    def __init__(self, client_id, client_secret, token_url):
        self.client_id = client_id
        self.client_secret = client_secret
        self.token_url = token_url
        self.access_token = None
        self.expires_at = 0

    def get_token(self):
        # If token expired or doesn't exist, get new one
        if time.time() >= self.expires_at:
            self._refresh_token()
        return self.access_token
```

### Visual: Highlight expiration check

Notice we check if the current time is past our expiration time. If so, we need a new token.

### Visual: Refresh method

```python
    def _refresh_token(self):
        response = requests.post(
            self.token_url,
            data={
                "grant_type": "client_credentials",
                "client_id": self.client_id,
                "client_secret": self.client_secret,
                "scope": "OR.Jobs OR.Queues"
            }
        )
        response.raise_for_status()

        data = response.json()
        self.access_token = data["access_token"]
        # Expire 60 seconds early for safety
        self.expires_at = time.time() + data["expires_in"] - 60

        print("Token refreshed")
```

### Visual: Usage example

**[Voice over]**

Now use it:

```python
# Create manager
token_mgr = TokenManager(CLIENT_ID, CLIENT_SECRET, TOKEN_URL)

# Get token (will fetch first time)
token = token_mgr.get_token()

# Later in code - get token again
# (will reuse cached token if still valid)
token = token_mgr.get_token()
```

### Visual: Demonstration of caching

**[On camera - Instructor]**

The first call fetches a new token. Subsequent calls within the hour return the cached token. After expiration, it automatically fetches a new one. This is production-ready token management.

---

## Scene 8: Security Best Practices (2 minutes)

### Visual: Security checklist

**[On camera - Instructor]**

Let's cover essential security practices you must follow.

### Visual: Best practices list

**[Voice over list]**

**1. Never hardcode secrets** - Always use environment variables or secret management systems.

**2. Add .env to .gitignore** - Never commit secrets to version control.

**3. Use least privilege scopes** - Request only the permissions you need.

**4. Don't log tokens** - Keep tokens out of log files and error messages.

**5. Rotate credentials regularly** - Regenerate Client Secrets periodically.

**6. Use production-grade secret managers** - In production, use Azure Key Vault, AWS Secrets Manager, or similar.

**7. Monitor for failures** - Alert on repeated authentication failures.

### Visual: Security violation examples crossed out

**[On camera - Instructor]**

Following these practices protects both your application and your Orchestrator environment from compromise.

---

## Scene 9: Troubleshooting (2 minutes)

### Visual: Common errors

**[On camera - Instructor]**

Let me show you the most common issues and their solutions.

### Visual: Error examples with solutions

**[Voice over]**

**401 Unauthorized when getting token:**
- Check Client ID and Secret are correct
- Verify no extra spaces in credentials
- Confirm External Application is enabled
- Check token endpoint URL

**403 Forbidden on API calls:**
- Token is valid but lacks required scope
- Add needed scopes to External Application
- Request those scopes when getting token

**Token works then stops:**
- Token expired (after 1 hour)
- Implement automatic token refresh
- Use the TokenManager pattern we created

---

## Scene 10: Hands-On Exercise Preview (1 minute)

### Visual: Exercise instructions

**[On camera - Instructor]**

Time for hands-on practice! Pause the video and complete this exercise:

1. Create an External Application in your Orchestrator
2. Set up a .env file with your credentials
3. Implement the complete OAuth flow
4. Make an API call to retrieve jobs
5. Implement the TokenManager class
6. Test token caching and refresh

This should take about 30 minutes. The solution is in the course materials.

---

## Scene 11: Conclusion (1 minute)

### Visual: Key takeaways

**[On camera - Instructor]**

Excellent work! You've mastered OAuth authentication with UiPath Orchestrator. Let's recap:

**[Voice over]**

- OAuth 2.0 provides secure, scoped authentication without exposing passwords
- External Applications give you Client ID and Secret
- Token endpoint exchanges credentials for time-limited access tokens
- Tokens expire and need refresh - implement caching
- Security best practices are non-negotiable

### Visual: Next module preview

In our next module, Manual Environment Configuration, we'll organize all these settings for multiple environments - development, staging, and production.

### Visual: End card

Thank you for watching. You're now ready to authenticate with any UiPath Orchestrator using modern, secure OAuth 2.0. See you in the next module!

---

## Production Notes

**Graphics needed:**
- OAuth 2.0 flow diagram (client to auth server to resource server)
- External Application creation process
- Token lifecycle (request → use → expire → refresh)
- Authorization header format
- Security best practices checklist
- Common errors troubleshooting flowchart

**Screen recordings:**
- Complete External Application creation in Orchestrator
- Creating and configuring .env file
- Writing and running oauth_auth.py script
- Implementing TokenManager class
- Successful API call with token
- Demonstrating token caching

**Code examples to display:**
- Bad example (hardcoded passwords)
- Good example (OAuth with .env)
- Token request
- Using token in API call
- Complete TokenManager class

**Callouts/Annotations:**
- Highlight Client Secret warning (shown only once)
- Point out underscore in cloud.uipath.com/identity_
- Annotate Authorization header format
- Mark expiration time in token response

**Pacing notes:**
- Slow down during External Application creation - critical step
- Emphasize the Client Secret warning
- Allow time for viewers to understand token lifecycle
- Clear, methodical code walkthroughs
- Pause after each security best practice

**Accessibility:**
- Accurate captions for all technical terms
- Describe Orchestrator UI navigation verbally
- High-contrast code display
- Verbal description of all diagrams
