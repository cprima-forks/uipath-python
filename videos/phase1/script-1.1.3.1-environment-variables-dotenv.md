# Video Script: Environment Variables with .env Files

**Module:** 1.1.3.1
**Duration:** 20 minutes
**Target Audience:** UiPath Agent Developers (Beginner to Intermediate)
**Prerequisites:** Module 1.1.1.4 (File I/O Operations)

---

## Section 1: Introduction (2 minutes)

**[VISUAL: Title slide with module objectives]**

**Presenter:**

"Welcome to Module 1.1.3.1: Environment Variables with .env Files. This module is all about managing configuration for your UiPath agents securely and efficiently.

If you've ever hard-coded an API key in your code, or wondered how to use different database URLs for development versus production, this module is for you.

We're going to learn:
- What environment variables are and why they matter
- How to use .env files to manage configuration
- How to load variables with python-dotenv
- How to organize configuration for multiple environments

By the end, you'll be able to build agents that are secure, configurable, and ready for any environment.

Let's start with a common problem."

**[VISUAL: Transition to code editor]**

---

## Section 2: The Configuration Problem (3 minutes)

**[VISUAL: Code editor showing hard-coded secrets]**

**Presenter:**

"Here's how NOT to manage configuration:

```python
# ❌ BAD CODE - Don't do this!
API_KEY = 'sk-prod-abc123xyz789'
DATABASE_URL = 'postgresql://admin:password123@prod-server:5432/mydb'
DEBUG = False
ORCHESTRATOR_URL = 'https://cloud.uipath.com'
```

This looks simple, but it has serious problems:

**[VISUAL: Highlight each problem]**

Problem 1: Secrets are exposed in your source code. Anyone with access to the repo sees your production API keys.

Problem 2: You can't change configuration without changing code. Want to point to a different database? You have to modify the Python file, commit, and deploy.

Problem 3: The same code can't run in different environments. Your development machine, staging server, and production server all need different values, but they're hard-coded to one set.

Problem 4: If you commit this to a public GitHub repo, your secrets are public forever. Even if you remove them later, they're still in git history.

**[VISUAL: Show git history with secrets]**

This is a security nightmare. So what's the solution?

**[VISUAL: Transition to environment variables]**

The solution is environment variables—storing configuration outside your code."

---

## Section 3: Environment Variables Basics (3 minutes)

**[VISUAL: Terminal showing environment variables]**

**Presenter:**

"Environment variables are key-value pairs set in the operating system:

```bash
# Linux/Mac
export API_KEY='secret123'
export DATABASE_URL='postgresql://localhost:5432/mydb'

# Windows
set API_KEY=secret123
set DATABASE_URL=postgresql://localhost:5432/mydb
```

**[VISUAL: Code editor showing Python access]**

You access them in Python using the os module:

```python
import os

api_key = os.getenv('API_KEY')
database_url = os.getenv('DATABASE_URL')
debug = os.getenv('DEBUG', 'False') == 'True'
```

**[VISUAL: Run the code, show it working]**

Now your secrets are outside the code. You can change them without touching Python files. Different environments can have different values.

But there's a problem: setting environment variables manually is tedious. If you have 20 configuration values, you need to run 20 export commands every time you set up a new development machine.

**[VISUAL: Show long list of export commands]**

This is where .env files come in."

---

## Section 4: Introduction to .env Files (4 minutes)

**[VISUAL: Show .env file]**

**Presenter:**

"A .env file is a simple text file that stores environment variables:

```
# .env
API_KEY=secret123
DATABASE_URL=postgresql://localhost:5432/mydb
DEBUG=True
MAX_RETRIES=3
TIMEOUT=30
```

**[VISUAL: Highlight the format]**

The format is simple: KEY=value, one per line. No spaces around the equals sign. No quotes needed unless the value contains spaces.

To use .env files in Python, we need the python-dotenv library:

```bash
pip install python-dotenv
```

**[VISUAL: Code editor showing usage]**

Then load the .env file in your Python code:

```python
from dotenv import load_dotenv
import os

# Load .env file
load_dotenv()

# Now access variables as before
api_key = os.getenv('API_KEY')
database_url = os.getenv('DATABASE_URL')
debug = os.getenv('DEBUG', 'False') == 'True'
```

**[VISUAL: Run the code, show it loading from .env]**

load_dotenv() reads the .env file and loads all the variables into the environment. After that, os.getenv() works just like before.

**[VISUAL: Show directory structure]**

By convention, place .env in your project root:

```
myproject/
  ├── .env          # Your environment variables
  ├── .gitignore    # .env should be here!
  ├── main.py
  └── config.py
```

This is much easier than setting variables manually. Just create one .env file and you're done."

---

## Section 5: .env File Syntax (2 minutes)

**[VISUAL: Code editor showing .env syntax]**

**Presenter:**

"Let me show you the full syntax for .env files:

```
# Comments start with hash
API_KEY=simple-value

# No spaces around equals (though they work)
DATABASE_URL=postgresql://localhost/db

# Quotes are optional for simple values
NAME=value
NAME='value'  # Same result

# Use quotes for values with spaces
MESSAGE='Hello World'
WELCOME_TEXT='Welcome to our app!'

# Multiline values for certificates
PRIVATE_KEY='-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
-----END RSA PRIVATE KEY-----'
```

**[VISUAL: Highlight key points]**

A few important notes:

1. Comments start with #
2. Format is KEY=value
3. Quotes are optional unless you have spaces
4. No variable expansion—$HOME won't work
5. Values are always strings in the file

**[VISUAL: Show type conversion in Python]**

Convert to other types in Python:

```python
max_retries = int(os.getenv('MAX_RETRIES', '3'))
timeout = float(os.getenv('TIMEOUT', '30.0'))
debug = os.getenv('DEBUG', 'False').lower() == 'true'
```

Simple and straightforward."

---

## Section 6: Security Best Practices (3 minutes)

**[VISUAL: Warning symbol]**

**Presenter:**

"Here's the most important rule: NEVER commit .env files to version control.

**[VISUAL: Show .gitignore]**

Always add .env to your .gitignore:

```
# .gitignore
.env
.env.local
.env.*.local
*.env
```

**[VISUAL: Show consequences diagram]**

Why? Because if you commit .env to git:
- Your secrets are in the git history forever
- Anyone who clones the repo gets your production credentials
- Public repos make your secrets public to the world
- It's nearly impossible to fully remove them from history

**[VISUAL: Show GitHub secret scanning alert]**

GitHub even scans for exposed secrets and will alert you, but the damage is done.

**[VISUAL: Show .env.example]**

Instead, commit a .env.example file as a template:

```
# .env.example
API_KEY=your-api-key-here
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname
DEBUG=False
MAX_RETRIES=3
```

**[VISUAL: Show workflow]**

The workflow:
1. Developer clones repo
2. Copies .env.example to .env
3. Fills in real values in .env
4. .env stays local, never committed

This keeps your secrets safe."

---

## Section 7: Multiple Environments (3 minutes)

**[VISUAL: Show multiple .env files]**

**Presenter:**

"Real applications run in multiple environments: development, staging, production. Each needs different configuration.

The best practice is separate .env files:

```
.env.development
.env.staging
.env.production
```

**[VISUAL: Show .env.development]**

Development config:

```
# .env.development
API_KEY=test-key-123
DATABASE_URL=postgresql://localhost:5432/myapp_dev
DEBUG=True
LOG_LEVEL=DEBUG
ORCHESTRATOR_URL=https://staging.uipath.com
```

**[VISUAL: Show .env.production]**

Production config:

```
# .env.production
API_KEY=sk-prod-abc123xyz789
DATABASE_URL=postgresql://prod-server:5432/myapp
DEBUG=False
LOG_LEVEL=WARNING
ORCHESTRATOR_URL=https://cloud.uipath.com
```

**[VISUAL: Code editor showing dynamic loading]**

Load the right file based on an environment variable:

```python
import os
from dotenv import load_dotenv

# Get environment (default to development)
env = os.getenv('ENVIRONMENT', 'development')

# Load appropriate .env file
load_dotenv(f'.env.{env}')

# Use configuration
api_key = os.getenv('API_KEY')
```

**[VISUAL: Terminal showing usage]**

Run with different environments:

```bash
# Development
python main.py

# Production
ENVIRONMENT=production python main.py
```

This pattern keeps environments isolated and clear."

---

## Section 8: Configuration Class Pattern (2 minutes)

**[VISUAL: Code editor showing Config class]**

**Presenter:**

"For larger applications, organize configuration in a class:

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    # API Settings
    API_KEY = os.getenv('API_KEY')
    API_BASE_URL = os.getenv('API_BASE_URL', 'https://api.example.com')

    # Database
    DATABASE_URL = os.getenv('DATABASE_URL')
    DATABASE_POOL_SIZE = int(os.getenv('DATABASE_POOL_SIZE', '10'))

    # Application
    DEBUG = os.getenv('DEBUG', 'False').lower() == 'true'
    LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')
    MAX_WORKERS = int(os.getenv('MAX_WORKERS', '4'))
```

**[VISUAL: Show usage]**

Use it throughout your application:

```python
from config import Config

if Config.DEBUG:
    print(f'Connecting to {Config.DATABASE_URL}')

client = APIClient(
    api_key=Config.API_KEY,
    base_url=Config.API_BASE_URL
)
```

**[VISUAL: Highlight benefits]**

Benefits:
- Single source of truth
- Type conversion in one place
- Easy to import and use
- Clear structure"

---

## Section 9: Validation and Best Practices (1 minute)

**[VISUAL: Code editor showing validation]**

**Presenter:**

"Always validate required configuration at startup:

```python
from dotenv import load_dotenv
import os
import sys

load_dotenv()

REQUIRED = ['API_KEY', 'DATABASE_URL', 'ORCHESTRATOR_URL']

missing = [var for var in REQUIRED if not os.getenv(var)]

if missing:
    print(f'Error: Missing required variables: {', '.join(missing)}')
    print('Please check your .env file')
    sys.exit(1)
```

**[VISUAL: Show error output]**

This fails fast with a clear message instead of cryptic errors later.

Better to find configuration problems immediately than halfway through processing."

---

## Section 10: Summary and Next Steps (1 minute)

**[VISUAL: Key takeaways slide]**

**Presenter:**

"Let's recap:

✅ Environment variables separate configuration from code
✅ .env files make managing variables easy
✅ python-dotenv loads .env into the environment
✅ Never commit .env to version control
✅ Use .env.example as a template
✅ Create separate files for different environments
✅ Validate required variables at startup

**[VISUAL: Next module preview]**

In the next module, Secret Handling Best Practices, we'll dive deeper into:
- Identifying sensitive data
- Using secret management systems
- Rotating credentials
- Following security principles

**[VISUAL: Lab assignment]**

For now, complete the hands-on lab where you'll build a configuration manager for a UiPath agent, supporting multiple environments with validation.

Environment variables and .env files are fundamental to building professional applications. Master them and your agents will be secure, flexible, and production-ready.

Thank you for watching!"

---

## Presenter Notes

### Key Teaching Points
1. **Start with the problem**: Show hard-coded secrets and why they're bad
2. **Show the solution progression**: Manual env vars → .env files → organized config
3. **Emphasize security**: Never commit .env files
4. **Demonstrate real usage**: Load files, access variables, type conversion
5. **Show multi-environment pattern**: Essential for real-world applications

### Common Questions to Address
- "Why not just use a config.py file?" → Secrets would still be in version control
- "Can I use .env in production?" → Better to use platform env vars or secret managers
- "What if .env doesn't exist?" → load_dotenv() fails silently (no error)
- "How do I share secrets with my team?" → Use secure channels, not git

### Demo Tips
- Show actual .env file being read
- Demonstrate missing variable error
- Show .gitignore protecting .env
- Run with different environment files

### Troubleshooting Common Issues
- If load_dotenv() doesn't work: Check file is in project root
- If variables are None: Check .env syntax (no spaces, correct format)
- If changes don't apply: Restart Python process after editing .env

### Time Management
- Section 1-2: Problem and solution (5 minutes)
- Section 3-4: Environment variables and .env basics (7 minutes)
- Section 5-7: Syntax, security, multi-environment (8 minutes)
- Section 8-10: Patterns and summary (0 minutes)

Total: 20 minutes

### Code Examples to Prepare
- Have working .env file ready
- Test all code before recording
- Prepare error examples
- Show different environment files

### Security Emphasis
- Repeat "never commit .env" multiple times
- Show real consequences (GitHub alerts)
- Demonstrate .gitignore protection
- Recommend .env.example pattern
