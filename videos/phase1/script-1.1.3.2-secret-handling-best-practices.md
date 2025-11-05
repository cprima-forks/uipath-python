# Video Script: Secret Handling Best Practices

**Module:** 1.1.3.2
**Duration:** 28 minutes
**Target Audience:** UiPath Agent Developers (Intermediate)
**Prerequisites:** Module 1.1.3.1 (Environment Variables with .env Files)

---

## Section 1: Introduction (2 minutes)

**[VISUAL: Title slide with learning objectives]**

**Presenter:**

"Welcome to Module 1.1.3.2: Secret Handling Best Practices. This module is about one of the most critical aspects of software development: security.

In the previous module, we learned how to use .env files to manage configuration. Now we're going to focus specifically on secrets—API keys, passwords, credentials—and how to handle them securely.

Poor secret handling is one of the most common security vulnerabilities. Every year, thousands of API keys, passwords, and credentials are accidentally exposed in GitHub repos, logs, and error messages. The consequences range from embarrassing to catastrophic: massive AWS bills, data breaches, and compliance violations.

In this module, we'll cover:
- What qualifies as a secret and why it matters
- Common mistakes that expose secrets
- How to implement secret rotation
- Best practices for production systems

By the end, you'll know how to build secure UiPath agents that properly protect sensitive data.

Let's start by understanding what secrets are and why they're so critical."

**[VISUAL: Transition to definition slide]**

---

## Section 2: Understanding Secrets (3 minutes)

**[VISUAL: List of secret types]**

**Presenter:**

"First, let's define what we mean by 'secrets.' Secrets are any sensitive data that grants access to resources or systems.

**[VISUAL: Highlight each type]**

This includes:
- API keys and tokens—like your OpenAI API key or UiPath Orchestrator key
- Passwords and passphrases
- Database credentials—username and password
- Private keys and TLS certificates
- OAuth tokens and refresh tokens
- Encryption keys

**[VISUAL: Contrast with configuration]**

What's NOT a secret? Regular configuration like:
- Application names
- Port numbers
- Log levels
- Feature flags
- Public URLs

The key difference: secrets grant access. If an attacker gets your secrets, they can impersonate you, access your data, or consume your resources.

**[VISUAL: Show real-world incident examples]**

Let me give you some real examples of what happens when secrets are exposed:

In 2020, a developer committed AWS credentials to a public GitHub repo. Within minutes, crypto-mining bots found the credentials and spun up thousands of EC2 instances. The bill: over $50,000 in one day.

In 2021, a major company had API keys in their source code. Attackers used them to access customer data—millions of records exposed.

These aren't rare occurrences. GitHub detects millions of secrets committed to public repos every year.

**[VISUAL: Emphasize the point]**

The message is clear: proper secret handling isn't optional. It's absolutely critical."

---

## Section 3: Common Mistake #1 - Hard-Coding Secrets (4 minutes)

**[VISUAL: Code editor showing hard-coded secrets]**

**Presenter:**

"Let's look at the most common mistake: hard-coding secrets directly in source code.

```python
# ❌ NEVER do this!
def connect_to_api():
    api_key = 'sk-prod-abc123xyz789'
    return requests.get(
        'https://api.example.com/data',
        headers={'Authorization': f'Bearer {api_key}'}
    )
```

**[VISUAL: Highlight the problems]**

This looks simple and works fine, but it has serious problems:

Problem 1: Anyone with access to the code sees your production API key. That includes every developer, everyone who clones the repo, and if the repo is public, literally anyone in the world.

Problem 2: You can't change the key without changing code and redeploying. Want to rotate your credentials? You need a code commit and deployment.

Problem 3: The same key is used in development, staging, and production. A compromised dev key means production is compromised.

Problem 4: If you commit this to git, the secret is in git history forever. Even if you delete it later, it's still there in previous commits. Git never forgets.

**[VISUAL: Show the solution]**

The solution: environment variables.

```python
# ✅ GOOD: Load from environment
import os

def connect_to_api():
    api_key = os.getenv('API_KEY')
    if not api_key:
        raise ValueError('API_KEY not set')

    return requests.get(
        'https://api.example.com/data',
        headers={'Authorization': f'Bearer {api_key}'}
    )
```

**[VISUAL: Show the benefits]**

Now:
- The secret is outside the code
- You can change it without redeployment
- Different environments can have different values
- It's not in version control

This is the fundamental principle: keep secrets out of source code."

---

## Section 4: Common Mistake #2 - Logging Secrets (3 minutes)

**[VISUAL: Code editor showing logging mistake]**

**Presenter:**

"Another common mistake: logging secrets.

```python
# ❌ BAD: Logs the secret
api_key = os.getenv('API_KEY')
logger.info(f'Connecting to API with key: {api_key}')
```

**[VISUAL: Show what happens to logs]**

This seems innocent, but think about what happens to logs:
- They're stored long-term—often years
- They're sent to monitoring systems—Datadog, Splunk, CloudWatch
- They're accessible to many people—operations, support, developers
- They're sometimes included in bug reports or support tickets
- They might end up in public places

**[VISUAL: Show error message example]**

Even worse is logging exception messages that contain secrets:

```python
try:
    db.connect(username, password)
except Exception as e:
    logger.error(f'Connection failed: {e}')
```

Some database libraries include credentials in their error messages: 'Authentication failed for user admin with password secret123'

**[VISUAL: Show the solution]**

The solution: sanitize logs.

```python
# ✅ GOOD: Mask the secret
api_key = os.getenv('API_KEY')
logger.info(f'Connecting to API with key: {api_key[:4]}****')

# ✅ GOOD: Generic error message
try:
    db.connect(username, password)
except Exception as e:
    logger.error('Database connection failed')
    # Don't log the exception message
```

**[VISUAL: Show the principle]**

The principle: only log what's safe for anyone to see. If in doubt, don't log it. You can always add more logging later, but you can never take back a logged secret."

---

## Section 5: Common Mistake #3 - Secrets in URLs (3 minutes)

**[VISUAL: Code editor showing URL mistake]**

**Presenter:**

"The third common mistake: putting secrets in URL query strings.

```python
# ❌ BAD: API key in URL
api_key = os.getenv('API_KEY')
url = f'http://api.example.com/data?key={api_key}'
response = requests.get(url)
```

**[VISUAL: Diagram showing where URLs get logged]**

This is problematic because URLs get logged everywhere:
- Web server access logs
- Proxy server logs
- Browser history
- Network monitoring tools
- Firewall logs

And notice this is using HTTP, not HTTPS, so the URL—including the API key—is transmitted unencrypted over the network.

**[VISUAL: Show the solution]**

The solution: use headers and HTTPS.

```python
# ✅ GOOD: API key in header, HTTPS
api_key = os.getenv('API_KEY')
response = requests.get(
    'https://api.example.com/data',  # HTTPS!
    headers={'Authorization': f'Bearer {api_key}'}
)
```

**[VISUAL: Highlight the two fixes]**

Two critical changes:
1. HTTPS instead of HTTP—encrypted transmission
2. API key in the Authorization header, not the URL

Headers are not logged like URLs are. They're transmitted securely over HTTPS. This is the standard pattern for API authentication."

---

## Section 6: Secret Rotation (4 minutes)

**[VISUAL: Calendar showing rotation schedule]**

**Presenter:**

"Now let's talk about secret rotation—periodically changing your credentials.

Why rotate secrets? Three main reasons:

First, it limits the exposure window. If a secret is compromised but you don't know it, regular rotation means the attacker loses access when you rotate. Without rotation, they have access forever.

Second, it reduces risk from former employees. When someone leaves your company, you should rotate all secrets they had access to. They may have saved credentials before leaving.

Third, it's a compliance requirement. Standards like PCI-DSS and SOC 2 require regular credential rotation.

**[VISUAL: Show rotation schedule]**

When should you rotate?

Minimum: every 90 days for production secrets.
Immediately: if you suspect compromise.
Immediately: when an employee with access leaves.
Immediately: after a security incident.

**[VISUAL: Show rotation process]**

Here's the safe rotation process:

Step 1: Generate a new secret
Step 2: Deploy the new secret WITHOUT removing the old one—both work temporarily
Step 3: Update all clients to use the new secret—gradual rollout
Step 4: Verify all clients have migrated—check usage logs
Step 5: Revoke the old secret—now safe to delete

**[VISUAL: Highlight the key point]**

The critical part: both old and new secrets work during the transition. This avoids downtime. If you immediately revoke the old secret, you'll break any client that hasn't migrated yet.

**[VISUAL: Show automated rotation]**

Better yet: use secret management systems that automate rotation:

```python
import boto3

client = boto3.client('secretsmanager')
client.rotate_secret(
    SecretId='prod/db/password',
    RotationRules={'AutomaticallyAfterDays': 90}
)
```

AWS Secrets Manager, Azure Key Vault, and HashiCorp Vault all support automated rotation. The system handles the entire process for you."

---

## Section 7: Secret Management Systems (4 minutes)

**[VISUAL: Logos of secret management systems]**

**Presenter:**

"For production systems, use dedicated secret management solutions rather than just environment variables.

**[VISUAL: Show AWS Secrets Manager interface]**

The major cloud providers all offer secret management:
- AWS Secrets Manager
- Azure Key Vault
- Google Secret Manager

And there are platform-agnostic solutions:
- HashiCorp Vault
- CyberArk

**[VISUAL: Show feature comparison]**

These systems provide:
- Encrypted storage—secrets encrypted at rest
- Access control—IAM policies define who can access what
- Audit logs—every access is logged
- Automatic rotation—handle rotation for you
- Version history—roll back if needed

**[VISUAL: Code editor showing usage]**

Here's how you use AWS Secrets Manager:

```python
import boto3
import json

def get_secret(secret_name):
    client = boto3.client('secretsmanager')

    try:
        response = client.get_secret_value(SecretId=secret_name)
        return json.loads(response['SecretString'])
    except Exception as e:
        logger.error('Failed to retrieve secret')
        raise

# Usage
secrets = get_secret('prod/database')
db_url = f'postgresql://{secrets['username']}:{secrets['password']}@{secrets['host']}/db'
```

**[VISUAL: Show the benefits]**

The benefits over environment variables:
- Encrypted at rest
- Fine-grained access control
- Complete audit trail
- Automatic rotation
- Centralized management

For production systems, especially in cloud environments, this is the way to go."

---

## Section 8: Environment-Specific Secrets (2 minutes)

**[VISUAL: Diagram showing dev/staging/prod]**

**Presenter:**

"Each environment should have completely different secrets.

```
Development:  API_KEY=test-key-123
Staging:      API_KEY=staging-key-456
Production:   API_KEY=prod-key-789
```

**[VISUAL: Show blast radius comparison]**

Why? Because it limits the blast radius.

If your development API key is compromised—maybe a developer's laptop was stolen—the attacker only gets access to development resources. Production remains secure.

If you use the same key everywhere, compromising the dev key means compromising production.

**[VISUAL: Show implementation]**

This is why we have different .env files:

```
.env.development
.env.staging
.env.production
```

And load the appropriate one based on the ENVIRONMENT variable.

**[VISUAL: Show principle of least privilege]**

Also apply the principle of least privilege. Give each credential the minimum permissions needed.

If your app only reads from the database, use a read-only database user, not an admin user. If credentials are compromised, the damage is limited."

---

## Section 9: Preventing Commits and Detection (3 minutes)

**[VISUAL: Terminal showing git-secrets]**

**Presenter:**

"Prevention is better than cleanup. Use tools to prevent committing secrets in the first place.

git-secrets is a pre-commit hook that scans for patterns:

```bash
git secrets --install
git secrets --register-aws
```

**[VISUAL: Show it catching a commit]**

Now when you try to commit a file with secrets:

```bash
git commit -m 'Add config'
# ERROR: Matched pattern: AWS_SECRET_ACCESS_KEY
```

The commit is blocked. Problem caught before it enters the repo.

**[VISUAL: Show scanning tools]**

For existing repos, use scanning tools:

TruffleHog:
```bash
trufflehog filesystem /path/to/repo
```

GitLeaks:
```bash
gitleaks detect --source=/path/to/repo
```

These scan git history for accidentally committed secrets.

**[VISUAL: Show GitHub secret scanning]**

GitHub also has built-in secret scanning. It's automatic for public repos and available for private repos with GitHub Advanced Security.

When it detects a secret, it alerts you and notifies the secret provider (AWS, etc.) to revoke the credential.

**[VISUAL: Emphasize the workflow]**

The ideal workflow:
1. Pre-commit hooks prevent commits
2. Regular scans catch anything that slips through
3. GitHub scanning is the last line of defense

Multiple layers of protection."

---

## Section 10: Incident Response (2 minutes)

**[VISUAL: Incident response flowchart]**

**Presenter:**

"Despite best efforts, secrets sometimes get exposed. You need an incident response plan.

**[VISUAL: Show the steps]**

If a secret is compromised:

Step 1: Rotate immediately. Don't wait. Don't investigate first. Rotate NOW.

Step 2: Revoke the old secret. Cut off attacker access.

Step 3: Audit usage logs. Check for unauthorized activity. What did they access?

Step 4: Assess the damage. Was data exposed? Were resources consumed?

Step 5: Update procedures. How did this happen? How do we prevent it?

Step 6: Document everything. Create an incident report.

Step 7: Notify affected parties if needed. Depending on what was accessed, you may have legal obligations.

**[VISUAL: Emphasize speed]**

The key: speed. Every minute counts. The faster you rotate, the less time the attacker has.

Have this plan documented before an incident occurs. In the heat of the moment, you don't want to be figuring out what to do."

---

## Section 11: Best Practices Summary (2 minutes)

**[VISUAL: Checklist slide]**

**Presenter:**

"Let me give you a checklist to use before deploying any application:

**[VISUAL: Check each item]**

- No secrets hard-coded in source code
- All secrets in environment variables or secret manager
- .env files in .gitignore
- Different secrets for each environment
- Minimum privilege for all credentials
- Secrets transmitted over HTTPS only
- No secrets in logs or error messages
- Validation at startup for required secrets
- Rotation policy defined and documented
- Audit logging enabled
- Incident response plan documented

**[VISUAL: Show a real code example]**

Here's what proper secret handling looks like in practice:

```python
import os
import sys

class SecureConfig:
    REQUIRED_SECRETS = ['ORCHESTRATOR_API_KEY', 'DATABASE_PASSWORD']

    def __init__(self):
        self._validate_secrets()
        self._load_secrets()

    def _validate_secrets(self):
        missing = [s for s in self.REQUIRED_SECRETS if not os.getenv(s)]
        if missing:
            print(f'Missing secrets: {missing}')
            sys.exit(1)

    def _load_secrets(self):
        self.orchestrator_key = os.getenv('ORCHESTRATOR_API_KEY')
        self.db_password = os.getenv('DATABASE_PASSWORD')
        logging.info('All secrets loaded successfully')
```

Validation at startup, fail-fast if missing, log success but not the actual secrets."

---

## Section 12: Summary and Next Steps (1 minute)

**[VISUAL: Key takeaways slide]**

**Presenter:**

"Let's recap the critical points:

✅ Never hard-code secrets in source code
✅ Never commit secrets to version control
✅ Never log secrets in plain text
✅ Always use HTTPS for transmission
✅ Rotate secrets regularly—every 90 days minimum
✅ Use different secrets per environment
✅ Grant minimum necessary permissions
✅ Fail fast if secrets are missing
✅ Use secret management systems for production

**[VISUAL: Next module preview]**

This completes our Environment Management section! You now know how to handle configuration and secrets securely.

In the next module, we'll move into UiPath-specific content, starting with UiPath Orchestrator Architecture. You'll learn about Orchestrator components, architecture patterns, and deployment models.

**[VISUAL: Lab assignment]**

For now, complete the hands-on lab where you'll secure an application's credentials, implement rotation, and set up proper secret handling.

Secret handling is one of the most important skills in security. Master these principles and you'll build applications that are secure by design.

Thank you for watching!"

---

## Presenter Notes

### Key Teaching Points
1. **Start with real consequences**: Show real incidents, real costs
2. **Demonstrate each mistake**: Show the bad code, explain why it's bad
3. **Provide clear solutions**: Always follow bad example with good example
4. **Emphasize automation**: Tools and systems prevent human error
5. **Make it actionable**: Give checklists and concrete steps

### Common Questions to Address
- "Why not just use strong passwords?" → Doesn't help if they're exposed in code/logs
- "Is secret rotation really necessary?" → Yes, it's industry standard and compliance requirement
- "Can I use .env files in production?" → Better to use platform secrets or secret managers
- "What if I already committed a secret?" → Rotate immediately, then clean history

### Demo Tips
- Show actual secret scanning tools in action
- Demonstrate git-secrets blocking a commit
- Show a secret management system interface
- Run a tool that finds secrets in code

### Troubleshooting Common Issues
- If students think rotation is too much work: Emphasize automation
- If they think it won't happen to them: Show statistics on exposed secrets
- If they want to use defaults: Explain why fail-fast is better

### Time Management
- Section 1-3: Mistakes and problems (12 minutes)
- Section 4-7: Rotation and management (13 minutes)
- Section 8-12: Prevention and best practices (3 minutes)

Total: 28 minutes

### Security Emphasis
- Repeat key principles multiple times
- Use real-world examples and incidents
- Make it personal—"this could be your AWS bill"
- End with actionable checklist

### Code Examples to Prepare
- Have bad examples ready to show
- Have good examples as solutions
- Test all tools before recording
- Prepare incident response scenarios
