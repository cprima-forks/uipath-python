# Video Script: Authentication and Authorization

**Module:** 1.2.1.2
**Duration:** 35 minutes
**Target Audience:** UiPath Agent Developers (Intermediate)
**Prerequisites:** Module 1.2.1.1 (UiPath Orchestrator Architecture)

---

## Section 1: Introduction (2 minutes)

**[VISUAL: Title slide with module objectives]**

**Presenter:**

"Welcome to Module 1.2.1.2: Authentication and Authorization. In this module, we're going to learn how to securely access the UiPath Orchestrator API using OAuth 2.0.

In the previous module, we learned about Orchestrator's architecture—its components, organizational structure, and deployment models. Now we're going to connect to it programmatically from Python.

To do that, we need to understand two fundamental concepts: authentication and authorization. These are the gatekeepers that control who can access the API and what they can do with it.

We'll cover:
- The difference between authentication and authorization
- How OAuth 2.0 works in UiPath
- Generating and using access tokens
- Managing permissions with scopes and roles
- Security best practices

By the end, you'll be able to implement secure, production-ready authentication in your UiPath Python applications.

Let's start with the fundamental distinction between authentication and authorization."

**[VISUAL: Transition to concepts slide]**

---

## Section 2: Authentication vs Authorization (3 minutes)

**[VISUAL: Side-by-side comparison]**

**Presenter:**

"First, let's clarify two terms that are often confused: authentication and authorization.

**[VISUAL: Highlight Authentication]**

Authentication answers the question: WHO are you? It's about proving your identity. Think of it like showing your driver's license at airport security—you're proving who you are.

In technical terms, authentication is when you provide credentials:
- Username and password
- API key
- Client ID and client secret
- Biometric data

The system verifies these credentials and confirms your identity.

**[VISUAL: Highlight Authorization]**

Authorization answers the question: WHAT can you do? It's about permissions and access rights. Continuing the airport analogy, once they know who you are, authorization determines if you can board a first-class flight or if you're limited to economy.

In technical terms, authorization checks:
- Do you have permission to read this data?
- Can you create a job?
- Are you allowed to delete that process?

**[VISUAL: Show the relationship]**

Here's the key: Authentication happens first, then authorization.

1. You authenticate—prove who you are
2. The system checks what you're authorized to do
3. Your request is allowed or denied based on permissions

**[VISUAL: Real example]**

Let's make this concrete:

You log in to Orchestrator with username and password—that's authentication.

Once logged in, the system checks your role. If you're a Viewer, you can read data but not modify it. If you're an Administrator, you can do anything. That's authorization.

Both are required for a secure system. Authentication without authorization means anyone who logs in has full access. Authorization without authentication means you can't verify who's making the request.

Now let's look at how UiPath implements this using OAuth 2.0."

---

## Section 3: Introduction to OAuth 2.0 (4 minutes)

**[VISUAL: OAuth 2.0 logo and overview]**

**Presenter:**

"UiPath Orchestrator uses OAuth 2.0 for API authentication. OAuth 2.0 is an industry-standard authorization framework used by Google, Microsoft, GitHub, and virtually every major platform.

**[VISUAL: Show problems with older methods]**

But first, why OAuth 2.0? What problems does it solve?

Older authentication methods had issues:

Basic Authentication: You send your username and password with every single API request. If someone intercepts the traffic or if the credentials are logged, they're exposed.

Static API Keys: These are long-lived credentials that never expire. If an API key leaks—and they often do—it's valid forever until manually revoked.

Shared Passwords: Multiple applications using the same credentials. When one application is compromised, all are compromised.

**[VISUAL: Show OAuth 2.0 benefits]**

OAuth 2.0 solves these problems:

First, it uses access tokens that are temporary. Tokens typically expire after one hour. If a token is compromised, it's only useful for that hour.

Second, it implements scoped permissions. When requesting a token, you specify what permissions you need. The token is limited to those permissions. Can't read robots? You can't use that token to read robots.

Third, it provides refresh tokens. Instead of re-authenticating when your access token expires, you use a refresh token to get a new access token. This enables long-running applications without repeatedly entering credentials.

Fourth, it's an industry standard with extensive tooling and security research behind it.

**[VISUAL: Show OAuth 2.0 components]**

OAuth 2.0 has several key components:

- Authorization Server: Issues tokens (UiPath Identity Server)
- Resource Server: The API you're accessing (Orchestrator API)
- Client: Your application requesting access
- Access Token: Temporary credential for API access
- Refresh Token: Long-lived credential to get new access tokens

**[VISUAL: Show different OAuth flows]**

OAuth 2.0 defines several flows for different scenarios:

Authorization Code Flow: For web applications where a user logs in through a browser.

Client Credentials Flow: For machine-to-machine communication with no user involved.

Implicit Flow: For single-page apps (now deprecated for security reasons).

For our use case—Python scripts and automation agents—we'll use the Client Credentials Flow. This is specifically designed for backend applications and services that need to access APIs without user interaction."

---

## Section 4: Client Credentials Flow (4 minutes)

**[VISUAL: Flow diagram]**

**Presenter:**

"Let's dive into the Client Credentials Flow since that's what we'll implement.

**[VISUAL: Step-by-step diagram]**

Here's how it works:

Step 1: Your application sends a request to the token endpoint with its credentials—specifically, a client_id and client_secret. These identify and authenticate your application.

Step 2: The authorization server validates these credentials. If valid, it generates an access token with the requested scopes.

Step 3: The server returns the access token to your application, along with information about when it expires.

Step 4: Your application includes this access token in API requests to Orchestrator.

Step 5: Orchestrator validates the token and processes the request if authorized.

**[VISUAL: Show code example]**

Let me show you what this looks like in code:

```python
import requests

# Step 1: Prepare credentials
token_url = 'https://cloud.uipath.com/identity_/connect/token'
client_id = 'your-client-id'
client_secret = 'your-client-secret'

# Step 2: Request token
response = requests.post(token_url, data={
    'grant_type': 'client_credentials',
    'client_id': client_id,
    'client_secret': client_secret,
    'scope': 'OR.Robots OR.Jobs'
})

# Step 3: Extract access token
token_data = response.json()
access_token = token_data['access_token']
expires_in = token_data['expires_in']

# Step 4: Use token in API calls
headers = {'Authorization': f'Bearer {access_token}'}
robots = requests.get(
    'https://cloud.uipath.com/org/tenant/odata/Robots',
    headers=headers
)
```

**[VISUAL: Highlight the key points]**

Key points:

The grant_type is 'client_credentials'—this tells the server which OAuth flow you're using.

The client_id and client_secret authenticate your application. These are like a username and password for your app.

The scope parameter specifies what permissions you're requesting. More on scopes in a moment.

The response includes an access_token and expires_in (usually 3600 seconds = 1 hour).

You include the token in the Authorization header using the Bearer scheme. This is the OAuth 2.0 standard.

**[VISUAL: Emphasize security]**

This flow is secure because:
- Credentials are only sent to the token endpoint, not with every API call
- Access tokens are short-lived
- Tokens can be scoped to specific permissions
- Communication happens over HTTPS (encrypted)

Now let's look at what's actually in these access tokens."

---

## Section 5: Access Tokens and JWTs (4 minutes)

**[VISUAL: Token structure]**

**Presenter:**

"Access tokens in UiPath are JWTs—JSON Web Tokens. Let's understand what that means.

**[VISUAL: Show JWT structure]**

A JWT has three parts separated by dots:

```
Header.Payload.Signature
```

The header contains metadata about the token—what algorithm was used to sign it.

The payload contains the actual claims—information about the user/application, permissions, expiration time.

The signature proves the token wasn't tampered with. Only the authorization server can create valid signatures.

**[VISUAL: Show decoded JWT]**

Here's what's inside the payload when decoded:

```json
{
  'sub': 'abc-123-def',
  'scope': 'OR.Robots OR.Jobs',
  'iss': 'https://cloud.uipath.com/identity',
  'exp': 1699999999,
  'iat': 1699996399,
  'client_id': 'your-client-id'
}
```

Let me explain each field:

'sub' (subject): Identifies the principal—in this case, your application.

'scope': The permissions this token has. We'll dive into scopes shortly.

'iss' (issuer): Who created this token—UiPath's identity server.

'exp' (expiration): Unix timestamp when this token expires.

'iat' (issued at): When this token was created.

'client_id': Your application's identifier.

**[VISUAL: Show token validation]**

When you send this token to Orchestrator, here's what happens:

1. Orchestrator checks the signature—is this token really from UiPath's identity server?
2. It checks the expiration—is this token still valid?
3. It checks the scopes—does this token have permission for the requested operation?

If all checks pass, the request is processed. If any fail, you get a 401 Unauthorized or 403 Forbidden response.

**[VISUAL: Security note]**

Important security note: JWTs are not encrypted, just encoded. Anyone who has the token can decode it and see what's inside. That's why:
- Never put sensitive data in tokens
- Always transmit tokens over HTTPS
- Never log tokens in plain text
- Store tokens securely

The signature prevents tampering, but not reading. Think of it like a sealed envelope—you can't change the contents without detection, but you can read it if you open it.

Now let's talk about token expiration and refresh."

---

## Section 6: Token Expiration and Refresh (4 minutes)

**[VISUAL: Token lifecycle diagram]**

**Presenter:**

"Access tokens are designed to be short-lived. In UiPath, they typically expire after one hour. This is a security feature—if a token is stolen, it's only useful for that hour.

But if your application runs for longer than an hour—and most do—you need a way to get new tokens without re-entering credentials. That's where refresh tokens come in.

**[VISUAL: Refresh token flow]**

Here's how refresh tokens work:

When you initially authenticate with client credentials, you might receive a refresh token along with the access token. This refresh token is long-lived—it might be valid for days or months.

When your access token expires, instead of starting over with client credentials, you send the refresh token to the token endpoint. The server validates the refresh token and issues a new access token.

This happens seamlessly without user intervention.

**[VISUAL: Code example]**

Let me show you how to implement this:

```python
def get_access_token(client_id, client_secret, refresh_token=None):
    token_url = 'https://cloud.uipath.com/identity_/connect/token'

    if refresh_token:
        # Use refresh token to get new access token
        data = {
            'grant_type': 'refresh_token',
            'refresh_token': refresh_token,
            'client_id': client_id,
            'client_secret': client_secret
        }
    else:
        # Initial authentication with client credentials
        data = {
            'grant_type': 'client_credentials',
            'client_id': client_id,
            'client_secret': client_secret,
            'scope': 'OR.Robots OR.Jobs'
        }

    response = requests.post(token_url, data=data)
    return response.json()
```

**[VISUAL: Automatic refresh strategy]**

A better strategy is proactive refresh—don't wait for the token to expire. Refresh it before expiration:

```python
import time

class TokenManager:
    def __init__(self, client_id, client_secret):
        self.client_id = client_id
        self.client_secret = client_secret
        self.access_token = None
        self.expires_at = 0

    def get_token(self):
        # Check if token is expired or will expire soon (5 min buffer)
        if time.time() >= self.expires_at - 300:
            self.refresh_token()

        return self.access_token

    def refresh_token(self):
        token_data = get_access_token(self.client_id, self.client_secret)
        self.access_token = token_data['access_token']
        self.expires_at = time.time() + token_data['expires_in']
```

**[VISUAL: Best practices]**

Best practices for token management:
- Store access tokens in memory, not on disk
- Refresh tokens proactively before expiration
- Handle 401 Unauthorized responses by refreshing and retrying
- If refresh fails, re-authenticate from scratch
- Log refresh events for monitoring

Proper token management makes your application robust and long-running without constant re-authentication."

---

## Section 7: Scopes and Permissions (4 minutes)

**[VISUAL: Scopes overview]**

**Presenter:**

"Now let's talk about scopes—the permissions that define what a token can do.

When you request an access token, you specify which scopes you need. The token is then limited to those permissions. This implements the principle of least privilege—request only what you need.

**[VISUAL: List of common scopes]**

UiPath defines many scopes. Here are the common ones:

OR.Robots - Read robot information
OR.Robots.Write - Create and modify robots
OR.Jobs - Read job information
OR.Jobs.Write - Create and start jobs
OR.Assets - Read asset values
OR.Assets.Write - Create and modify assets
OR.Queues - Read queue information
OR.Queues.Write - Add queue items
OR.Processes - Read process information
OR.Administration - Administrative operations

**[VISUAL: Scope composition]**

You can request multiple scopes by separating them with spaces:

```python
scope = 'OR.Robots OR.Jobs OR.Assets'
```

This token can read robots, jobs, and assets, but cannot modify anything.

**[VISUAL: Show permission check]**

When you make an API call, Orchestrator checks if your token has the required scope. If it doesn't, you get a 403 Forbidden response:

```python
# Token has scope: 'OR.Robots'
# Trying to create a job
response = requests.post(
    f'{base_url}/odata/Jobs/UiPath.Server.Configuration.OData.StartJobs',
    headers={'Authorization': f'Bearer {token}'},
    json=job_data
)

# Result: 403 Forbidden
# Token doesn't have 'OR.Jobs.Write' scope
```

**[VISUAL: Principle of least privilege]**

Here's a key principle: request the minimum scopes required for your application's function.

If your monitoring dashboard only reads robot status, request only OR.Robots, not OR.Administration.

Why? Two reasons:

First, security. If your credentials are compromised, the attacker is limited to what those scopes allow. With only read permissions, they can't create jobs, modify assets, or delete processes.

Second, auditability. When reviewing access logs, you can clearly see what each application was designed to do based on its scopes.

**[VISUAL: Practical example]**

Let's say you're building an agent that:
- Reads pending queue items
- Processes them
- Updates their status

Required scopes:
```python
scope = 'OR.Queues OR.Queues.Write'
```

That's it. No robot access, no job creation, no asset management. Just queues.

This follows least privilege perfectly."

---

## Section 8: Creating External Applications (3 minutes)

**[VISUAL: UiPath Cloud Portal screenshot]**

**Presenter:**

"Now let's see how to actually get these OAuth credentials. In UiPath Cloud, you create an External Application.

**[VISUAL: Step-by-step walkthrough]**

Here's the process:

Step 1: Log in to cloud.uipath.com and navigate to Admin → External Applications.

Step 2: Click 'Add Application'.

Step 3: Configure the application:
- Name: Give it a descriptive name like 'Python Automation Agent'
- Application Type: Select 'Confidential Application'
- Scopes: Select the permissions you need (OR.Robots, OR.Jobs, etc.)

Step 4: Click 'Add'.

Step 5: Copy the client_id and client_secret. The client_secret is only shown once—store it securely immediately!

**[VISUAL: Show the credentials]**

You'll receive something like:

```
Client ID: abc123-def456-ghi789
Client Secret: super-long-random-string
```

These are your application's credentials. Treat the client_secret like a password—never commit it to git, never share it publicly.

**[VISUAL: Best practices]**

Best practices for External Applications:
- Create separate applications for different purposes (monitoring vs execution vs development)
- Use descriptive names
- Grant minimum required scopes
- Regularly review and remove unused applications
- Rotate secrets periodically
- Monitor usage in audit logs

**[VISUAL: On-Premises note]**

Note: On-Premises Orchestrator has a different process for creating OAuth clients, typically done through the admin interface or configuration files. Consult your Orchestrator admin for the exact process."

---

## Section 9: Role-Based Access Control (3 minutes)

**[VISUAL: RBAC diagram]**

**Presenter:**

"Having a valid token with scopes isn't enough. You also need the right role permissions in Orchestrator. This is where Role-Based Access Control comes in.

**[VISUAL: Show the hierarchy]**

RBAC works like this:

Users are assigned to Roles.
Roles contain sets of Permissions.
Permissions define specific actions (read robots, create jobs, etc.).

So the flow is: User → Role → Permissions.

**[VISUAL: Built-in roles]**

UiPath provides several built-in roles:

Administrator: Has all permissions. Can do everything.

Automation Developer: Can manage processes, assets, and schedules. Typical for developers.

Automation User: Can execute attended automations. For end users.

Robot: Can execute unattended automations. For service accounts.

Viewer: Read-only access. For auditors and managers.

**[VISUAL: Folder-level permissions]**

Here's where it gets interesting: permissions are folder-specific.

A user might be:
- Administrator in the Development folder (full control)
- Viewer in the Production folder (read-only)

Same user, different permissions per folder.

When your application authenticates, it assumes the permissions of the user or service account associated with the External Application.

**[VISUAL: API permissions]**

So even if your token has the OR.Jobs.Write scope, if your user role doesn't have permission to create jobs in that folder, the API call will fail with 403 Forbidden.

Both are required:
1. Token must have the scope
2. User must have the role permission

Think of scopes as what the token can do, and roles as what the user can do. Both must allow the action.

**[VISUAL: Checking permissions]**

When you get a 403 Forbidden error, check two things:
1. Does my token have the required scope?
2. Does my user have the required role permission in this folder?

One of these is the problem."

---

## Section 10: Security Best Practices (4 minutes)

**[VISUAL: Security checklist]**

**Presenter:**

"Let's talk about security best practices for authentication. This is critical—poor authentication practices lead to breaches.

**[VISUAL: Credentials storage]**

First, credential storage:

Never hard-code credentials in your source code:
```python
# ❌ WRONG
client_secret = 'my-secret-123'
```

Use environment variables:
```python
# ✅ CORRECT
client_secret = os.getenv('CLIENT_SECRET')
```

Or use a secret manager like AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault for production.

**[VISUAL: Token handling]**

Second, token handling:

Store access tokens in memory only. Don't persist them to disk in plain text.

Never log tokens:
```python
# ❌ WRONG
logger.debug(f'Token: {access_token}')

# ✅ CORRECT
logger.info('Token obtained successfully')
```

Never put tokens in URLs:
```python
# ❌ WRONG
url = f'https://api.example.com/robots?token={access_token}'

# ✅ CORRECT
headers = {'Authorization': f'Bearer {access_token}'}
```

**[VISUAL: HTTPS enforcement]**

Third, always use HTTPS, never HTTP. HTTP transmits tokens in plain text over the network. Anyone sniffing network traffic can steal them.

All UiPath Cloud URLs use HTTPS by default. For On-Premises, ensure your Orchestrator is configured with TLS certificates.

**[VISUAL: Credential rotation]**

Fourth, rotate credentials regularly. Just like you should change passwords periodically, rotate your client_secret:
- Every 90 days minimum
- Immediately if you suspect compromise
- When team members leave

Go to your External Application in UiPath Cloud and click 'Regenerate Secret'.

**[VISUAL: Monitoring]**

Fifth, monitor authentication events:
- Review audit logs regularly
- Set up alerts for failed authentication attempts
- Monitor for unusual access patterns
- Track token usage

**[VISUAL: Service accounts]**

Sixth, use dedicated service accounts for automation:
- Don't use personal user accounts
- Create specific service accounts with descriptive names
- Assign only required permissions
- Monitor their activity separately

**[VISUAL: Least privilege]**

Seventh, follow least privilege:
- Request minimum scopes needed
- Assign minimum role permissions
- Don't give everything Administrator access
- Review and reduce permissions over time

**[VISUAL: Disaster recovery]**

Finally, have a revocation plan:
- Know how to quickly revoke compromised credentials
- Document the process
- Test it periodically
- Have backup authentication methods

Security isn't a one-time setup—it's an ongoing practice."

---

## Section 11: Common Errors and Troubleshooting (2 minutes)

**[VISUAL: Error codes table]**

**Presenter:**

"Let's talk about common authentication errors you'll encounter and how to fix them.

**[VISUAL: 401 Unauthorized]**

401 Unauthorized means authentication failed:
- Your token expired—refresh it
- Your token is invalid—get a new one
- You forgot the Authorization header—add it
- You used the wrong token format—should be 'Bearer {token}'

**[VISUAL: 403 Forbidden]**

403 Forbidden means authentication succeeded but authorization failed:
- Your token lacks required scopes—request more scopes
- Your user lacks required role permissions—check roles in Orchestrator
- You're accessing wrong folder—switch to correct folder

**[VISUAL: 400 Bad Request]**

400 Bad Request in authentication typically means:
- Invalid client_id or client_secret—check your credentials
- Wrong token endpoint URL—verify the URL
- Missing required parameters like grant_type

**[VISUAL: Debugging tips]**

Debugging tips:

Enable request logging to see exactly what's being sent:
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

Decode your JWT to inspect its contents:
```python
import jwt
decoded = jwt.decode(access_token, options={'verify_signature': False})
print(decoded['scope'])
```

Test your credentials with curl before implementing in Python:
```bash
curl -X POST https://cloud.uipath.com/identity_/connect/token \
  -d 'grant_type=client_credentials' \
  -d 'client_id=your-id' \
  -d 'client_secret=your-secret' \
  -d 'scope=OR.Robots'
```

Most authentication issues come down to: wrong credentials, expired tokens, insufficient permissions, or missing headers. Check these first."

---

## Section 12: Summary and Next Steps (2 minutes)

**[VISUAL: Key takeaways slide]**

**Presenter:**

"Let's summarize what we've learned:

✅ Authentication verifies WHO you are, authorization determines WHAT you can do

✅ UiPath uses OAuth 2.0 with the Client Credentials flow for machine-to-machine auth

✅ Access tokens are temporary (typically 1 hour) JWT tokens

✅ Refresh tokens enable getting new access tokens without re-authentication

✅ Include tokens in the Authorization header with Bearer scheme

✅ Scopes define token permissions—request only what you need

✅ RBAC roles control user permissions—both token scopes and user roles must allow the action

✅ Create External Applications in UiPath Cloud to get OAuth credentials

✅ Follow security best practices: use environment variables, HTTPS, least privilege, and monitoring

**[VISUAL: Practical implementation]**

You now have everything you need to implement authentication in your Python applications. In upcoming modules, we'll use these concepts extensively as we interact with the Orchestrator API.

**[VISUAL: Next module preview]**

In the next module, Tenants and Folders, we'll dive deeper into Orchestrator's organizational structure and how to manage multi-tenant environments programmatically.

**[VISUAL: Lab assignment]**

For now, complete the hands-on lab where you'll:
1. Create an External Application
2. Implement token acquisition in Python
3. Make authenticated API calls
4. Handle token expiration and refresh
5. Implement error handling

Authentication is the foundation of everything we'll build. Get this right, and everything else follows.

Thank you for watching!"

---

## Presenter Notes

### Key Teaching Points
1. **Clear distinction**: Authentication vs Authorization
2. **Why OAuth 2.0**: Show problems it solves
3. **Practical focus**: Emphasize Client Credentials flow
4. **Security first**: Stress best practices throughout
5. **Hands-on code**: Show real Python examples

### Common Questions to Address
- "Can I just use an API key?" → OAuth 2.0 is required, more secure
- "Why do tokens expire?" → Security—limits exposure window
- "What if I need admin access?" → Follow least privilege, request minimum scopes
- "How do I debug auth issues?" → Check token validity, scopes, and roles

### Demo Tips
- Show actual External Application creation in UiPath Cloud
- Demonstrate token request with curl or Python
- Show decoded JWT contents
- Display 401 vs 403 error responses

### Troubleshooting Common Issues
- If token request fails: Check client_id/secret
- If API returns 401: Token expired or invalid
- If API returns 403: Check scopes and roles
- If confused about scopes vs roles: Explain both are required

### Time Management
- Section 1-3: Introduction and OAuth basics (9 minutes)
- Section 4-6: Token flows and management (12 minutes)
- Section 7-9: Scopes, roles, and setup (10 minutes)
- Section 10-12: Security and troubleshooting (4 minutes)

Total: 35 minutes

### Security Emphasis
- Repeat "never commit secrets" multiple times
- Stress HTTPS requirement
- Emphasize least privilege principle
- Show consequences of poor practices

### Code Examples to Prepare
- Working token acquisition script
- Token refresh implementation
- Error handling examples
- TokenManager class for reference
