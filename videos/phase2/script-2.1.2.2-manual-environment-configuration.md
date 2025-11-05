# Video Script: Manual Environment Configuration
## Module 2.1.2.2 - SDK Fundamentals

**Duration:** 22 minutes
**Target Audience:** Developers managing multi-environment deployments
**Prerequisites:** Modules 2.1.1.1-2.1.2.1, understanding of environment variables
**Learning Outcomes:** Build robust multi-environment configuration system

---

## Scene 1: Introduction (1.5 minutes)

### Visual: Title slide

**[On camera - Instructor]**

Welcome to the final module in our Installation and Setup section: Manual Environment Configuration. Congratulations on making it this far - you've learned installation, versioning, dependencies, and authentication. Now we're tying it all together.

### Visual: Multiple environment diagram (dev/staging/prod)

**[Voice over visual]**

In real-world development, you work with multiple environments: your local development machine, a staging server for testing, and production serving real users. Each needs different settings - different URLs, different credentials, different timeouts.

### Visual: Learning objectives

**[On camera - Instructor]**

In this video, you'll learn how to manage configurations professionally using the 12-factor app methodology. By the end, you'll have a production-ready configuration system that scales from solo projects to enterprise deployments.

Let's build it!

---

## Scene 2: The Problem (2 minutes)

### Visual: Code with hardcoded values (with red X's)

**[On camera - Instructor]**

Let me show you what NOT to do - a pattern I see far too often.

### Visual: Bad code example

**[Voice over code]**

```python
# DON'T DO THIS
ORCHESTRATOR_URL = "https://prod.company.com/orchestrator_"
CLIENT_SECRET = "abc123secret"
TIMEOUT = 30
```

This looks simple, but it creates serious problems.

### Visual: Problems list appearing

**[Voice over]**

**Problem 1:** To switch from dev to prod, you must edit code. That means different code runs in different environments - a testing nightmare.

**Problem 2:** Your production credentials are committed to version control. Anyone with repo access has production access.

**Problem 3:** Changing a timeout requires code deployment. Configuration should be deployment-independent.

**Problem 4:** No easy way to test locally with dev settings then deploy to prod with prod settings.

### Visual: 12-Factor App logo

**[On camera - Instructor]**

The solution is the 12-Factor App methodology, specifically factor #3: Store config in environment variables. Let me show you how.

---

## Scene 3: The Solution - Environment Files (3 minutes)

### Visual: Project structure diagram

**[On camera - Instructor]**

We'll use a three-layer approach: environment files for values, a configuration class for loading and validation, and application code that consumes configuration.

### Screen recording - creating project structure

**[Voice over screen recording]**

Let's set this up. Create a project directory:

```bash
mkdir uipath-config-demo
cd uipath-config-demo
```

Now create environment files for each environment. Start with development:

```bash
cat > .env.dev << EOF
ORCHESTRATOR_URL=https://cloud.uipath.com/myorg/dev/orchestrator_
ORCHESTRATOR_FOLDER=Development
UIPATH_CLIENT_ID=dev_abc123
UIPATH_CLIENT_SECRET=dev_secret_xyz
ENVIRONMENT=dev
LOG_LEVEL=DEBUG
API_TIMEOUT=30
MAX_RETRIES=3
EOF
```

### Visual: Highlight environment-specific values

Notice these are development values - a dev Orchestrator URL, dev credentials.

Now create production config:

```bash
cat > .env.prod << EOF
ORCHESTRATOR_URL=https://cloud.uipath.com/myorg/prod/orchestrator_
ORCHESTRATOR_FOLDER=Production
UIPATH_CLIENT_ID=prod_xyz789
UIPATH_CLIENT_SECRET=prod_secret_abc
ENVIRONMENT=prod
LOG_LEVEL=INFO
API_TIMEOUT=60
MAX_RETRIES=5
EOF
```

### Visual: Comparison of dev vs prod values

**[On camera - Instructor]**

See the differences? Production has longer timeouts, more retries, less verbose logging, and of course, different URLs and credentials.

---

## Scene 4: Documentation and Security (2 minutes)

### Screen recording - creating .env.example

**[On camera - Instructor]**

Before we go further, two critical files: .env.example and .gitignore.

**[Voice over screen recording]**

Create an example file to document what variables are needed:

```bash
cat > .env.example << EOF
# Orchestrator Configuration
ORCHESTRATOR_URL=https://cloud.uipath.com/org/tenant/orchestrator_
ORCHESTRATOR_FOLDER=Production

# OAuth Credentials
UIPATH_CLIENT_ID=your_client_id_here
UIPATH_CLIENT_SECRET=your_client_secret_here

# Application Settings
ENVIRONMENT=dev
LOG_LEVEL=INFO
API_TIMEOUT=30
MAX_RETRIES=3
EOF
```

### Visual: Highlight - no secrets in example file

This documents the structure without exposing real values. We commit THIS file, not the real .env files.

### Visual: .gitignore file

Now protect your secrets:

```bash
cat > .gitignore << EOF
.env
.env.*
!.env.example
__pycache__/
*.pyc
EOF
```

### Visual: Annotation explaining the pattern

That exclamation mark means "except .env.example" - we ignore all .env files but allow the example through.

---

## Scene 5: Configuration Class (4 minutes)

### Visual: Configuration class diagram

**[On camera - Instructor]**

Now for the heart of the system: a configuration class that loads, validates, and provides access to settings.

### Screen recording - creating config.py

**[Voice over screen recording]**

Create config.py:

```python
import os
from dotenv import load_dotenv

class OrchestratorConfig:
    def __init__(self, env_file=None):
        # Load environment file
        if env_file:
            load_dotenv(env_file)
        else:
            load_dotenv()  # Uses .env by default
```

### Visual: Highlight load_dotenv with env_file parameter

This flexibility lets us choose which environment file to load.

```python
        # Required variables
        self.orchestrator_url = self._get_required("ORCHESTRATOR_URL")
        self.folder = self._get_required("ORCHESTRATOR_FOLDER")
        self.client_id = self._get_required("UIPATH_CLIENT_ID")
        self.client_secret = self._get_required("UIPATH_CLIENT_SECRET")
```

### Visual: Highlight _get_required method

We'll define _get_required to ensure these critical values are set.

```python
        # Optional with defaults
        self.environment = os.getenv("ENVIRONMENT", "dev")
        self.log_level = os.getenv("LOG_LEVEL", "INFO")
        self.api_timeout = int(os.getenv("API_TIMEOUT", "30"))
        self.max_retries = int(os.getenv("MAX_RETRIES", "3"))
```

### Visual: Highlight type conversion (int())

**[On camera - Instructor]**

Critical detail: environment variables are always strings. We convert numeric values to integers.

### Visual: Helper method

```python
    def _get_required(self, key):
        value = os.getenv(key)
        if not value:
            raise ValueError(f"Required variable '{key}' not set")
        return value
```

This enforces that required variables must be present.

---

## Scene 6: Validation (2 minutes)

### Screen recording - adding validation method

**[On camera - Instructor]**

Configuration loading is just the start. We need to validate values make sense.

**[Voice over screen recording]**

Add a validate method:

```python
    def validate(self):
        # URL must use HTTPS
        if not self.orchestrator_url.startswith('https://'):
            raise ValueError("Orchestrator URL must use HTTPS")

        # Timeout must be reasonable
        if self.api_timeout < 1 or self.api_timeout > 300:
            raise ValueError("Timeout must be between 1-300 seconds")

        # Retries can't be negative
        if self.max_retries < 0:
            raise ValueError("Max retries must be non-negative")

        return True
```

### Visual: Each validation highlighted

**[On camera - Instructor]**

This catches configuration errors immediately at startup - the "fail fast" principle. Much better than mysterious failures deep in your application.

---

## Scene 7: Using the Configuration (3 minutes)

### Screen recording - creating main.py

**[On camera - Instructor]**

Now let's use our configuration system in an application.

**[Voice over screen recording]**

Create main.py:

```python
import sys
from config import OrchestratorConfig

def main():
    # Determine environment from command line
    env = sys.argv[1] if len(sys.argv) > 1 else 'dev'
    env_file = f'.env.{env}'

    print(f"Loading configuration from {env_file}")

    try:
        # Load and validate configuration
        config = OrchestratorConfig(env_file=env_file)
        config.validate()

        print(f"Configuration loaded successfully!")
        print(f"Environment: {config.environment}")
        print(f"Orchestrator: {config.orchestrator_url}")
        print(f"Folder: {config.folder}")
        print(f"Timeout: {config.api_timeout}s")

    except ValueError as e:
        print(f"Configuration error: {e}")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

### Visual: Running with different environments

Let's test it:

```bash
# Development
python main.py dev
```

### Visual: Output showing dev config

```bash
# Production
python main.py prod
```

### Visual: Output showing prod config

**[On camera - Instructor]**

Same code, different configuration. That's the power of environment-based config.

---

## Scene 8: Advanced Pattern - Pydantic (2 minutes)

### Visual: Pydantic logo

**[On camera - Instructor]**

For production applications, I recommend using Pydantic for automatic validation and type safety.

### Screen recording - Pydantic version

**[Voice over screen recording]**

```python
from pydantic import BaseSettings, Field, validator

class OrchestratorConfig(BaseSettings):
    orchestrator_url: str
    folder: str
    client_id: str
    client_secret: str
    environment: str = "dev"
    api_timeout: int = Field(default=30, ge=1, le=300)
    max_retries: int = Field(default=3, ge=0)

    @validator('orchestrator_url')
    def must_be_https(cls, v):
        if not v.startswith('https://'):
            raise ValueError('must use HTTPS')
        return v

    class Config:
        env_file = '.env'
        case_sensitive = False
```

### Visual: Highlight type annotations

**[Voice over]**

Type annotations give you automatic validation. Field constraints (ge=1, le=300) enforce ranges. Custom validators handle complex logic.

**[On camera - Instructor]**

Pydantic handles all the boilerplate we wrote manually. It's the professional choice.

---

## Scene 9: Production Secrets Management (2 minutes)

### Visual: Cloud providers (Azure, AWS, GCP)

**[On camera - Instructor]**

A word about production: .env files are fine for development, but production should use proper secret management systems.

### Visual: Architecture diagram showing secret manager integration

**[Voice over visual]**

For Azure, use Key Vault. For AWS, use Secrets Manager. For Google Cloud, use Secret Manager. These services provide:

- Encryption at rest and in transit
- Access audit logs
- Automatic rotation
- Fine-grained access control
- High availability

### Visual: Code example

**[Voice over]**

Integration is straightforward:

```python
if os.getenv('ENVIRONMENT') == 'prod':
    # Production: Load from Azure Key Vault
    secrets = load_from_azure_keyvault()
    config = OrchestratorConfig(**secrets)
else:
    # Dev: Load from .env file
    config = OrchestratorConfig()
```

---

## Scene 10: Best Practices Recap (1.5 minutes)

### Visual: Best practices checklist

**[On camera - Instructor]**

Let's recap the configuration best practices we've covered:

### Visual: Practices appearing one by one

**[Voice over]**

1. **Never commit secrets** - .env files in .gitignore
2. **Document requirements** - .env.example committed
3. **Validate early** - Fail fast at startup
4. **Type conversion** - Environment variables are strings
5. **Environment-specific** - Separate files per environment
6. **Production secrets** - Use secret managers, not .env
7. **Hide in logs** - Never log full secrets

---

## Scene 11: Hands-On Exercise (30 seconds)

### Visual: Exercise instructions

**[On camera - Instructor]**

Time for hands-on practice! Pause and complete this exercise:

1. Create .env.dev and .env.prod files for your Orchestrator
2. Implement the OrchestratorConfig class with full validation
3. Add .env.example documentation
4. Test switching between environments
5. Bonus: Implement the Pydantic version

This should take about 30 minutes. Solution in course materials.

---

## Scene 12: Conclusion (30 seconds)

### Visual: Module completion graphic

**[On camera - Instructor]**

Congratulations! You've completed the Installation and Setup section. You now have all the fundamentals: SDK installation, version management, dependency handling, OAuth authentication, and professional configuration management.

### Visual: Next section preview

In the next section, Core Services, we'll use everything you've learned to interact with Orchestrator services - starting jobs, managing queues, handling assets, and more.

### Visual: End card

Thank you for watching, and I'll see you in the Core Services section!

---

## Production Notes

**Graphics needed:**
- Multi-environment architecture diagram
- Three-layer configuration architecture
- .gitignore pattern visualization
- Configuration class UML
- Validation flow diagram
- Production secrets architecture

**Screen recordings:**
- Complete project setup from scratch
- Creating all .env files
- Implementing config.py step-by-step
- Running with different environments
- Pydantic implementation
- Error scenarios (missing variables, invalid values)

**Code examples:**
- Bad example (hardcoded)
- Good example (environment-based)
- Complete OrchestratorConfig class
- Pydantic version
- Azure Key Vault integration

**Callouts/Annotations:**
- Highlight .gitignore pattern negation
- Point out type conversions
- Mark validation checks
- Show environment switching

**Pacing notes:**
- Clear explanation of 12-factor methodology
- Methodical code walkthrough
- Emphasize security practices
- Practical demonstration of environment switching

**Accessibility:**
- Accurate captions for technical terms
- Verbal description of all diagrams
- High-contrast code display
- Clear visual separation of different environment files
