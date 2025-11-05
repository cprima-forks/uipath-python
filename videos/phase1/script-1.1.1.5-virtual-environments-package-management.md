# Video Script: Virtual Environments and Package Management

**Module**: 1.1.1.5
**Duration**: 30 minutes
**Target Audience**: Beginners learning Python for UiPath Agent development
**Prerequisites**: Module 1.1.1.4 (File I/O Operations)

---

## Section 1: Introduction and The Problem [00:00 - 03:00]

### Presenter Notes
- Start with a relatable problem scenario
- Make the pain point vivid
- Present virtual environments as the elegant solution
- Set expectations for comprehensive coverage

### On-Screen Content
**TITLE SLIDE**
- "Virtual Environments and Package Management"
- "Module 1.1.1.5 - Dependency Management for Agents"

**THE PROBLEM:**
```
Project A: needs requests 2.28.0
Project B: needs requests 2.31.0
System: Can only have ONE version!
💥 Conflict!
```

### Script

**[00:00]** PRESENTER:
"Welcome back! We've learned Python fundamentals - variables, functions, error handling, and file I/O. Today we're tackling something absolutely critical for professional development: virtual environments and package management.

**[00:30]** Let me start with a problem you'll definitely encounter. Imagine you're working on two UiPath agents. Agent A uses the requests library version 2.28.0. Agent B needs requests 2.31.0 for a new feature. 

Your system Python can only have ONE version of requests installed at a time. If you upgrade for Agent B, you break Agent A. If you downgrade for Agent A, Agent B doesn't work. You're stuck.

**[01:15]** Or worse: you develop an agent on your machine, everything works perfectly. You deploy it to production. Crash! Why? Because production has different package versions. 'Works on my machine' is not acceptable for production code.

**[01:45]** The solution to both problems: virtual environments. They let each project have its own isolated set of packages. No conflicts. Clean deployments. Professional reliability.

**[02:00]** By the end of this video, you'll be able to:
- Create and manage virtual environments
- Install packages with pip and the new, blazing-fast uv tool
- Manage dependencies with requirements files
- Apply professional packaging practices

**[02:30]** This knowledge is non-negotiable for UiPath Agent development. Every professional Python project uses virtual environments. Let's learn how."

---

## Section 2: What are Virtual Environments? [03:00 - 06:30]

### Presenter Notes
- Explain the concept clearly with analogies
- Show directory structure
- Emphasize isolation benefits
- Connect to agent development workflow

### On-Screen Content
**CODE EXAMPLE 1: Creating a Virtual Environment**
```bash
# Create virtual environment
python -m venv venv

# Directory created:
venv/
├── bin/         # (Linux/Mac) Executables
├── lib/         # Installed packages
└── pyvenv.cfg   # Configuration
```

**VISUAL: Sandbox Diagram**
- System Python (shared)
- Project A venv (isolated)
- Project B venv (isolated)

### Script

**[03:00]** PRESENTER:
"So what exactly is a virtual environment? Think of it as a sandbox - an isolated Python installation for your project.

**[03:15]** When you create a virtual environment, Python creates a directory containing:
- A link to your system Python interpreter
- Its own site-packages folder for installed packages  
- Its own pip package installer
- Activation scripts

**[DEMONSTRATE CODE EXAMPLE 1]**

**[03:45]** This command - python -m venv venv - creates a virtual environment in a directory named 'venv'. You can name it anything, but 'venv' is the convention.

**[04:00]** The magic is isolation. Each virtual environment has its own site-packages directory. When you install a package, it goes there - not in your system Python. Agent A's venv can have requests 2.28.0, Agent B's venv can have 2.31.0, and they never interfere.

**[04:30]** Why is this crucial for UiPath agents?

First, development-production parity. Your agent uses the exact same package versions in development and production. No surprises.

Second, clean system Python. You're not polluting your system with dozens of packages from different projects.

Third, reproducibility. You can recreate the exact environment anywhere - your teammate's machine, a server, a Docker container.

**[05:15]** Here's the workflow: One virtual environment per project. When you start a new agent, create a new venv. Keep them isolated. This is how professionals work.

**[05:30]** You might be thinking: 'Isn't this extra work?' Initially, yes. But the first time you avoid a version conflict or deploy successfully because your environment is controlled - you'll understand the value. This small investment saves enormous headaches."

---

## Section 3: Creating and Activating Virtual Environments [06:30 - 10:30]

### Presenter Notes
- Show creation process step-by-step
- Demonstrate activation on multiple platforms
- Show what activation actually does
- Practice checking active environment

### On-Screen Content
**CODE EXAMPLE 2: Complete Workflow**
```bash
# 1. Create virtual environment
python -m venv venv

# 2. Activate (Linux/Mac)
source venv/bin/activate

# 2. Activate (Windows)
venv\Scripts\activate

# Prompt changes:
(venv) user@machine:~/project$

# 3. Verify
which python
python --version

# 4. Work on project...

# 5. Deactivate when done
deactivate
```

### Script

**[06:30]** PRESENTER:
"Let's create and activate a virtual environment step by step.

**[DEMONSTRATE CODE EXAMPLE 2]**

**[06:45]** Step one: python -m venv venv. The -m flag runs the venv module. The second 'venv' is the directory name. This takes a few seconds - Python is setting up the isolated environment.

**[07:00]** Step two: Activate it. On Linux and Mac, use 'source venv/bin/activate'. On Windows, it's 'venv\Scripts\activate' - no 'source' needed.

**[07:20]** Notice your prompt changes. It now shows (venv) at the beginning. This visual indicator tells you you're in a virtual environment. Always check your prompt before installing packages!

**[07:40]** What did activation do? It modified your PATH environment variable. Now when you type 'python' or 'pip', you're using the versions from venv, not your system Python.

Let's verify:

```bash
which python
# /home/user/project/venv/bin/python

python --version
# Python 3.11.5
```

Perfect. We're using the venv's Python.

**[08:30]** Now you work on your project - install packages, run code, write your agent. Everything happens in this isolated environment.

**[08:45]** When you're done, type 'deactivate'. Simple as that. Your prompt returns to normal, and you're back to using system Python.

**[09:00]** You can create multiple virtual environments and switch between them:

```bash
# Project A
cd project-a
source venv/bin/activate
# Work on A
deactivate

# Project B
cd project-b
source venv/bin/activate
# Work on B
deactivate
```

Each is completely independent.

**[09:30]** Common mistake: forgetting to activate. You install packages thinking you're in the venv, but you're actually installing to system Python. Always check your prompt! That (venv) indicator is your friend.

**[10:00]** Another tip: if you're using an IDE like VS Code or PyCharm, they detect virtual environments automatically and activate them for you. Very convenient."

---

## Section 4: Package Management with pip [10:30 - 14:30]

### Presenter Notes
- Show essential pip commands
- Explain version specifiers clearly
- Demonstrate requirements files
- Connect to agent dependency management

### On-Screen Content
**CODE EXAMPLE 3: pip Commands**
```bash
# Install package
pip install requests

# Install specific version
pip install requests==2.31.0

# Install minimum version
pip install requests>=2.28.0

# Install version range
pip install "requests>=2.28.0,<3.0.0"

# Upgrade package
pip install --upgrade requests

# Uninstall
pip uninstall requests

# List installed packages
pip list

# Show package details
pip show requests
```

### Script

**[10:30]** PRESENTER:
"Now that we have an activated virtual environment, let's install packages with pip - Python's package installer.

**[DEMONSTRATE CODE EXAMPLE 3]**

**[10:45]** The basic command: pip install package-name. This installs the latest version. Simple.

But for production agents, you want control over versions. Let's talk version specifiers.

**[11:15]** Double equals (==) means exact version. pip install requests==2.31.0 installs exactly that version. Nothing else.

Greater-than-or-equal (>=) means minimum version. pip install requests>=2.28.0 allows 2.28.0 or any newer version.

Less-than (<) means up to but not including. Usually combined with greater-than for a range: 'requests>=2.28.0,<3.0.0' means any 2.x version from 2.28.0 up.

**[12:00]** Why ranges? Semantic versioning. Major versions (3.0) can have breaking changes. Minor versions (2.31) add features but maintain compatibility. Patch versions (2.31.1) are bug fixes.

A range like '>=2.28.0,<3.0.0' says 'any 2.x above 2.28, but not 3.x which might break things.' This balances getting updates with avoiding breakage.

**[12:45]** pip list shows all installed packages with versions. pip show gives details about one package - its version, dependencies, location.

These are your diagnostic tools when debugging 'why isn't this working?'

**[13:15]** Now, you don't want to manually track which packages you installed. That's where requirements files come in:

```bash
# Save installed packages
pip freeze > requirements.txt

# Install from requirements
pip install -r requirements.txt
```

**[13:35]** pip freeze outputs all installed packages with exact versions. Redirect that to requirements.txt and you have a complete snapshot of your environment.

Give that file to a teammate or deploy it to production, and they can recreate your exact environment with pip install -r requirements.txt.

**[14:00]** For agent development, this workflow is standard:
1. Create venv
2. Activate
3. Install packages as you need them
4. pip freeze > requirements.txt
5. Commit requirements.txt to version control
6. Teammates/production use requirements.txt to get exact versions

Clean, reproducible, professional."

---

## Section 5: Introducing uv - The Fast Package Installer [14:30 - 17:30]

### Presenter Notes
- Introduce uv with enthusiasm for the speed
- Show side-by-side comparison
- Demonstrate uv commands
- Explain when speed matters most

### On-Screen Content
**CODE EXAMPLE 4: pip vs uv**
```bash
# Install uv
pip install uv

# Use uv like pip
uv pip install requests
uv pip install -r requirements.txt
uv pip list

# Speed comparison:
# pip: ~45 seconds for 50 packages
# uv:  ~2 seconds for 50 packages
# 20x faster!
```

### Script

**[14:30]** PRESENTER:
"pip is great, but it can be slow - especially with many packages or large projects. Enter uv - a game changer for Python packaging.

uv is a drop-in replacement for pip, written in Rust, and it's 10 to 100 times faster. Not 10% faster - ten TIMES faster. Let me show you.

**[DEMONSTRATE CODE EXAMPLE 4]**

**[15:00]** First, install uv itself with pip. One-time setup.

Now use uv exactly like pip, but prefix commands with 'uv pip':
- uv pip install requests
- uv pip install -r requirements.txt
- uv pip list

All the same commands. Same behavior. Just dramatically faster.

**[15:30]** How much faster? In a test installing 50 packages:
- pip: 45 seconds
- uv: 2 seconds

That's 20x faster. For large projects or CI/CD pipelines running repeatedly, this saves enormous time.

**[16:00]** Why is uv so fast? It's written in Rust, a compiled language optimized for performance. It parallelizes downloads and installations. It has smarter caching. Every optimization you can think of.

**[16:20]** When does speed matter most?

CI/CD pipelines that install dependencies on every run.
Large projects with many dependencies.
When you're iterating rapidly and reinstalling often.
Team environments where multiple developers install frequently.

**[16:45]** Should you use uv or pip? My recommendation: learn both. pip is universal - it's everywhere. uv is the future - dramatically faster. Use pip when you need compatibility, uv when you need speed.

For UiPath Agent development, I'd use uv in development for fast iterations, and pip in production for maximum compatibility. But honestly, uv is becoming so standard that using it everywhere is fine.

**[17:15]** The Python packaging ecosystem is evolving. uv represents the new generation - faster, more reliable, better dependency resolution. Get comfortable with it now."

---

## Section 6: Managing Dependencies [17:30 - 21:30]

### Presenter Notes
- Explain requirements files in detail
- Show version specifier strategies
- Introduce pyproject.toml
- Demonstrate lock files for reproducibility

### On-Screen Content
**CODE EXAMPLE 5: Requirements Files**
```
# requirements.txt
uipath-sdk==1.0.0
langchain>=0.1.0,<0.2.0
pandas~=2.0.0
python-dotenv

# requirements-dev.txt
pytest>=7.4.0
black
mypy
```

**CODE EXAMPLE 6: pyproject.toml**
```toml
[project]
name = "document-agent"
version = "1.0.0"
requires-python = ">=3.10"
dependencies = [
    "uipath-sdk>=1.0.0",
    "langchain>=0.1.0,<0.2.0",
]

[project.optional-dependencies]
dev = ["pytest>=7.4.0", "black"]
```

### Script

**[17:30]** PRESENTER:
"Let's talk about managing dependencies properly. requirements.txt is the traditional approach - simple, widely supported, works everywhere.

**[DEMONSTRATE CODE EXAMPLE 5]**

**[17:45]** Here's a typical requirements.txt for a UiPath agent. Notice different version specifiers:

Exact versions (==) for critical packages where you need specific behavior.
Ranges (>=,<) for libraries where you want bug fixes but not breaking changes.
Tilde (~) for compatible releases - ~=2.0.0 means >=2.0.0 but <2.1.0.

**[18:20]** You can also have multiple requirements files. requirements-dev.txt for development-only tools like pytest and black. Install both in development, only requirements.txt in production. Keeps production lean.

**[18:45]** Now, the modern approach: pyproject.toml.

**[DEMONSTRATE CODE EXAMPLE 6]**

**[19:00]** pyproject.toml is the new standard - PEP 621. It combines project metadata and dependencies in one standardized file. Build tools, IDEs, and package managers all read it.

You define dependencies in the dependencies array. Optional dependencies go in optional-dependencies - like dev tools that production doesn't need.

**[19:30]** To install from pyproject.toml:
```bash
pip install .              # Core dependencies
pip install ".[dev]"       # Core + dev dependencies
```

**[19:45]** So which should you use? My recommendation: both.

Use pyproject.toml as your source of truth - it's modern, standardized, tool-friendly.
Generate requirements.txt for deployment - it's simple, universal, works everywhere.

Best of both worlds.

**[20:15]** One more concept: lock files. pip freeze gives exact versions - a snapshot. Some tools generate lock files automatically (like uv.lock).

Lock files ensure exact reproducibility. Same versions every time, everywhere. Critical for production reliability.

**[20:40]** The pattern:
- pyproject.toml: Source of truth, flexible version ranges
- requirements.txt (frozen): Exact deployment snapshot
- Both in version control

This gives flexibility in development and reproducibility in production."

---

## Section 7: Real-World Workflows [21:30 - 25:30]

### Presenter Notes
- Show complete new project workflow
- Demonstrate existing project setup
- Present CI/CD integration
- Connect all concepts together

### On-Screen Content
**CODE EXAMPLE 7: New Project Workflow**
```bash
# 1. Create project
mkdir document-agent && cd document-agent

# 2. Create venv
python -m venv venv

# 3. Activate
source venv/bin/activate

# 4. Upgrade pip, install uv
python -m pip install --upgrade pip
pip install uv

# 5. Install dependencies
uv pip install uipath-sdk langchain pandas

# 6. Save requirements
uv pip freeze > requirements.txt

# 7. Create pyproject.toml (optional)
# 8. Start coding!
```

### Script

**[21:30]** PRESENTER:
"Let's put everything together with real-world workflows. First, starting a new project.

**[DEMONSTRATE CODE EXAMPLE 7]**

**[21:45]** Step by step:

Create your project directory. This is your workspace.

Create a virtual environment inside it. Convention is to name it 'venv'.

Activate the venv. Always check your prompt!

First thing: upgrade pip. The version that ships with venv is often outdated. Then install uv for speed.

**[22:30]** Now install packages as you need them. For a UiPath agent, you might start with uipath-sdk, langchain, pandas - whatever your agent needs.

As you add packages, periodically run pip freeze to update requirements.txt. This keeps your dependency list current.

Optionally, create pyproject.toml for modern project configuration.

**[23:00]** Now start coding! Your environment is set up, dependencies are tracked, you're ready to build your agent.

**[23:15]** Second workflow: joining an existing project.

```bash
# 1. Clone repository
git clone <repo-url>
cd project-name

# 2. Create venv
python -m venv venv

# 3. Activate
source venv/bin/activate

# 4. Install dependencies
pip install -r requirements.txt
# Or with uv:
pip install uv
uv pip install -r requirements.txt

# 5. Start working!
```

**[23:45]** You clone the repo, create your own venv (never clone someone else's venv!), and install from requirements.txt. Within minutes you have the exact environment the project needs.

**[24:10]** Third workflow: CI/CD pipelines.

```yaml
# GitHub Actions example
- name: Set up Python
  uses: actions/setup-python@v4
  with:
    python-version: '3.11'

- name: Install dependencies
  run: |
    python -m venv venv
    source venv/bin/activate
    pip install uv
    uv pip install -r requirements.txt

- name: Run tests
  run: |
    source venv/bin/activate
    pytest tests/
```

**[24:40]** Even automated pipelines use virtual environments - create venv, install dependencies, run tests. All in an isolated, reproducible environment.

This pattern - venv → activate → install from requirements.txt → work - becomes second nature. It's how professional Python development works."

---

## Section 8: Best Practices and Common Issues [25:30 - 28:30]

### Presenter Notes
- Present essential best practices
- Show common mistakes and solutions
- Cover .gitignore
- Provide troubleshooting tips

### On-Screen Content
**BEST PRACTICES:**
```
✅ One venv per project
✅ requirements.txt in version control
✅ venv/ in .gitignore (never commit!)
✅ Activate before installing
✅ Pin versions for production
✅ Upgrade pip first
✅ Use uv for speed
✅ Separate dev dependencies
```

**COMMON ISSUES:**
```
❌ "pip not found" → Ensure venv activated
❌ Wrong Python version → Specify on creation
❌ Permission errors (Windows) → Allow scripts
❌ Forgot to activate → Check prompt!
```

### Script

**[25:30]** PRESENTER:
"Let's cover best practices and common issues to save you headaches.

**[25:45]** Best practices:

One virtual environment per project. Keep them isolated.

Always commit requirements.txt to version control. Never commit venv/ directory - it's huge and not portable.

Always activate before installing packages. Check your prompt!

Pin exact versions for production deployments. Use flexible ranges for development.

Upgrade pip before installing packages - newer pip has better dependency resolution.

Use uv for faster installations in development and CI/CD.

Separate development dependencies from production.

**[26:45]** Common issues:

'pip not found' after activation - the venv might be corrupted. Delete and recreate it.

Wrong Python version in venv - when creating, specify: python3.11 -m venv venv.

Permission errors on Windows with activation - run: Set-ExecutionPolicy RemoteSigned.

Most common issue: forgetting to activate. You install packages to system Python instead of venv. Always check your prompt for (venv)!

**[27:30]** Your .gitignore should always include:

```gitignore
venv/
__pycache__/
*.pyc
.env
```

Virtual environment, Python cache, and environment variables - never commit these.

**[27:50]** Security note: regularly check for vulnerable packages:

```bash
pip install safety
safety check

# Or:
pip install pip-audit
pip-audit
```

Keep your dependencies updated for security patches."

---

## Section 9: Summary and Next Steps [28:30 - 30:00]

### Presenter Notes
- Recap all key concepts
- Emphasize the importance for professional development
- Preview next module
- Motivate for practice

### On-Screen Content
**KEY TAKEAWAYS:**
```
✓ Virtual environments isolate project dependencies
✓ One venv per project - no conflicts
✓ pip: Traditional, universal
✓ uv: Modern, 10-100x faster
✓ requirements.txt: Track dependencies
✓ pyproject.toml: Modern standard
✓ Always in version control: requirements.txt
✓ Never in version control: venv/
```

### Script

**[28:30]** PRESENTER:
"Let's recap. Virtual environments are isolated Python installations - one per project. They prevent version conflicts and enable reproducible builds.

**[28:45]** Create them with python -m venv venv. Activate with source venv/bin/activate (Linux/Mac) or venv\Scripts\activate (Windows). Your prompt changes to show you're activated.

Install packages with pip or the much faster uv. Track dependencies in requirements.txt with pip freeze. Install from requirements with pip install -r.

Use pyproject.toml as the modern standard for project configuration.

**[29:15]** This workflow - create venv, activate, install, freeze requirements, code - is fundamental to professional Python development. Master it now and it becomes automatic.

**[29:30]** For UiPath Agent development, proper dependency management is non-negotiable. Agents deployed to production must have reproducible environments. No surprises, no 'works on my machine' excuses.

**[29:45]** In our next module, we'll dive into asynchronous programming - async/await, coroutines, and building concurrent agents. This is advanced but essential for modern agent development.

Thank you for learning with me today. You now have professional-level package management skills. Practice the workflows, and see you next time!"

**[30:00]** END

---

## Production Notes

### Visual Elements
- Show terminal with prompt changes during activation
- Display directory structure side-by-side
- Animate package installation speed comparison
- Show venv creation process step-by-step
- Display requirements.txt file contents

### Graphics Needed
- Virtual environment isolation diagram
- Project directory structure tree
- pip vs uv speed comparison chart
- Workflow flowcharts (new project, existing project)
- Version specifier syntax guide
- .gitignore template

### Code Files
- All examples in course repository
- Sample pyproject.toml templates
- Sample requirements.txt files
- Reference setup script
- Lab starter files

### Accessibility
- Closed captions with command syntax clearly spelled
- Transcript with all code examples
- High contrast terminal output
- Clear narration of all commands

### Common Student Questions (FAQ)
1. "Do I need a venv for every project?"
2. "Why is my venv folder so big?"
3. "Can I move a venv to another directory?"
4. "What's the difference between venv and virtualenv?"
5. "Should I use pip or uv?"
6. "Why does pip freeze show so many packages?"

---

## Timing Breakdown

| Section | Duration | Topics |
|---------|----------|--------|
| 1. Introduction | 3:00 | Problem, solution overview |
| 2. What are venvs | 3:30 | Concept, benefits |
| 3. Create/Activate | 4:00 | Setup, activation |
| 4. pip Commands | 4:00 | Installing, managing |
| 5. uv Introduction | 3:00 | Fast alternative |
| 6. Dependencies | 4:00 | requirements, pyproject |
| 7. Workflows | 4:00 | Real-world patterns |
| 8. Best Practices | 3:00 | Tips, troubleshooting |
| 9. Summary | 1:30 | Recap, next steps |
| **Total** | **30:00** | |

---

## Related Resources
- venv Documentation: docs.python.org/3/library/venv.html
- pip Documentation: pip.pypa.io
- uv Project: github.com/astral-sh/uv
- PEP 621 - pyproject.toml: peps.python.org/pep-0621/
- Python Packaging Guide: packaging.python.org
